# Configure Proxy In Vite

Vite에서는 개발 서버 설정을 이용하여 프록시를 구성할 수 있습니다. 프론트엔드와 백엔드를 독립적으로 개발할 때 종종 유용합니다.

예를 들어 프론트엔드 로컬 개발 환경에서 실제 백엔드 API를 호출하는 경우 발생하는 CORS 에러를 해결할 수 있습니다.

## Vite 설정하기

`vite.config.js` 파일에서 개발 서버 설정을 통해 프록시를 구성할 수 있습니다. 다음은 기본 예시입니다.

```ts
import react from "@vitejs/plugin-react-swc";
import { defineConfig } from "vite";

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      "/api": {
        target: "https://example.com",
        changeOrigin: true,
        secure: false,
        // rewrite: (path) => path.replace(/^\/api/, ""),
      },
    },
  },
});
```

이 설정의 내용은 다음과 같습니다.

1. `/api`로 시작하는 모든 요청을 프록시합니다.
2. `target`은 프록시 요청을 전달할 대상 서버의 주소입니다. `/api`로 시작하는 모든 요청은 이 주소로 전달됩니다.
3. `changeOrigin: true`는 요청의 출처(Origin)을 대상 서버의 주소로 변경합니다. 이 설정이 CORS 에러를 해결합니다.
4. `secure: false`은 HTTPS 요청 시 SSL 인증서 검증을 비활성화합니다. 개발 환경에서 자체 서명된 인증서를 사용하고 있다면 유용합니다.
5. `rewrite`는 주소를 고칩니다. 예시 코드의 주석된 `rewrite` 함수는 `/api` 접두사(Prefix)를 제거합니다. .

## 사용 예시

```ts
fetch("/api/todos");
```

이 코드에서 `/api/todos`로 요청을 보내면, Vite의 개발 서버는 이 요청을 `https://example.com/api/todos`로 전달합니다. 그리고 서버의 응답은 프론트엔드로 반환됩니다.
