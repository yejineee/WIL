# tsconfig.json에서 module이 뭘까

ts project를 실행시킬 때, 다음과 같은 에러가 났다.

```text
(node:27135) Warning: To load an ES module, set "type": "module" in the package.json or use the .mjs extension.
```

esmodule을 사용하기 위해 추가적인 설정이 필요했다. 서칭해본 결과 tsconfig.json에서 module을 commonjs로 바꾸어서 해결할 수 있었다.

이참에 tsconfig.json에서 module을 설정하면 컴파일러가 무엇을 해주는지를 파악해보았다.

## tsconfig.json란?

* tsconfig.json 파일이 위치하는 곳이 타입스크립트 프로젝트의 루트임을 나타낸다. 
* tsconfig.json에 typescript 컴파일러 설정을 할 수 있다.

## compilerOption

tsconfig.json에서 compilerOption을 정할 수 있다. compilerOption은 컴파일러 옵션에서 많은 타입스크립트 설정을 할 수 있고, 타입스크립트가 어떻게 동작해야 하는지를 다룬다.

* tsconfig.json

  \`\`\`json

  {

  "compilerOptions": {

  "module": "commonjs",

  "declaration": true, // d.ts 파일 생성 여부

  ...

  }

  }

```text
### Module

- 프로그램의 모듈 시스템을 설정한다. 
- "module"을 설정하는 것은 moduleResolution에 영향을 준다.

Module은 현재 프로젝트가 esmodule 시스템을 사용하는지를 나타내는게 아니고, 컴파일된 결과가 어떤 모듈 시스템일지를 설정한다.

다음과 같은 ts 파일은 설정된 module system을 목표로 하여 컴파일된다. 
```typescript=
// @showEmit
// @module: commonjs
// @noErrors
import { valueOfPi } from "./constants";

export const twoPi = valueOfPi * 2;
```

* CommonJS
  * 모듈 정의는 exports 객체를 이용하고, 모듈 사용은 require 함수를 이용한다.
  * node.js 서버에서도 모듈 시스템을 사용하기 위해 만들어진 모듈 시스템이다. 
  * CJS의 동작 방식

    `require()`가 **동기적**이다; 프라미스를 리턴하거나, 콜백을 호출하지 않는다. require\(\)가 디스크에서부터 스크립트를 읽어와서 즉시 실행시킨다. 그 스크립트는 I/O를 수행하거나, 다른 사이드 이펙트를 실행시킬 수 있다. 그 후에 `module.exports`에서 세팅한 값을 반환한다.

  * .js

    ```javascript
      "use strict";
      Object.defineProperty(exports, "__esModule", { value: true });
      exports.twoPi = void 0;
      // @showEmit
      // @module: commonjs
      // @noErrors
      const constants_1 = require("./constants");
      exports.twoPi = constants_1.valueOfPi * 2;
    ```
* ES2015
  * 모듈 정의는 export, 모듈 사용은 import 문을 사용한다.
  * ESM의 동작 방식

    모듈 로더는 **비동기**로 동작한다. 처음엔 파싱을 하게된다. 스크립트를 파싱하여 import와 export를 호출한 부분을 찾는다\(찾기만 하고 실행시키지는 않는다\). 파싱할 때, named import에서의 오타를 감지하고, exception을 던질 수 있다. 실제 의존 코드는 실행시키지 않고도 오류를 감지할 수 있는 것이다.

    파싱이 끝나고 나면, 비동기적으로 다운로드받고, import한 스크립트를 파싱하게 된다. 더이상 import하는 스크립트가 없을 때까지 의존 관계를 나타내는 "module graph"를 그리게 된다. 스크립트를 실행해도 된다면, 스크립트를 실행하게 된다.

  * .js

    ```javascript
      // @showEmit
      // @module: commonjs
      // @noErrors
      import { valueOfPi } from "./constants";
      export const twoPi = valueOfPi * 2;
    ```

## 참고

* [typsconfig \#modules - typescriptlang.org](https://www.typescriptlang.org/tsconfig#module)
* [Node Modules at War: Why CommonJS and ES Modules Can’t Get Along](https://redfin.engineering/node-modules-at-war-why-commonjs-and-es-modules-cant-get-along-9617135eeca1)
* [Modules, introduction - javascript.info](https://javascript.info/modules-intro)

