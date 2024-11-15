# Implementing Error Cause Support in http-errors with patch-package

Node.js 애플리케이션에서 HTTP 에러를 처리할 때 널리 사용되는 http-errors 라이브러리는 아직 ES2022에서 도입된 Error cause를 공식적으로 지원하지 않습니다. 이는 에러 디버깅과 로깅에 있어 중요한 컨텍스트 정보를 놓치게 만듭니다. patch-package를 활용하여 http-errors 라이브러리를 수정함으로써 이를 해결할 수 있습니다. 이 방법은 라이브러리가 공식적으로 cause를 지원하기 전까지 사용할 수 있는 실용적인 방법입니다.

## 패치

두 가지 파일을 수정합니다

- http-errors - 생성자에 options 파라미터 추가
- @types/http-errors - 타입 정의에 ErrorOptions 추가

`node_modules/http-errors/index.js`를 다음과 같이 수정합니다. 코드를 변경한 후 `npx patch-package http-errors`를 실행하면 루트 디렉토리에 patches 폴더 안에 `http-errors+2.0.0.patch`가 생성됩니다.

```patch
@@ -127,10 +127,10 @@ function createHttpErrorConstructor () {
 function createClientErrorConstructor (HttpError, name, code) {
   var className = toClassName(name)

-  function ClientError (message) {
+  function ClientError (message, options) {
     // create the error object
     var msg = message != null ? message : statuses.message[code]
-    var err = new Error(msg)
+    var err = new Error(msg, options)

     // capture a stack trace to the construction point
     Error.captureStackTrace(err, ClientError)
@@ -196,10 +196,10 @@ function createIsHttpErrorFunction (HttpError) {
 function createServerErrorConstructor (HttpError, name, code) {
   var className = toClassName(name)

-  function ServerError (message) {
+  function ServerError (message, options) {
     // create the error object
     var msg = message != null ? message : statuses.message[code]
-    var err = new Error(msg)
+    var err = new Error(msg, options)

     // capture a stack trace to the construction point
     Error.captureStackTrace(err, ServerError)
```

`node_modules/@types/http-errors/index.d.ts`를 다음과 같이 수정합니다. 코드를 변경한 후 `npx patch-package @types/http-errors`를 실행하면 루트 디렉토리에 patches 폴더 안에 `@types+http-errors+2.0.4.patch`가 생성됩니다.

```patch
@@ -18,8 +18,8 @@ declare namespace createHttpError {
     type UnknownError = Error | string | { [key: string]: any };

     interface HttpErrorConstructor<N extends number = number> {
-        (msg?: string): HttpError<N>;
-        new(msg?: string): HttpError<N>;
+        (msg?: string, options?: ErrorOptions): HttpError<N>;
+        new(msg?: string, options?: ErrorOptions): HttpError<N>;
     }

     interface CreateHttpError {
```

## 테스트

```ts
import { describe, it, expect } from "vitest";
import createError from "http-errors";

describe("http-errors with Error Cause Support", () => {
  it("should create BadRequest error with Error instance as cause", () => {
    const originalError = new Error("Validation failed");
    const error = createError.BadRequest("Invalid request", {
      cause: originalError,
    });

    expect(error.status).toBe(400);
    expect(error.message).toBe("Invalid request");
    expect(error.cause).toBe(originalError);
  });
});
```
