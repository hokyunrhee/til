# Efficient Lambda Deployment with AWS CDK: Using Checksum Instead of Timestamp

AWS CDK Construct로 Lambda 함수를 구성할 때, 최초 배포 이후 코드가 변경되어도 함수가 자동으로 재배포되지 않는 문제가 있습니다. 많은 개발자들이 이 문제를 해결하기 위해 description에 현재 시간을 추가하는 방식을 사용합니다:

```ts
const lambda = new NodejsFunction(this, "Lambda", {
  // 기존 설정...
  description: `Create at: ${new Date().toISOString()}`,
});
```

하지만 이 방식은 `cdk deploy` 명령을 실행할 때마다 코드 변경 여부와 관계없이 매번 새로운 함수가 배포되는 비효율이 발생합니다.

이를 개선하기 위해, Lambda 함수의 소스 코드가 실제로 변경되었을 때만 배포가 이루어지도록 할 수 있습니다. handler 파일의 체크섬(Checksum)을 계산하여 이를 description에 활용하는 방식입니다. 이렇게 하면 소스 코드에 변경이 없는 경우 description 값이 동일하게 유지되어 불필요한 배포를 방지할 수 있으며, 코드가 변경된 경우에만 새로운 체크섬 값으로 인해 재배포가 트리거됩니다.

다음은 체크섬을 활용한 구현 예시입니다:

```ts
import { readFileSync } from "fs";
import { createHash } from "crypto";

export const getHash = (filePath: string): string => {
  const fileBuffer = readFileSync(filePath);
  return createHash("sha256").update(fileBuffer).digest("hex");
};
```

```ts
import { join } from "path";

const hash = getHash(join(__dirname, "handler.js"));

const lambda = new NodejsFunction(this, "Lambda", {
  // 기존 설정...
  description: hash,
});
```

## References

- [Deploying new version of lambda function](https://github.com/aws/aws-cdk/issues/5334#issuecomment-562981777)
