# Creating CloudWatch RUM with AWS CDK

```ts
import { CfnOutput } from "aws-cdk-lib";
import {
  CfnIdentityPool,
  CfnIdentityPoolRoleAttachment,
} from "aws-cdk-lib/aws-cognito";
import { FederatedPrincipal, PolicyStatement, Role } from "aws-cdk-lib/aws-iam";
import { CfnAppMonitor } from "aws-cdk-lib/aws-rum";
import { Construct } from "constructs";

interface RumProps {
  appMonitorName: string;
  applicationDomain: string;
}

export class Rum extends Construct {
  constructor(scope: Construct, id: string, props: RumProps) {
    super(scope, id);

    const { appMonitorName, applicationDomain } = props;

    const rumIdentityPool = new CfnIdentityPool(this, "RumIdentityPool", {
      allowUnauthenticatedIdentities: true,
    });

    const guestRole = new Role(this, "RumGuestRole", {
      assumedBy: new FederatedPrincipal(
        "cognito-identity.amazonaws.com",
        {
          StringEquals: {
            "cognito-identity.amazonaws.com:aud": rumIdentityPool.ref,
          },
          "ForAnyValue:StringLike": {
            "cognito-identity.amazonaws.com:amr": "unauthenticated",
          },
        },
        "sts:AssumeRoleWithWebIdentity"
      ),
    });

    guestRole.addToPolicy(
      new PolicyStatement({
        actions: ["rum:PutRumEvents"],
        resources: ["*"],
      })
    );

    new CfnIdentityPoolRoleAttachment(this, "RumIdentityPoolRoleAttachment", {
      identityPoolId: rumIdentityPool.ref,
      roles: {
        unauthenticated: guestRole.roleArn,
      },
    });

    const appMonitor = new CfnAppMonitor(this, "RumAppMonitor", {
      name: appMonitorName,
      domain: applicationDomain,
      appMonitorConfiguration: {
        allowCookies: true,
        sessionSampleRate: 1,
        telemetries: ["errors", "performance", "http"],
        guestRoleArn: guestRole.roleArn,
        metricDestinations: [
          {
            destination: "CloudWatch",
          },
        ],
      },
      customEvents: {
        status: "ENABLED",
      },
      cwLogEnabled: true,
    });

    new CfnOutput(this, "RumAppMonitorId", {
      exportName: "rum-app-monitor-id",
      value: appMonitor.attrId,
    });

    new CfnOutput(this, "RumIdentityPoolId", {
      exportName: "rum-identity-pool-id",
      value: rumIdentityPool.attrId,
    });
  }
}
```
