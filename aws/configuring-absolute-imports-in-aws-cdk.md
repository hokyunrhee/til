# Configuring absolute imports in AWS CDK

`ts-node`를 사용하고 있기 때문에 `ts-node`가 `tsconfig.json`의 `paths` 옵션을 따르도록 설정하면 됩니다.

## Install `tsconfig-paths`

```
npm i tsconfig-paths
```

## Configure tsconfig.json

`tsconfig.json`

```json
{
  "compilerOptions": {
    "baseUrl": "."
    // 다른 옵션은 생략
  },
  "ts-node": {
    "require": ["tsconfig-paths/register"]
  }
}
```
