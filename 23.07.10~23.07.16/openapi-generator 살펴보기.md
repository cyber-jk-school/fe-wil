# OpenAPI Generator 살펴보기 1편

## 들어가며

회사에서 백엔드 개발자와 프론트엔드 개발자 간 API 관련 소통을 원활하게 하기 위해 Swagger 문서 기반으로 OpenAPI Generator를 사용해 API 호출 함수와 그에 따른 매개변수 및 응답값의 타입을 자동 생성해 사용하고 있습니다. OpenAPI Generator를 사용했을 때 제가 느낀 장점으로는

1. 백엔드 개발자와 API 관련해서 불필요하게 소통하는 게 줄었다.
   - Swagger 문서를 통해 API 명세를 확인하기 때문에 API 엔드포인트가 어디고 파라미터나 바디는 어떤 걸 넣어야 하는지 등을 공유하는 과정이 사라졌습니다. (근데 사실 이건 대부분 회사가 안 그럴 것 같긴 하네요)
2. 타입 정의를 하지 않아도 됨.
   - 타입스크립트를 사용해 API를 호출할 때의 가장 큰 문제는 응답값에 대한 타입을 정의해야 한다는 건데요. 타입을 정의할 수는 있지만 유지/관리 관점에서 문제가 생깁니다.
   - 프로젝트 초반에는 정리된 문서 등을 보면서 타입을 직접 정의할 수 있습니다. 그런데 작업이 진행되면서 응답값의 형식이 변경되고, 타입이 변경되면 그에 따라 기존에 정의했던 타입들을 직접 수정해야 합니다. 근데 이건 너무 힘들죠. 결국 개발자 경험이 나빠지는 것 같아요. OpenAPI Generator를 사용하면 Swagger 문서 기반으로 타입이 자동으로 정의되니까 정의된 타입을 사용할 수 있습니다. 엄청 편해요.
3. 이 외에도 다양한 장점이 있지만, 구글링 하면 많이 나오니까 한 번 찾아보셔도 좋습니다!

## 실제로 사용해보기

회사에서의 설정은 이미 되어 있었기 때문에, 직접 설정해보기로 했습니다. 생각했던 과정은 다음과 같습니다.

1. OpenAPI Generator가 잘 동작하는지 테스트해보기
2. Github Actions를 사용해 자동으로 코드 생성하고 Github Packages로 배포하기

### 1. OpenAPI Generator가 잘 동작하는지 테스트해보기

일단 새 프로젝트를 위한 폴더를 만들고, `yarn add @openapitools/openapi-generator-cli` 명령어를 실행해 패키지를 설치합니다. 해당 라이브러리는 JVM 기반으로 동작하기 때문에 필수적으로 JDK가 설치되어 있어야 합니다.  
그 후, generator 설정 파일을 생성합니다. `openapi-generator-cli` 설정에는 2가지 방법이 있는데

1. `{설정 파일 이름}.json` 파일 생성 후 `yarn openapi-generator-cli generate -c {설정 파일 이름}.json` 명령어 실행
2. `openapitools.json` 파일 생성 후 `yarn openapi-generator-cli generate --generate-key {generator-key}` 명령어 실행
   - 이 때 `openapitools.json`의 내용은 다음과 같음.
   ```
   {
    "$schema": "node_modules/@openapitools/openapi-generator-cli/config.schema.json",
    "spaces": 2,
    "generator-cli": {
        "version": "5.4.0",
        "generators": {
        "v3.0": { // 이게 generator-key, 아무 값이나 넣어도 됨.
            /* 변환 방법 결정 */
            "generatorName": "typescript-axios",
            /* 변환 결과물 경로 */
            "output": "./output/",
            /* OpenAPI Specification 문서가 정의되어 있는 경로 */
            "glob": "api-docs/api.yaml"
          }
        }
      }
    }
   ```
   위의 generatorName은 제너레이터를 설정하는 속성입니다. 제너레이터는 각 언어별로 있는데 타입스크립트와 axios를 사용하기 때문에 `typescript-axios`를 사용했습니다. 자세한 제너레이터 리스트는 [여기](https://openapi-generator.tech/docs/generators)

저는 두 번째 방법으로 했습니다. 위의 명령어를 실행하면 output 경로로 설정한 곳에 코드가 생성됩니다. 생성되는 코드의 형태도 커스텀 할 수 있다고 하는데 전 아직까지는 그 정도가 안돼서... 다음 기회에

### 2. Github Actions를 사용해 자동으로 코드 생성하고 Github Packages로 배포하기

사실 이번 주는 여기에서 삽질하다가 시간이 다 날아갔습니다. 일단 Github Packages는 어디에 쓰는거냐하면, npm registry 같은 경우 public 레포지토리는 무료로 사용할 수 있지만 private 레포지토리의 경우 유료로 사용해야 한다고 합니다. 그래서 private 레포지토리의 코드를 배포하기 위해 사용한다고 합니다. 여기는 말로 설명 하겠슴... 말하고 나서 수정 예정임다
