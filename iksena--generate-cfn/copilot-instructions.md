## generate-cfn

> File path to save the generated YAML template


# CloudFormation Template Generation Skill

You are an expert AWS DevOps engineer. When this skill is invoked, you will:

1. Generate an AWS CloudFormation YAML template following the exact prompt
   structure defined in the **Generation Prompt** section below.
2. Save the template to the file path specified by `{{output}}`.
3. Call the `validate_cfn` skill on the saved file to validate it.
4. Report the outcome clearly using the **Output Format** section.

Do NOT delegate generation to another model or tool. You generate the template
yourself by following the instructions in this file.

---

## Generation Prompt

Assemble the prompt in the following order and use it to generate the template:

### System context (apply to your own reasoning)

> You are an expert in AWS CloudFormation template generation. Your task is to
> generate and improve templates based on feedback. Write your complete
> CloudFormation YAML template inside `<iac_template></iac_template>` tags.

### User message structure

Combine these three parts in order:

**TOP:**
```
You are an expert AWS DevOps engineer with extensive experience in creating
CloudFormation templates. Your task is to generate a valid, production ready
and deployable AWS CloudFormation YAML template based on the following
business need:

<business_need>
{{prompt}}
</business_need>
```

**BOTTOM (chain-of-thought instructions):**
```
Instructions:
1. Analyze the business need carefully.
2. Generate a complete CloudFormation YAML template that fulfills this need.
3. Be sure to fully specify all resources needed in the Resources section.
4. Ensure high accuracy and deployment success by following these guidelines:
   a. Start the template with 'AWSTemplateFormatVersion: 2010-09-09'.
   b. Include all necessary resources to meet the business need.
   c. Provide all required properties for each resource.
   d. Use proper YAML syntax and indentation (2-space).
   e. Follow AWS CloudFormation best practices and cfn-lint rules.
   f. End the template with the last property of the last resource.
5. Do not include any explanations, markdown formatting, or backticks in
   your output.

Before generating the final template, wrap your planning process in
<template_planning> tags. In this section:
1. List the key AWS services mentioned or implied in the business need.
2. Outline the main sections of the template (Parameters, Resources, Outputs).
3. Consider dependencies between resources and their ordering.
4. Think about any parameters or mappings needed for flexibility.
5. Consider outputs useful after stack creation.

After your planning, provide the complete CloudFormation YAML template as
your final output inside <iac_template></iac_template> tags.
```

**KNOWLEDGE INJECTION (append after BOTTOM — mandatory rules):**

```
--- CRITICAL RULES (must follow to avoid known deployment failures) ---

PARAMETERS & GREENFIELD DEPLOYMENTS:
- NEVER create Parameters for VpcId, SubnetId, SubnetIds, KeyName,
  KeyPairName, BucketName, S3BucketName, EmailAddress, or any value
  that requires user input at deploy time. This is the #1 cause of
  ValidationError deployment failures (8.3% of all errors observed).
- If VPC/Subnet is needed, CREATE them inline as AWS::EC2::VPC and
  AWS::EC2::Subnet resources. Never reference existing infrastructure.
- If EC2 KeyPair is needed, create AWS::EC2::KeyPair or use SSM Session
  Manager instead.

DEPRECATED / INVALID RUNTIMES — use these exact values:
- Lambda Python  → 'python3.12'   (python3.8 disabled 2025-02-28)
- Lambda Node.js → 'nodejs20.x'   (nodejs14.x disabled 2024-07-09)
- RDS MySQL      → one of: '8.0.32','8.0.36','8.0.39','8.0.40','8.4.3'
                   NOT '8.0' or '5.7'
- Aurora MySQL   → '8.0.mysql_aurora.3.07.0' or '8.0.mysql_aurora.3.08.0'

S3:
- Do NOT use the 'AccessControl' property; use AWS::S3::BucketPolicy.
- Set PublicAccessBlockConfiguration inside AWS::S3::Bucket — do NOT
  create 'AWS::S3::BucketPublicAccessBlock' (resource type does not exist).
- S3 event notifications go inside NotificationConfiguration in
  AWS::S3::Bucket — do NOT create 'AWS::S3::BucketNotification'.
- Always pair DeletionPolicy with UpdateReplacePolicy on stateful resources.

INTRINSIC FUNCTIONS:
- Use !Sub only when the string contains ${Variable} references.
- Do NOT use Fn::Sub on static strings (cfn-lint warns: 'Fn::Sub isn't
  needed because there are no variables').
- Always use AWSTemplateFormatVersion: '2010-09-09' (NOT '2010-09-01').

API GATEWAY:
- Valid IntegrationType values: 'AWS', 'AWS_PROXY', 'HTTP', 'HTTP_PROXY',
  'MOCK' — never lowercase ('aws', 'aws_proxy').

DYNAMODB:
- AttributeDefinitions must list ONLY attributes used in KeySchema or GSI.
- ReadCapacityUnits/WriteCapacityUnits are invalid inside GSI when
  BillingMode is PAY_PER_REQUEST.

SECURITY GROUPS:
- Omit FromPort/ToPort when IpProtocol is '-1' (all traffic).

IAM:
- Always use Version: '2012-10-17' in policy documents.
- Do NOT create AWS::IAM::RolePolicyAttachment — it does not exist; use
  inline Policies or AWS::IAM::ManagedPolicy + attach via ManagedPolicyArns.

RESOURCE TYPES THAT DO NOT EXIST (never generate):
- AWS::S3::BucketPublicAccessBlock
- AWS::S3::BucketNotification
- AWS::IAM::RolePolicyAttachment
- AWS::IAM::AccountPasswordPolicy
- AWS::ECR::RepositoryPolicy  (inline the policy in AWS::ECR::Repository)
- AWS::RDS::DBClusterInstance (use AWS::RDS::DBInstance + DBClusterIdentifier)

CIRCULAR DEPENDENCIES:
- Do not create explicit DependsOn loops. Use !GetAtt / !Ref to express
  dependencies implicitly.
```

---

## Few-Shot Examples

Study these verified, deployable templates from the
[IaCGen benchmark](https://github.com/Tianyi2/IaCGen/tree/main/Data/groud_truth/template)
as structural references. Do NOT copy them literally — use them to understand
correct property usage, resource ordering, and output patterns.

---

### EASY — Single resource, minimal configuration
**Prompt:** "We need a CloudFormation template that creates an AWS SQS queue
with basic configuration."
**Source:** [`sqs_easy.yaml`](https://github.com/Tianyi2/IaCGen/blob/main/Data/groud_truth/template/sqs_easy.yaml)
**Difficulty signals:** 1 resource, ~5 lines, no cross-resource refs, no outputs

```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: CloudFormation Template for single SQS.

Resources:
  MyQueue:
    Type: AWS::SQS::Queue
    Properties:
      VisibilityTimeout: 5
```

**What to observe:**
- Minimal `Properties` — include only what the business need requires
- No Parameters, no Outputs needed for a simple single-resource template
- `AWSTemplateFormatVersion` is a bare integer `2010-09-09`, not a quoted string

---

### MEDIUM — Multi-resource, cross-resource refs, DeletionPolicy, Outputs
**Prompt:** "Create an S3 bucket for static website hosting with a bucket
policy allowing public read access, a retention policy so the bucket is not
deleted with the stack, and output the website URL."
**Source:** [`s3_webhost_and_deletion_policy.yaml`](https://github.com/Tianyi2/IaCGen/blob/main/Data/groud_truth/template/s3_webhost_and_deletion_policy.yaml)
**Difficulty signals:** 2 resources, cross-resource !Ref/!Join, DeletionPolicy,
UpdateReplacePolicy, Outputs section

```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: CloudFormation Template for an Amazon S3 bucket for website
  hosting and with a DeletionPolicy.

Resources:
  S3Bucket:
    Type: 'AWS::S3::Bucket'
    Properties:
      PublicAccessBlockConfiguration:
        BlockPublicAcls: false
        BlockPublicPolicy: false
        IgnorePublicAcls: false
        RestrictPublicBuckets: false
      WebsiteConfiguration:
        IndexDocument: index.html
        ErrorDocument: error.html
    DeletionPolicy: Retain
    UpdateReplacePolicy: Retain

  BucketPolicy:
    Type: 'AWS::S3::BucketPolicy'
    Properties:
      PolicyDocument:
        Id: MyPolicy
        Version: 2012-10-17
        Statement:
          - Sid: PublicReadForGetBucketObjects
            Effect: Allow
            Principal: '*'
            Action: 's3:GetObject'
            Resource: !Join
              - ''
              - - 'arn:aws:s3:::'
                - !Ref S3Bucket
                - /*
      Bucket: !Ref S3Bucket

Outputs:
  WebsiteURL:
    Value: !GetAtt
      - S3Bucket
      - WebsiteURL
    Description: URL for website hosted on S3
  S3BucketSecureURL:
    Value: !Join
      - ''
      - - 'https://'
        - !GetAtt
          - S3Bucket
          - DomainName
    Description: Name of S3 bucket to hold website content
```

**What to observe:**
- `DeletionPolicy` and `UpdateReplacePolicy` are **siblings of Properties**, not inside it
- `PublicAccessBlockConfiguration` is inside `AWS::S3::Bucket` — not a separate resource
- `BucketPolicy` uses `AWS::S3::BucketPolicy`, NOT the deprecated `AccessControl` property
- `!Join` to build ARNs; `!GetAtt` in Outputs to surface deployment results
- IAM policy `Version` is `2012-10-17`

---

### HARD — Multi-service integration, IAM roles, Lambda + API Gateway
**Prompt:** "Create a REST API Gateway that triggers a Python Lambda function.
The Lambda should have an IAM execution role with CloudWatch Logs permissions.
Include the API deployment and a stage named 'prod'. Output the API endpoint URL."
**Source:** [`apigateway_lambda_integration.yaml`](https://github.com/Tianyi2/IaCGen/blob/main/Data/groud_truth/template/apigateway_lambda_integration.yaml)
**Difficulty signals:** 7 resources, IAM role, Lambda permission, API Gateway
Method + Integration + Deployment + Stage, circular-dependency avoidance,
cross-resource GetAtt/Ref chains

```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: REST API Gateway integrated with a Python Lambda function.

Resources:
  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: 'sts:AssumeRole'
      ManagedPolicyArns:
        - 'arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole'

  LambdaFunction:
    Type: AWS::Lambda::Function
    Properties:
      Runtime: python3.12
      Handler: index.handler
      Role: !GetAtt LambdaExecutionRole.Arn
      Code:
        ZipFile: |
          def handler(event, context):
              return {
                  'statusCode': 200,
                  'body': 'Hello from Lambda'
              }

  RestApi:
    Type: AWS::ApiGateway::RestApi
    Properties:
      Name: MyApi

  ApiResource:
    Type: AWS::ApiGateway::Resource
    Properties:
      RestApiId: !Ref RestApi
      ParentId: !GetAtt RestApi.RootResourceId
      PathPart: hello

  ApiMethod:
    Type: AWS::ApiGateway::Method
    Properties:
      RestApiId: !Ref RestApi
      ResourceId: !Ref ApiResource
      HttpMethod: GET
      AuthorizationType: NONE
      Integration:
        Type: AWS_PROXY
        IntegrationHttpMethod: POST
        Uri: !Sub
          - 'arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${LambdaArn}/invocations'
          - LambdaArn: !GetAtt LambdaFunction.Arn

  LambdaPermission:
    Type: AWS::Lambda::Permission
    Properties:
      FunctionName: !GetAtt LambdaFunction.Arn
      Action: 'lambda:InvokeFunction'
      Principal: apigateway.amazonaws.com
      SourceArn: !Sub 'arn:aws:execute-api:${AWS::Region}:${AWS::AccountId}:${RestApi}/*'

  ApiDeployment:
    Type: AWS::ApiGateway::Deployment
    DependsOn: ApiMethod
    Properties:
      RestApiId: !Ref RestApi

  ApiStage:
    Type: AWS::ApiGateway::Stage
    Properties:
      RestApiId: !Ref RestApi
      DeploymentId: !Ref ApiDeployment
      StageName: prod

Outputs:
  ApiEndpoint:
    Value: !Sub 'https://${RestApi}.execute-api.${AWS::Region}.amazonaws.com/prod/hello'
    Description: URL of the prod API endpoint
```

**What to observe:**
- Lambda runtime is `python3.12` — never `python3.8` or `python3.9`
- Integration `Type` is `AWS_PROXY` (uppercase) — never `aws_proxy`
- `LambdaExecutionRole` is referenced via `!GetAtt` in `LambdaFunction` — this
  expresses the dependency implicitly without circular DependsOn
- `ApiDeployment` uses explicit `DependsOn: ApiMethod` because there is no
  intrinsic reference from Deployment to Method — this is the one correct use case
- `ApiStage` references `ApiDeployment` via `!Ref` (implicit dependency), so no
  DependsOn needed there
- `!Sub` with a variable map `- LambdaArn: !GetAtt LambdaFunction.Arn` is the
  correct pattern for substituting GetAtt results into ARN strings

---

## Common Error Mitigations

These are the most frequent failure patterns observed across 9,158 errors
from 6 LLM models on the IaCGen benchmark
([source](https://github.com/iksena/llm-iac-data-analysis/tree/main/error_tracking)).
Memorise these before generating.

| # | Error | Gate | Fix |
|---|-------|------|-----|
| 1 | `Parameters: [KeyName/VpcId/...] must have values` | deployment | Never create these as Parameters; create the resources inline instead |
| 2 | `Fn::Sub isn't needed because there are no variables` | syntax | Only use `!Sub` when the string contains `${...}` references |
| 3 | `Both UpdateReplacePolicy and DeletionPolicy are needed` | syntax | Always pair them on stateful resources (S3, RDS, DynamoDB) |
| 4 | `AccessControl is a legacy property` | syntax | Use `AWS::S3::BucketPolicy` instead |
| 5 | `python3.8 was deprecated` | syntax | Use `python3.12` |
| 6 | `nodejs14.x was deprecated` | syntax | Use `nodejs20.x` |
| 7 | `AWS::S3::BucketPublicAccessBlock does not exist` | syntax | Use `PublicAccessBlockConfiguration` inside `AWS::S3::Bucket` |
| 8 | `AWSTemplateFormatVersion 2010-09-01 invalid` | syntax | Must be `2010-09-09` |
| 9 | `FromPort/ToPort ignored when IpProtocol is -1` | syntax | Omit them for all-traffic rules |
| 10 | `Additional properties not allowed ('Default' was unexpected)` | syntax | `Default` belongs under `Parameters/<Name>`, not inside `Properties` |
| 11 | `Circular dependency` | syntax | Express dependencies via `!Ref`/`!GetAtt`; only use `DependsOn` when there is no intrinsic reference (e.g. Deployment → Method) |
| 12 | `'aws' is not one of ['AWS','AWS_PROXY',...]` | syntax | API Gateway `Type` must be UPPERCASE |
| 13 | `Resource type '...' does not exist` | syntax | See the RESOURCE TYPES list above — never generate those types |
| 14 | `AttributeDefinitions / KeySchemas must match` | syntax | Only define attributes in `AttributeDefinitions` that appear in `KeySchema` or GSI `KeySchema` |
| 15 | `Fn::Select cannot select nonexistent value at index 1` | syntax/deploy | Validate that `Fn::GetAZs` + `Fn::Select` index exists for the target region |

---

## Execution Steps

When this skill is called with `{{prompt}}`:

1. **Plan** — Silently reason through the business need using `<template_planning>`.

2. **Generate** — Produce the YAML template using the Generation Prompt above,
   applying the Knowledge Injection rules and drawing on the Few-Shot Examples.
   The template must start with `AWSTemplateFormatVersion: 2010-09-09`.

3. **Extract** — Pull the content between `<iac_template>` and `</iac_template>`.

4. **Save** — Write the extracted YAML to `{{output}}`.

5. **Validate** — Call the `validate_cfn` skill:
   ```
   use skill validate_cfn with template={{output}} skip_deploy={{skip_deploy}}
   ```

6. **Report** — Display the result using the Output Format below.

---

## Output Format

After generation and validation, always report using this structure:

## CloudFormation Template Generation Result

**Status:** ✅ PASSED  |  ❌ FAILED
**Template saved to:** <path>
**Model used:** (your own model name, e.g. Claude 3.7 Sonnet)

### Validation Gates

| Gate | Result | Details |
|------|--------|---------|
| YAML syntax | ✅ / ❌ | <error count or "no errors"> |
| cfn-lint | ✅ / ❌ | <error count, key messages> |
| Live AWS deploy | ✅ / ❌ / ⏭ skipped | <stack id or error> |

### Generated Template

```yaml
<paste the full generated YAML here>
```

### What was built
<1-2 sentence plain-English summary of the resources created>

If validation **failed**, also include:

### Errors to fix
<list each validation error with a brief explanation of why it occurred
and how to fix it, referencing the Common Error Mitigations table above>

---

## Setup

The `validate_cfn` skill must be installed at:
```
~/.openclaw/workspace/skills/validate-cfn/
```

For live deployment validation, configure AWS credentials in your environment:
```bash
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_DEFAULT_REGION=ap-southeast-2
```

To skip live deployment (zero AWS cost, faster feedback):
```
use skill generate_cfn with prompt="..." skip_deploy=true
```

---
> Source: [iksena/generate_cfn](https://github.com/iksena/generate_cfn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
