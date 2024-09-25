# Get better at building AWS CDK constructs with Aspects

Aspects를 사용하여 AWS CDK constructs를 더 잘 구성하는 방법을 다룹니다.

Aspect를 이용하여 construct tree 내의 원하는 모든 constructs를 변경할 수 있습니다. 변경 사항을 한 곳에서 관리하고 여러 곳에 일괄 적용할 수 있기 때문에 유지보수가 용이해지는 장점이 있습니다.

다음과 같은 경우에 유용합니다.

- 리소스에 태그 일괄 적용
- 보안 및 규정 사항 검증 및 강제
- 네이밍 컨벤션 강제

```ts
cdk.Aspects.of(myConstruct).add(new MyAspect());
```

## `Aspect`의 작동 방식

Aspect는 `IAspect` 인터페이스를 구현하여 만듭니다. 이 인터페이스는 `visit` 메서드를 가지고 있는데, [synthesize](https://github.com/aws/aws-cdk/blob/main/packages/aws-cdk-lib/core/lib/private/synthesis.ts#L34) 과정에서 construct tree를 순회하며 각 노드에 대해 `visit` 메서드를 호출합니다.

```ts
interface IAspect {
  visit(node: IConstruct): void;
}
```

## 사용 사례: 태그 일괄 적용

```ts
// apply-tags.ts
import { IAspect, ITaggable, TagManager } from "aws-cdk-lib";
import { IConstruct } from "constructs";

type Tags = Record<string, string> & {
  stage: "dev" | "prod";
  project: string;
};

export class ApplyTags implements IAspect {
  #tags: Tags;

  constructor(tags: Tags) {
    this.#tags = tags;
  }

  visit(node: IConstruct) {
    if (TagManager.isTaggable(node)) {
      Object.entries(this.#tags).forEach(([key, value]) => {
        this.applyTag(node, key, value);
      });
    }
  }

  applyTag(resource: ITaggable, key: string, value: string) {
    resource.tags.setTag(key, value);
  }
}
```
