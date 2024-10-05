# Managing Env Variables with mode

여러 환경에 배포해야하는 프로젝트의 경우 `development`, `production`, `staging` 등 환경별로 변수를 관리할 필요가 생기게 됩니다. Vite의 `--mode` 옵션을 이용하면 환경별로 깔끔하게 관리할 수 있습니다.

## 사용 예시

### `.env.[mode]` 파일

프로젝트의 루트에 환경 파일을 생성합니다.

```
.env.development
.env.staging
.env.production
```

예를 들어 환경 변수 파일에 다음과 같이 작성합니다.

```
VITE_CLIENT_ID=xxxxxxxxxxxxxxxxxxxxxxxx
```

### 타입 정의

`src` 디렉토리에 `vite-env.d.ts` 파일을 생성하고 다음과 같이 작성합니다.

```ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_CLIENT_ID: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

### 환경 변수 사용

```ts
console.log(import.meta.env.VITE_CLIENT_ID); // "xxxxxxxxxxxxxxxxxxxxxxxx"
```

### npm script 설정

package.json에 다양한 환경을 위한 스크립트를 추가하여 편리하게 사용할 수 있습니다

```json
{
  "scripts": {
    "dev": "vite",
    "build:development": "vite build --mode development",
    "build:staging": "vite build --mode staging",
    "build:production": "vite build --mode production"
  }
}
```

## References

- [Env Variables and Modes](https://vite.dev/guide/env-and-mode)
