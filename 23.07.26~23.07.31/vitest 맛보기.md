# Vitest 맛보기

일을 할 때 명확하게 해야 할 작업을 나누고 작업을 하면 좀 괜찮을 수도 있는데, 명확하게 정리가 되지 않은 상태에서 일을 하면 어떤 작업을 해야 하는지 또는 요구 사항을 잘 만족했는지 등을 잘 파악할 수 없었습니다. 그래서 테스트 도입해서 구현해야 할 사항들을 테스트로 작성하고 해당 테스트를 만족시키는 방향으로 개발하면 좋겠다라고 막연히 느꼈습니다. 이게 아마 TDD가 아닐까 싶기도...

## Vitest란

[Vitest](https://vitest.dev/)는 Vite와 사용하기 편한 테스트 라이브러리입니다. Vitest 공식 문서에도 이렇게 적혀있습니다.

> **Vite Powered**  
> Reuse Vite's config, transformers, resolvers, and plugins - consistent across your app and tests.

사용해보니 Vite 설정 파일에 테스트 설정까지 한 번에 작성할 수 있더라고요. 나머지 transformers, resolvers, plugins 들은 아직 잘 모르겠지만 장점이 있는 건 분명합니다.

그 외에도 기존에 유명했던 테스트 라이브러리인 Jest와 비슷한 API를 제공합니다. 자세하게 비교했을 때 차이점이 있을 수도 있지만, 간단하게 작성된 코드를 봤을 때 Jest와 Vitest로 작성한 코드의 차이점이 없었습니다.

이 외의 다양한 장점들은 공식 문서를 확인하세요.

## Vite + Vitest 설정

Vite + Vitest 설정은 간단합니다. `vite.config.js` 파일 내에 Vitest 설정을 포함할 수 있습니다.

```js
import { defineConfig } from "vite";

export default defineConfig({
  test: {
    // ...
  },
});
```

## Next.js + Vitest 설정

Vite 이외의 번들러를 사용할 경우 `vite.config.js` 파일이 없기 때문에 테스트 설정을 위해 `vitest.config.js` 파일을 생성해야 합니다.

```js
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    // ...
  },
});
```

test 내부의 설정은 Vite 설정과 동일합니다. 자세한 설정은 [문서](https://vitest.dev/config/)를 참고하세요.

설정 파일을 작성한 후 프로젝트의 `package.json`에 스크립트를 추가합니다.

```json
{
    ...
    "scripts": {
        ...
        "test": "vitest"
    }
}
```

test 명령어를 실행하면 테스트가 동작합니다. 이 때, 테스트 파일은 include 속성을 통해 명시할 수 있습니다.

```js
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    include: ["./src/**/*.{test, spec}.{js,ts}"],
  },
});
```

include 기본값은 `['**/*.{test,spec}.?(c|m)[jt]s?(x)']`입니다.

## 테스트 코드 작성하기

실제 테스트 코드를 작성하기 위해 include 형식에 맞는 파일을 생성합니다. (ex: `index.test.ts`) 해당 파일 내에 원하는 대로 테스트 코드를 작성하면 됩니다. 자세한 내용은 [문서](https://vitest.dev/guide/features.html)를 참고하세요.

### UI 요소 테스트하기

어떤 버튼을 클릭 했을 때 특정한 모달이 뜨는지 테스트하고 싶다면, 컴포넌트를 렌더링하고 렌더링 된 버튼을 클릭 했을 때 모달이 렌더링 되는지 확인해야 합니다. 이를 위해선 Vitest 뿐 아니라 다른 테스트 라이브러리를 사용해야 합니다. 저는 [react-testing-library](https://testing-library.com/docs/react-testing-library/intro/)를 사용했습니다. 테스트 라이브러리를 사용하면 가상 DOM을 이용해 DOM 요소를 확인해야 하는데 이 때 Vitest 설정을 추가해야 합니다. 해줘야 합니다.

```js
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    environment: "jsdom",
  },
});
```

jsdom 라이브러리를 사용해 가상 DOM으로 테스트를 진행할 수 있습니다.

## 문제점

테스트를 진행하기 위해서는 테스트 환경을 구성해야 합니다. 예를 들면 필요한 라이브러리를 설치한다거나 하는 사전 작업이 필요한데 아직 테스트에 대해 잘 알지 못해서 테스트 시 오류가 발생하고 있습니다. Next.js 13 버전부터 추가된 next font 기능을 사용했는데 해당 함수를 불러올 때 `default is not a function` 이라는 에러가 발생하고 있는데 아직 해결을 못했네요ㅠㅠ Vitest 패키지도 까보고 찾아보고 있는데 아직 정보를 못찾았어요...
