# Vue3의 반응형 알아보기

## Vue3의 반응형 알아보기

## 📕 Reactivity In Depth

Models는 **proxied Javascript Object**이다. 모델을 변경하면, 뷰가 업데이트 된다. 이게 어떻게 이루어지는지를 뷰의 반응형 시스템의 내부 구조를 파악하면서 이해해보자.

### 📍 Vue는 어떻게 코드가 Running하고 있는 것을 알까? : effect

* 참고 : [vue-next의 effect.ts 코드](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/effect.ts)

value가 바뀔 때, sum이 바뀌기 위해서는 sum을 함수 안에 sum을 넣어야 한다.

```javascript
const updateSum = () => {
  sum = val1 + val2
}
```

Vue에게 어떻게 이 함수를 설명할 수 있을까?

**Vue는 `effect`를 사용하여 현재 동작하고 있는 함수를 추적**한다. **`effect`는 함수를 감싸는 wrapper인데, 이는 함수가 호출되기 전에 tracking을 시작한다.** 뷰는 어떠한 effect가 동작하고 있는지를 어느 시점에서든지 알 수 있고, 필요할 때 다시 실행시킬 수 있다.

이를 이해하기 위해, 뷰가 하는 역할과 비슷한 것을 흉내내보자.

```javascript
createEffect(() => {
  sum = val1 + val2
})
```

sum을 감싸는 `createEffect`를 만든다.

* createEffect는 sum이 언제 동작하고 있는지를 추적해야 한다.
  1. 현재 동작하고 있는 effect들을 저장하는 runningEffects라는 배열을 둔다. 
  2. effect가 호출되면, 함수 호출 직전에 자기 자신을 runningEffects 배열에 추가한다.

     =&gt; **어떠한 effect가 현재 실행되고 있는지는 runningEffects 배열을 확인하면 된다.**

```javascript
// Maintain a stack of running effects
const runningEffects = []

const createEffect = fn => {
  // Wrap the passed fn in an effect function
  const effect = () => {
    runningEffects.push(effect)
    fn()
    runningEffects.pop()
  }

  // Automatically run the effect immediately
  effect()
}
```

Effects는 다른 기능들의 starting point가 된다. 예를 들어, **component rendering과 computed property는 내부적으로 effect를 사용한다**. 어떤 데이타가 변경되었을 때, 반응하는 것이 있다면, 그것이 effect로 감싸져있구나를 생각하면 된다.

### 📍  Vue는 어떻게 변화를 추적할까

현재 실행하고 있는 함수는 effect로 감싸서 알 수 있음을 위에서 살펴보았다. 여기서는 1. 뷰가 어떻게 effect와 Data 간의 의존관계를 알아내는지 2. 어떻게 Reactivity를 만들어내는지를 살펴볼 것이다.

local 변수를 재할당하는 것은 추적할 수 없다. 그러한 메커니즘이 자바스크립트에는 없기 때문이다. 추적할 수 있는 것은 객체의 프로퍼티가 변경되었을 경우이다.

**컴포넌트의 `data` 함수가 plain js object를 반환하면, 뷰는 그 object와 get, set handler를 `Proxy`로 감싼다.**

> 참고: **Proxy**는 해당 객체를 intercept하여 원하는 operation을 수행하도록 조작할 수 있는 객체이다. **Reflect**는 this binding이 Proxy로 되도록 해준다.

빠르게 위의 질문에 대한 답을 내리자면, 1. 뷰가 어떻게 effect와 Data 간의 의존관계를 알아내는가

* 프록시로 get을 intercept하여 프로퍼티와 effect간의 의존관계를 파악할 수 있다.

1. 어떻게 Reactivity를 만들어내는가
   * 프록시로 set을 intercept하여 프로퍼티가 변경되었을 때 effect를 다시 실행시킬 수 있다.

```javascript
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, property, receiver) {
    track(target, property)
    return Reflect.get(...arguments)
  },
  set(target, property, value, receiver) {
    trigger(target, property)
    return Reflect.set(...arguments)
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)

// tacos
```

#### 1. data와 effect 간의 의존 관계 파악하기 : intercept `get`

```javascript
const handler = {
  get(target, property, receiver) {
    track(target, property)
    return Reflect.get(...arguments)
  },
  ...
}
```

Proxy로 Reactivity를 구현하는 첫 번째 방법은 **Property가 읽혀질 때, 어떠한 effect가 해당 프로퍼티에 접근하였는지를 알아내는 것이다.**

이를 추적하기 위해서는 현재 이 프로퍼티에 접근한 것이 무엇인지를 확인해야 한다.

이를 위해서는 getter를 intercept해야 한다.

프록시의 get handler는 track이라는 함수에 argument로 target과 property를 넣어 호출한다.

**track 함수는 현재 동작하고 있는 `effect`가 무엇인지 확인하고, `target`, `property`와 같이 기록한다.**

이렇게하여 **뷰는 해당 effect가 target의 property에 의존하고 있음을 알 수 있다.**

#### 2. data가 변경되었을 때, 그 값에 의존하는 effect 다시 실행시키기 : intercept `set`

```javascript
const handler = {
  set(target, property, value, receiver) {
    trigger(target, property)
    return Reflect.set(...arguments)
  },
  ...
}
```

어떠한 데이터가 변경되면, setter가 호출된다.

Proxy의 핸들러는 **set을 가로채서, 현재 target의 property에 의존하고 있는 effects들을 다시 실행시킨다.**

> 정리

* dependency-tracking은 handler.get에서 처리
* change-notification은 handler.set에서 처리할 수 있게 된다. 

#### Proxied Objects

Vue는 내부적으로 reactive하게 만든 모든 객체를 추적하고 있다. 따라서 항상 같은 객체에 대해 같은 프록시를 반환한다.

만약 reactive proxy에 있는 중첩된 프로퍼티에 접근한다면, 그 **중첩된 객체 또한 프록시로 변환**된다.

```javascript
const handler = {
  get(target, property, receiver) {
    track(target, property)
    const value = Reflect.get(...arguments)
    if (isObject(value)) {
      // Wrap the nested object in its own reactive proxy
      return reactive(value)
    } else {
      return value
    }
  }
  // ...
}
```

#### Proxy vs original identity

**Proxy 객체와 오리지날 객체는 `===`로 판단했을 때 같지 않다.** `.includes()`나 `.indexOf()`와 같이 strict equality comparison을 하는 연산에도 이러한 특성이 영향을 끼치게 된다.

```javascript
const obj = {}
const wrapped = new Proxy(obj, handlers)

console.log(obj === wrapped) // false
```

**가장 좋은 것은 오리지날 객체를 참조\(Reference\)하는 것을 두지 않는 것이다.** 오로지 reactive한 객체로만 동작하도록 해야 한다. 이렇게 해야 equality comparison과 reactivity가 원하던대로 동작하게 된다.

```javascript
const obj = reactive({
  count: 0
}) // no reference to original
```

vue는 primitive value에 대해서는 Proxy로 감싸지 않는다. 따라서 primitive value는 `===` 를 사용할 수 있다.

```javascript
const obj = reactive({
  count: 0
})

console.log(obj.count === 0) // true
```

### 📍 렌더링은 어떻게 변화에 반응할 수 있을까

컴포넌트의 템플릿은 `render` 함수로 컴파일된다. render 함수는 `VNodes`를 생성한다. VNode는 컴포넌트가 어떻게 렌더링해야 하는지를 설명한다. 뷰는 **VNodes를 effect로 감싸서, 실행되는 동안 어떠한 프로퍼티에 접근하였는지를 알아낸다.**

`render` 함수는 개념적으로 `computed` 프로퍼티와 매우 유사하다. 뷰는 정확히 어떻게 dependency가 사용되었는지를 추적하지 않는다. 오직 그 함수가 실행되는 동안 어떠한 시점에 dependency가 사용되었는지만 알 수 있다.

그중 어떠한 프로퍼티가 나중에 변경이되면, effect를 다시 실행시키게끔한다. **`render` 함수를 다시 실행시켜서 새로운 VNode**를 만드는 것이다. 이 VNode는 DOM에 필요한 변화를 만드는데 사용된다.

## 📕 Reactivity Fundamentals

### 📍 Reactive State 선언하기 : `reactive`

자바스크립트 객체에서 reactive state를 만드려면, `reactive` method를 사용해야 한다.

```javascript
import { reactive } from 'vue';

const state = reactive({
    count: 0,
})
```

`reactive`는 vue2의 `Vue.observable()` API와 같고, 이름만 변경되었다. 여기서 **반환되는 값은 반응형 객체**이다. 반응형으로 전환되는 것은 "deep"하게 이뤄진다. 즉, **모든 중첩된 프로퍼티들도 다 반응형**이다.

뷰에서 반응형 상태를 사용하는 중요한 경우는, 렌더링할 때 사용한다는 것이다. 의존성을 추적하기 때문에, 반응형 상태가 변경되면, 뷰는 자동으로 업데이트된다.

**data\(\) 함수에서 객체를 반환하면, 그 객체는 내부적으로 reactive\(\)로 인하여 반응형이 된다.** render 함수로 컴파일 되는 템플릿이 반응형 프로퍼티들을 사용하게 된다.

### 📍  독립적인 반응형 값 만들기 : `ref`

독립적인 **primitive value인 String을 반응형으로 만들고 싶다고 가정**하자. 물론 하나의 String 프로퍼티를 갖는 객체를 만들어서 `reactive`에 넘기면 된다. 뷰에는 이와 같은 역할을 해주는 메소드가 있는데, 이를 `ref`라고 한다.

**`ref`는 `value`라는 프로퍼티 하나만 갖고 있는 반응형이자 변경이 가능한 객체를 반환**한다.

```javascript
import { ref } from 'vue'

const count = ref(0)
console.log(count.value) // 0

count.value++
console.log(count.value) // 1
```

#### Ref Unwrapping

ref가 render context\(setup\(\)이 반환하는 객체\)에서 리턴되고, 템플릿에서 접근하게 될 때, 자동으로 내부의 값을 shallow unwrap한다. 따라서, 중첩된 ref에만 `.value`를 사용하면 된다.

```javascript
<template>
  <div>
    <span>{{ count }}</span>
    <button @click="count++">Increment count</button>
    <button @click="nested.count.value++">Nested Increment count</button>
  </div>
</template>

<script>
  import { ref } from 'vue'
  export default {
    setup() {
      const count = ref(0)
      return {
        count,

        nested: {
          count
        }
      }
    }
  }
</script>
```

> TIP 실제 객체에 접근하지 않아도 된다면, reactive로 감쌀 수 있다.
>
> ```javascript
> nested: reactive({
>     count
> })
> ```
>
> ```text
> <div>nested: {{nested.count}}</div>
> ```

#### 반응형 객체에 접근하기

`ref`가 **반응형 객체의 프로퍼티로서 접근되거나 변경될 때, 자동으로 내부의 값을 unwrap**하여 일반적인 프로퍼티처럼 동작하도록 한다.

```javascript
const count = ref(0)
const state = reactive({
  count
})

console.log(state.count) // 0 -> 객체의 프로퍼티로 접근

state.count = 1
console.log(count.value) // 1 -> ref는 .value로 내부 값에 접근해야 한다.
```

> **Collection Type에서의 반응형 접근 -&gt; unwrapping이 되지 않음.**

* Ref unwrapping은 반응형 `Object` 내부에서 중첩될 경우에만 이루어진다. 
* ref가 `Array`나 `Map`처럼 collection type인 것들에서 접근될 때는 unwrapping이 이루어지지 않는다. 따라서 이때는 `.value`로 내부의 값에 접근해야 한다.

```javascript
const books = reactive([ref('value~')]);
console.log(books[0].value); // value~

const map = reactive(new Map([['count', ref(0)]]));
console.log(map.get('count').value); // 0
```

### 📍 Destructing Reative State : `toRefs`

다음과 같이 **reactive object를 ES6 destructing으로 가져오게 되면, reactivity가 사라지게 된다.**

```javascript
import { reactive } from 'vue'

const book = reactive({
  author: 'Vue Team',
  year: '2020',
  title: 'Vue 3 Guide',
  description: 'You are reading this book right now ;)',
  price: 'free'
})

let { author, title } = book // reactivity가 사라지게 됨
```

Reactivity를 유지하기 위해서는 반응형 객체를 set of refs로 변경시켜야 한다.

set of refs로 만들기 위해서는 `toRefs` API를 사용해야 한다.

**toRefs는 반응형 객체를 각 프로퍼티가 ref인 plain object로 변경**시킨다.

이 ref들은 source object와 reative connection을 유지하게 된다.

toRefs로 각 프로퍼티들은 ref가 되었으므로, 이 내부 값에 접근하기 위해서는 `.value`를 사용해야 한다.

```javascript
import { reactive, toRefs } from 'vue'

const book = reactive({
  author: 'Vue Team',
  year: '2020',
  title: 'Vue 3 Guide',
  description: 'You are reading this book right now ;)',
  price: 'free'
})

let { author, title } = toRefs(book)

title.value = 'Vue 3 Detailed Guide' // we need to use .value as title is a ref now
console.log(book.title) // 'Vue 3 Detailed Guide'
```

### 📍  `readonly`로 반응형 객체를 변경시키는 것을 막기

ref, reactive로 반응형 객체를 만들어서 변경을 추적하게 된다.

그러나 가끔은 어떠한 지점에서 변화를 막아야하는 경우가 생긴다.

데이타를 provide할 때, inject한 쪽에서 그 값을 변경 못시키게 해야할 경우가 있을 것이다.

이러한 경우에 readonly로 감싸서 변경시키지 못하게 해야 한다.

```javascript
<script>
import { provide, reactive, readonly, ref } from 'vue'
import MyMarker from './MyMarker.vue'

export default {
  components: {
    MyMarker
  },
  setup() {
    const location = ref('North Pole')
    provide('location', readonly(location))
  }
}
</script>
```

## 📕 Computed and Watch

### 📍 Computed Values

* [코드샌드박스 예제 - computed values](https://codesandbox.io/s/vue3-computed-and-watch-mhbyw?file=/src/App.vue)

**computed property**는 state에 의존하고 있는 state이다. computed를 만들려면, `computed` method를 사용해야 한다.

> **computed** computed method는 getter 함수를 받아서, getter가 반환한 값에서 **불변 반응형 ref 객체**를 만들어 반환한다.

computed값은 readonly다. 따라서, 이 값을 변경시키는 것은 할 수 없다.

```javascript
const count = ref(1)
const plusOne = computed(() => count.value + 1)

console.log(plusOne.value) // 2

plusOne.value++ // Write operation failed: computed value is readonly
```

* computed에서 Writable ref object 반환받기

computed가 `get`, `set` 함수가 있는 객체를 받으면, writable ref 객체가 된다.

```javascript
const count = ref(1);
const plusOne = computed(() => count.value + 1);
const plusTen = computed({
  get: () => count.value + 10,
  set: (val) => {
    count.value = val;
  },
});
console.log(plusOne.value); //2
console.log(plusTen.value); //11
plusTen.value = 100;
console.log(count.value); //100
```

### 📍 `watchEffect`

* [코드샌드박스 예제 - watchEffect](https://codesandbox.io/s/vue3-watcheffect-7zg5q)

**반응형 state에 따라 side effect를 자동으로 실행시키기 위해서, `watchEffect` 메소드를 사용**할 수 있다. watchEffect는 dependency를 tracking할 때 즉시 실행되고, dependency가 변경될때마다 함수를 다시 실행시킨다.

**watchEffect\(\)는 DOM이 마운트거나 업데이트 되기 전에 실행된다.**

```javascript
const count = ref(0);
watchEffect(() => console.log(count.value)); // logs 0 : dependency tracking하며 즉시 실행
setTimeout(() => {
  count.value++; // logs 1 -> dependency 변경되며 다시 실행
}, 100);
```

#### Stopping the Watcher

`watchEffect`가 `setup()`이나 `lifecycle hooks`에서 호출될 떄, watcher는 컴포넌트의 라이프사이클과 연결되며, 컴포넌트가 언마운트될 때 자동으로 멈춰진다.

명시적으로 Watcher를 멈추고 싶은 경우, `watchEffect`의 반환값을 이용하면 된다. 이 반환값은 stop handler인데, 이를 호출하여 watcher를 멈출 수 있다.

```javascript
  setup() {
    const count = ref(0);
    const stop = watchEffect(() => console.log(count.value)); // logs 0
    stop(); // stop watcher
    setTimeout(() => {
      count.value++; // Watcher가 멈춰졌기에, 로그 1이 찍히지 않는다. 
    }, 100);
  },
```

#### Side Effect Invalidation

* [side effect invalidation](https://v3.vuejs.org/guide/reactivity-computed-watchers.html#side-effect-invalidation)

watched effect 함수가 비동기 side effect를 수행할 경우가 있다. 그 비동기 사이드 이펙트는 invalidate한 경우가 생기면, 클린업되어야 한다.

Effect 함수는 `onInvalidate` 함수를 받는데, 이 함수는 invalidation callback으로 등록된다.

invalidation callback은 다음과 같은 경우에 호출된다.

* effect가 다시 실행되는 시점
* watcher가 멈추는 시점 \(i.e. watchEffect가 setup이나 lifecycle hook에 등록되었을 때, 컴포넌트가 언마운트되는 시점\)

invalidation callback을 함수의 인자로 넘겨서 등록하였다. 왜냐하면 리턴값이 async 에러 핸들링에 중요하기 때문이다.

다음 예가, data fetching할 때, async effect function의 흔한 예이다.

```javascript
const data = ref(null)
watchEffect(async (onInvalidate) => {
  onInvalidate(() => { /* ... */ }) // we register cleanup function before Promise resolves
  data.value = await fetchData(props.id)
})
```

async 함수는 내재적으로 프라미스를 반환하는데, 클린업 함수는 프라미스가 resolve되기 전에 등록되어야 한다. 또한, 뷰는 프라미스 체인에서 발생할 수 있는 에러를 자동으로 처리하기 위해 반환되는 프라미스에 의존한다.

> In addition, Vue relies on the returned Promise to automatically handle potential errors in the Promise chain.

### 🖤 출처

* [Reactivity Fundamentals - vue3](https://v3.vuejs.org/guide/reactivity-fundamentals.html#declaring-reactive-state)
* [Refs API](https://v3.vuejs.org/api/refs-api.html#ref)
* [Computed and Watch - vue3](https://v3.vuejs.org/guide/reactivity-computed-watchers.html#computed-values)

