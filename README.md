# AI-powered anomaly detection for Session Manager logs

This sample shows how to automatically analyze AWS Systems Manager Session Manager transcripts using Amazon Bedrock. It classifies each session, alerts your security team on suspicious activity, and lets analysts query session history in natural language through Amazon Bedrock AgentCore.

This repository accompanies the AWS blog post [AI-powered anomaly detection for AWS Systems Manager Session Manager logs](<BLOG_POST_URL>).

> **Note:** This is sample code for demonstration and learning. Review and adapt it before using it in production. As part of deployment, the solution attaches an IAM policy to your managed node roles automatically through an AWS Systems Manager State Manager association.

## How it works

The solution has two modules. Module 1 analyzes each session automatically when it ends: it reads the transcript from Amazon S3, looks up the initiating identity and source IP in AWS CloudTrail, classifies the session with Amazon Bedrock (Normal, Suspicious, or Critical), stores a summary in Amazon DynamoDB, and sends an Amazon SNS alert for Suspicious or Critical sessions. Module 2 lets your Security Operations Center (SOC) team query session history on demand, in plain language, through an Amazon Bedrock AgentCore harness backed by Amazon DynamoDB, Amazon CloudWatch Logs, and AWS CloudTrail. Together they cover both automatic detection and hands-on investigation.

## Prerequisites

Before you deploy this solution, verify that you have the following:

- An AWS account with permissions to deploy AWS CloudFormation stacks.
- At least one managed node (Amazon EC2 instance or hybrid-activated server) with the SSM Agent installed and an IAM role that has the `AmazonSSMManagedInstanceCore` managed policy. The solution grants the additional session-logging permissions for you (see [How managed nodes get logging permissions](#how-managed-nodes-get-logging-permissions)).
- Access to an Amazon Bedrock foundation model (Claude Haiku 4.5 or your preferred model). For instructions, see [Add or remove access to Amazon Bedrock foundation models](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html).

Your managed nodes must be set up for Session Manager. That means the SSM Agent is installed, the node role has the required permissions, and the node has network access to the Systems Manager and Amazon S3 endpoints. Recent Amazon EC2 Amazon Linux and Windows AMIs already include the SSM Agent. For on-premises nodes and the full network and setup steps, see [Setting up Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-getting-started.html).

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

That is the full setup. Managed node permissions are applied automatically, so there is no manual policy step.

### How managed nodes get logging permissions

The template creates an AWS Systems Manager State Manager association that runs an automation runbook. The runbook finds your managed nodes, locates the IAM role attached to each node, and attaches the session-logging policy that the stack created. It runs on deploy, when a new node comes online, and on the schedule you set with `RolePermissionAutomationSchedule`. New nodes are covered automatically, so you do not have to edit node roles by hand.

The automation is scoped narrowly. It can attach only the one session-logging policy created by this stack, and nothing else.

### Parameters

| Parameter | Description | Default |
|---|---|---|
| `NotificationEndpoint` | Email address for security alerts | (required) |
| `NotificationProtocol` | Amazon SNS subscription protocol | `email` |
| `AnalysisModelId` | Amazon Bedrock model ID | `global.anthropic.claude-haiku-4-5-20251001-v1:0` |
| `RolePermissionAutomationSchedule` | How often the automation reapplies logging permissions to managed nodes | `rate(30 minutes)` |

### Outputs

The stack **Outputs** tab lists the transcript bucket name and ARN, the session log group, the summary table, the alert topic ARN, the KMS key ARN, both Lambda function ARNs, and the AgentCore harness ARN and gateway ID.

## Customize the classification

The model reviews the full session transcript, not just the commands entered. It also considers command output, the order of actions, and the overall context of the session. You control this behavior through the classification prompt in the analyzer Lambda function. Edit that prompt to adjust the criteria for each category, add your own rules, or define categories that fit your operational needs. You can also change the `AnalysisModelId` parameter to use a different foundation model.

## Test

**Module 1.** Start a Session Manager session, run a few commands, and check the summary table for a `Normal` classification. Start a second session, run unusual commands, and confirm you receive an alert.

**Module 2.** Open the harness in the Amazon Bedrock AgentCore console, choose **Test Harness**, and ask a question in plain language, such as "Show me all suspicious sessions from the past 24 hours."

## Clean up

Delete the stack to remove all resources:

```
aws cloudformation delete-stack --stack-name ssm-session-anomaly-detection
```

Before deleting, detach the session-logging managed policy from any node roles it was attached to. The automation attaches this policy to your node roles, and a managed policy cannot be deleted while it is still attached, so the stack delete fails otherwise. If deletion is still blocked, empty the S3 buckets first, then delete the stack again.

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for how to report a security issue.

## License

This library is licensed under the MIT-0 License. See the [LICENSE](LICENSE) file.
