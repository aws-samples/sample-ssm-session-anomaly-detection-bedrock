# AI-powered anomaly detection for Session Manager logs

This sample shows how to automatically analyze AWS Systems Manager Session Manager transcripts using Amazon Bedrock. It classifies each session, alerts your security team on suspicious activity, and lets analysts query session history in natural language through Amazon Bedrock AgentCore.



## How it works

The solution has two modules.

The solution has two modules. Module 1 analyzes each session automatically when it ends and flags anything Suspicious or Critical. Module 2 lets your Security Operations Center (SOC) team query session history on demand, in plain language. Together they cover both automatic detection and hands-on investigation.

## Prerequisites

- An AWS account with permissions to deploy AWS CloudFormation stacks.
- Access to an Amazon Bedrock foundation model (for example, Claude Haiku 4.5). See [Add or remove access to Amazon Bedrock foundation models](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html).
- At least one managed node configured for Session Manager. See [Setting up Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-getting-started.html).

## Deploy

1. Deploy the template:

   ```
   aws cloudformation deploy \
     --template-file template.yaml \
     --stack-name ssm-session-anomaly-detection \
     --capabilities CAPABILITY_NAMED_IAM \
     --parameter-overrides NotificationEndpoint=you@example.com
   ```

2. Confirm the Amazon SNS subscription email you receive.
3. Attach the generated instance-role policy to your managed node's IAM role. Copy it from the `InstanceRolePolicySnippet` value on the stack **Outputs** tab.

### Parameters

| Parameter | Description | Default |
|---|---|---|
| `NotificationEndpoint` | Email address for security alerts | (required) |
| `NotificationProtocol` | Amazon SNS subscription protocol | `email` |
| `AnalysisModelId` | Amazon Bedrock model ID | `global.anthropic.claude-haiku-4-5-20251001-v1:0` |

### Outputs

The stack **Outputs** tab lists the transcript bucket, session log group, summary table, alert topic, KMS key, both Lambda functions, the AgentCore gateway and harness, and a ready-to-attach IAM policy snippet.

## Test

**Module 1.** Start a Session Manager session, run a few commands, and check the summary table for a `Normal` classification. Start a second session, run suspicious commands, and confirm you receive an alert.

**Module 2.** Open the harness in the Amazon Bedrock AgentCore console, choose **Test Harness**, and ask a question in plain language, such as "Show me all suspicious sessions from the past 24 hours."

## Clean up

Delete the stack to remove all resources:

```
aws cloudformation delete-stack --stack-name ssm-session-anomaly-detection
```

If deletion is blocked, empty the S3 buckets first, then delete the stack again.

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for how to report a security issue.

## License

This library is licensed under the MIT-0 License. See the [LICENSE](LICENSE) file.
