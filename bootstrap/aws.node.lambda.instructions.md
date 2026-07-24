# Bootstrap: AWS Node.js Lambda API

Step-by-step guide to create a new Lambda REST API from zero using this project as template.

---

## Prerequisites

- Node.js 22.x
- Yarn
- AWS CLI configured (`aws configure`)
- AWS CDK CLI (`npm install -g aws-cdk`)
- AWS account with permissions to create Lambda, DynamoDB, API Gateway, IAM, CloudWatch

---

## 1. Project Initialization

```bash
mkdir my-service && cd my-service
yarn init -y
```

---

## 2. package.json

Replace the generated `package.json`:

```json
{
  "name": "@services/my-service",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "nodemon src/api.ts",
    "cdk": "cdk",
    "cdk-synt": "cdk synthesize",
    "cdk-deploy": "cdk deploy",
    "test": "jest",
    "test-w": "jest --watch",
    "test-cover": "jest --coverage",
    "format": "prettier -c -w ./src"
  },
  "devDependencies": {
    "@trivago/prettier-plugin-sort-imports": "^6.0.2",
    "@types/cors": "^2.8.17",
    "@types/express": "^5.0.6",
    "@types/jest": "^30.0.0",
    "@types/node": "^26.1.1",
    "aws-cdk": "2.1129.0",
    "body-parser": "^2.2.2",
    "cors": "^2.8.5",
    "dotenv": "^17.3.1",
    "express": "^5.2.1",
    "jest": "^30.2.0",
    "nodemon": "^3.1.0",
    "prettier": "^3.2.5",
    "ts-jest": "^29.4.11",
    "ts-node": "^10.9.2",
    "typescript": "~6.0.3"
  },
  "dependencies": {
    "aws-cdk-lib": "^2.238.0",
    "constructs": "^10.5.0",
    "dynamoose": "^4.2.0",
    "zod": "^4.4.3"
  },
  "peerDependencies": {
    "aws-cdk-lib": "2.137.0",
    "constructs": "^10.3.0"
  }
}
```

```bash
yarn install
```

---

## 3. TypeScript Config

**`tsconfig.json`**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["es2020", "dom"],
    "declaration": false,
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noImplicitThis": true,
    "alwaysStrict": true,
    "noUnusedLocals": false,
    "noUnusedParameters": false,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": false,
    "inlineSourceMap": true,
    "inlineSources": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "strictPropertyInitialization": false,
    "outDir": "dist/",
    "rootDir": ".",
    "types": ["node", "jest", "express"]
  },
  "ts-node": {
    "skipIgnore": true,
    "transpileOnly": true
  },
  "exclude": ["node_modules", "cdk.out", "dist"]
}
```

---

## 4. Prettier Config

**`.prettierrc`**

```json
{
  "tabWidth": 2,
  "useTabs": false,
  "printWidth": 100,
  "singleQuote": true,
  "arrowParens": "avoid",
  "semi": true,
  "importOrder": ["^src/(.*)$", "^server/(.*)$", "^[./]"],
  "importOrderSeparation": true,
  "importOrderSortSpecifiers": true,
  "plugins": ["@trivago/prettier-plugin-sort-imports"]
}
```

---

## 5. Jest Config

**`jest.config.ts`**

```typescript
import type { Config } from 'jest';

const config: Config = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  verbose: true,
  roots: ['./src/'],
};

export default config;
```

---

## 6. CDK Config

**`cdk.json`**

```json
{
  "app": "npx ts-node --prefer-ts-exts src/stack.ts",
  "watch": {
    "include": ["**"],
    "exclude": [
      "cdk*.json",
      "**/*.d.ts",
      "**/*.js",
      "tsconfig.json",
      "package*.json",
      "yarn.lock",
      "node_modules",
      "test"
    ]
  },
  "context": {
    "@aws-cdk/aws-lambda:recognizeLayerVersion": true,
    "@aws-cdk/core:checkSecretUsage": true,
    "@aws-cdk/core:target-partitions": ["aws", "aws-cn"],
    "@aws-cdk-containers/ecs-service-extensions:enableDefaultLogDriver": true,
    "@aws-cdk/aws-ec2:uniqueImdsv2TemplateName": true,
    "@aws-cdk/aws-ecs:arnFormatIncludesClusterName": true,
    "@aws-cdk/aws-iam:minimizePolicies": true,
    "@aws-cdk/core:validateSnapshotRemovalPolicy": true,
    "@aws-cdk/aws-codepipeline:crossAccountKeyAliasStackSafeResourceName": true,
    "@aws-cdk/aws-s3:createDefaultLoggingPolicy": true,
    "@aws-cdk/aws-sns-subscriptions:restrictSqsDescryption": true,
    "@aws-cdk/aws-apigateway:disableCloudWatchRole": true,
    "@aws-cdk/core:enablePartitionLiterals": true,
    "@aws-cdk/aws-events:eventsTargetQueueSameAccount": true,
    "@aws-cdk/aws-iam:standardizedServicePrincipals": true,
    "@aws-cdk/aws-ecs:disableExplicitDeploymentControllerForCircuitBreaker": true,
    "@aws-cdk/aws-iam:importedRoleStackSafeDefaultPolicyName": true,
    "@aws-cdk/aws-s3:serverAccessLogsUseBucketPolicy": true,
    "@aws-cdk/aws-route53-patters:useCertificate": true,
    "@aws-cdk/customresources:installLatestAwsSdkDefault": false,
    "@aws-cdk/aws-rds:databaseProxyUniqueResourceName": true,
    "@aws-cdk/aws-codedeploy:removeAlarmsFromDeploymentGroup": true,
    "@aws-cdk/aws-apigateway:authorizerChangeDeploymentLogicalId": true,
    "@aws-cdk/aws-ec2:launchTemplateDefaultUserData": true,
    "@aws-cdk/aws-secretsmanager:useAttachedSecretResourcePolicyForSecretTargetAttachments": true,
    "@aws-cdk/aws-redshift:columnId": true,
    "@aws-cdk/aws-stepfunctions-tasks:enableEmrServicePolicyV2": true,
    "@aws-cdk/aws-ec2:restrictDefaultSecurityGroup": true,
    "@aws-cdk/aws-apigateway:requestValidatorUniqueId": true,
    "@aws-cdk/aws-kms:aliasNameRef": true,
    "@aws-cdk/aws-autoscaling:generateLaunchTemplateInsteadOfLaunchConfig": true,
    "@aws-cdk/core:includePrefixInUniqueNameGeneration": true,
    "@aws-cdk/aws-efs:denyAnonymousAccess": true,
    "@aws-cdk/aws-opensearchservice:enableOpensearchMultiAzWithStandby": true,
    "@aws-cdk/aws-lambda-nodejs:useLatestRuntimeVersion": true,
    "@aws-cdk/aws-efs:mountTargetOrderInsensitiveLogicalId": true,
    "@aws-cdk/aws-rds:auroraClusterChangeScopeOfInstanceParameterGroupWithEachParameters": true,
    "@aws-cdk/aws-appsync:useArnForSourceApiAssociationIdentifier": true,
    "@aws-cdk/aws-rds:preventRenderingDeprecatedCredentials": true,
    "@aws-cdk/aws-codepipeline-actions:useNewDefaultBranchForCodeCommitSource": true,
    "@aws-cdk/aws-cloudwatch-actions:changeLambdaPermissionLogicalIdForLambdaAction": true,
    "@aws-cdk/aws-codepipeline:crossAccountKeysDefaultValueToFalse": true,
    "@aws-cdk/aws-codepipeline:defaultPipelineTypeToV2": true,
    "@aws-cdk/aws-kms:reduceCrossAccountRegionPolicyScope": true
  }
}
```

---

## 7. Environment Variables

**`.env`** — never commit this file

```env
STACK_NAME="my-service-dev"
TASK_TABLE_NAME="my-service-dev-task-table-v3"
TEST_API_URL="https://<your-api-id>.execute-api.<region>.amazonaws.com/prod"
AWS_REGION="us-east-1"
AWS_ACCESS_KEY_ID="<your-key-id>"
AWS_SECRET_ACCESS_KEY="<your-secret-key>"
```

Add `.env` to `.gitignore`.

---

## 8. Directory Structure

```
src/
├── api.ts                         # Express local dev server
├── stack.ts                       # CDK stack entry point
├── config/
│   └── dotenv.ts                  # Loads .env at runtime
├── constants/
│   ├── apiMessages.ts             # Standardized response codes/messages
│   ├── environment.ts             # Typed env vars object
│   └── resources.ts               # AWS resource name builder
├── utils/
│   ├── aws.ts                     # CORS preflight helpers
│   ├── express.ts                 # Express → Lambda event adapter
│   ├── lambda.ts                  # Response helpers and types
│   └── zod.ts                     # Zod DTO factory
└── lib/
    ├── dynamodb/
    │   └── <entity>.table.ts      # DynamoDB Table CDK construct
    ├── gateway/
    │   └── <entity>.gateway.ts    # API Gateway CDK construct
    └── lambda/
        ├── common/
        │   └── api.dto.ts         # SuccessDto / ErrorDto
        └── <entity>/
            ├── <entity>.lambda.ts # Lambda function CDK constructs
            ├── <entity>.service.ts # Lambda handlers (business logic)
            ├── <entity>.model.ts   # Dynamoose schema + model
            ├── <entity>.dto.ts     # Zod schemas + DTO classes
            └── <entity>.spec.ts    # Integration tests
```

---

## 9. Source Files

### `src/config/dotenv.ts`

```typescript
require('dotenv').config({ quiet: true }); // sort-imports-ignore
```

### `src/constants/environment.ts`

```typescript
const dotenv = {
  STACK_NAME: process.env.STACK_NAME || '',
  TEST_API_URL: process.env.TEST_API_URL || '',
  TASK_TABLE_NAME: process.env.TASK_TABLE_NAME || '',
  AWS_REGION: process.env.AWS_REGION || '',
  AWS_ACCESS_KEY_ID: process.env.AWS_ACCESS_KEY_ID || '',
  AWS_SECRET_ACCESS_KEY: process.env.AWS_SECRET_ACCESS_KEY || '',
};

export default dotenv;
```

Add a key per table/env var your service needs.

### `src/constants/resources.ts`

```typescript
import dotenv from './environment';

export const resourceNames = {
  taskTable: dotenv.STACK_NAME + '-task-table-v3',
  taskApiGateway: dotenv.STACK_NAME + '-task-api-gateway',
  logGroup: dotenv.STACK_NAME + '-log-group',
  findAllTaskLambda: dotenv.STACK_NAME + '-find-all-task-lambda',
  findOneTaskLambda: dotenv.STACK_NAME + '-find-one-task-lambda',
  createTaskLambda: dotenv.STACK_NAME + '-create-task-lambda',
  updateTaskLambda: dotenv.STACK_NAME + '-update-task-lambda',
  deleteTaskLambda: dotenv.STACK_NAME + '-delete-task-lambda',
};
```

### `src/constants/apiMessages.ts`

```typescript
export const apiMessages = {
  SUCCESS: { code: 'success', message: 'Success' },
  CREATED_SUCCESSFULLY: { code: 'created_successfully', message: 'Created successfully' },
  UPDATED_SUCCESSFULLY: { code: 'updated_successfully', message: 'Updated successfully' },
  DELETED_SUCCESSFULLY: { code: 'deleted_successfully', message: 'Deleted successfully' },
  FETCHED_SUCCESSFULLY: { code: 'fetched_successfully', message: 'Fetched successfully' },
  INVALID_REQUEST: { code: 'invalid_request', message: 'Invalid request' },
  INTERNAL_SERVER_ERROR: { code: 'internal_server_error', message: 'Internal server error' },
  NOT_FOUND: { code: 'not_found', message: 'Resource not found' },
  UNAUTHORIZED: { code: 'unauthorized', message: 'Unauthorized' },
  DATABASE_ERROR: { code: 'database_error', message: 'Database error' },
  VALIDATION_ERROR: { code: 'validation_error', message: 'Validation error' },
};
```

### `src/utils/lambda.ts`

```typescript
import { apiMessages } from '../constants/apiMessages';
import { ErrorDto } from '../lib/lambda/common/api.dto';

export type APIGatewayHandler = (event: any) => Promise<any>;

export type environment = { [key: string]: string };

export type APIGatewayResponse = {
  statusCode: number;
  body: string;
  headers: Record<string, string | number | boolean>;
};

export type CreateResponseOptions = (
  code: number,
  data: any,
  headers?: Record<string, string>,
) => APIGatewayResponse;

export const createResponse: CreateResponseOptions = (code, data, headers) => {
  return {
    statusCode: code,
    body: JSON.stringify(data),
    headers: {
      'Content-Type': 'application/json',
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Credentials': 'true',
      'Access-Control-Allow-Headers': 'Content-Type,Authorization,lang',
      'Access-Control-Allow-Methods': 'OPTIONS,GET,POST,PUT,PATCH,DELETE',
      ...headers,
    },
  };
};

export const badRequest = (details: unknown) => {
  return createResponse(400, ErrorDto.create({ ...apiMessages.INVALID_REQUEST, details }));
};

export const internalError = (err: any) => {
  console.error(err);
  return createResponse(
    500,
    ErrorDto.create({ details: err.message, ...apiMessages.INTERNAL_SERVER_ERROR }),
  );
};
```

### `src/utils/zod.ts`

```typescript
import { ZodObject, ZodRawShape, z } from 'zod';

type ZodDtoConstructor<T extends ZodRawShape> = {
  new (data: z.infer<ZodObject<T>>): z.infer<ZodObject<T>>;
  create(data: unknown): z.infer<ZodObject<T>>;
  schema: ZodObject<T>;
};

export function createZodDto<T extends ZodRawShape>(schema: ZodObject<T>): ZodDtoConstructor<T> {
  class ZodDto {
    static readonly schema = schema;

    constructor(data: z.infer<ZodObject<T>>) {
      Object.assign(this, data);
    }

    static create(data: unknown): z.infer<ZodObject<T>> {
      return schema.parse(data);
    }
  }

  return ZodDto as unknown as ZodDtoConstructor<T>;
}
```

### `src/utils/aws.ts`

```typescript
import * as cdk from 'aws-cdk-lib';
import * as gateway from 'aws-cdk-lib/aws-apigateway';

export const addPreflight = (resource: cdk.aws_apigateway.Resource) => {
  resource.addCorsPreflight({
    allowOrigins: ['http://localhost:3000'],
    allowMethods: gateway.Cors.ALL_METHODS,
    allowHeaders: [...gateway.Cors.DEFAULT_HEADERS, 'Cookie'],
    allowCredentials: true,
  });
};

export const addCorsPreflight = (resource: cdk.aws_apigateway.Resource) => {
  resource.addCorsPreflight({
    allowOrigins: gateway.Cors.ALL_ORIGINS,
    allowMethods: gateway.Cors.ALL_METHODS,
    allowHeaders: [...gateway.Cors.DEFAULT_HEADERS, 'lang'],
    allowCredentials: true,
  });
};
```

### `src/utils/express.ts`

```typescript
import { Request, Response } from 'express';

import { APIGatewayHandler } from './lambda';

export const expressToLambdaEvent = (lambda: APIGatewayHandler) => {
  return async (req: Request, res: Response) => {
    const lambdaResponse = await lambda({
      resource: req.path,
      path: req.path,
      httpMethod: req.method,
      queryStringParameters: req.query,
      multiValueQueryStringParameters: req.query,
      pathParameters: req.params,
      headers: {
        Authorization: req.headers.authorization,
        ...req.headers,
      },
      stageVariables: {},
      body: JSON.stringify(req.body),
      isBase64Encoded: false,
    });

    res
      .status(lambdaResponse.statusCode)
      .set(lambdaResponse.headers || {})
      .send(lambdaResponse.body);
  };
};
```

### `src/lib/lambda/common/api.dto.ts`

```typescript
import { z } from 'zod';

import { createZodDto } from '../../../utils/zod';

export class SuccessDto extends createZodDto(
  z.object({
    message: z.string(),
    code: z.string().optional(),
    data: z.any().optional(),
  }),
) {}

export class ErrorDto extends createZodDto(
  z.object({
    message: z.string(),
    code: z.string().optional(),
    details: z.any().optional(),
  }),
) {}
```

---

## 10. Entity Files (repeat per domain entity)

Replace `task` / `Task` with your entity name throughout.

### `src/lib/lambda/<entity>/<entity>.dto.ts`

```typescript
import { z } from 'zod';

import { createZodDto } from '../../../utils/zod';

export const taskSchema = {
  id: z.string().uuid(),
  name: z.string().min(1).max(200),
  description: z.string().optional(),
  status: z.string().default('pending'),
  priority: z.string().default('normal'),
  dueDate: z.string().optional(),
  progressList: z.array(z.string()).default([]),
  createdAt: z.string(),
};

export class GetTaskResponseDto extends createZodDto(
  z.object({
    id: taskSchema.id,
    name: taskSchema.name,
    description: taskSchema.description,
    status: taskSchema.status,
    priority: taskSchema.priority,
    dueDate: taskSchema.dueDate,
    progressList: taskSchema.progressList,
    createdAt: taskSchema.createdAt,
  }),
) {}

export class CreateTaskDto extends createZodDto(
  z.object({
    name: taskSchema.name,
    description: taskSchema.description,
    status: taskSchema.status,
    priority: taskSchema.priority,
    dueDate: taskSchema.dueDate,
    progressList: taskSchema.progressList,
  }),
) {}

export class UpdateTaskDto extends createZodDto(
  z
    .object({
      name: taskSchema.name,
      description: taskSchema.description,
      status: taskSchema.status,
      priority: taskSchema.priority,
      dueDate: taskSchema.dueDate,
      progressList: taskSchema.progressList,
    })
    .partial(),
) {}
```

### `src/lib/lambda/<entity>/<entity>.model.ts`

```typescript
import * as crypto from 'crypto';
import * as dynamoose from 'dynamoose';

import dotenv from '../../../constants/environment';

const schema = new dynamoose.Schema({
  id: { type: String, required: true, default: () => crypto.randomUUID() },
  name: { type: String, required: true },
  description: { type: String },
  status: { type: String, required: true, default: 'pending' },
  priority: { type: String, required: true, default: 'normal' },
  dueDate: { type: String },
  progressList: { type: Array, schema: [String], default: [] },
  createdAt: { type: String, required: true, default: () => new Date().toISOString() },
});

export class TaskModel {
  public readonly model = dynamoose.model('Tasks', schema, {
    tableName: dotenv.TASK_TABLE_NAME,
  });
}
```

### `src/lib/lambda/<entity>/<entity>.service.ts`

```typescript
import { z } from 'zod';

import { apiMessages } from '../../../constants/apiMessages';
import { APIGatewayHandler, badRequest, createResponse, internalError } from '../../../utils/lambda';
import { ErrorDto, SuccessDto } from '../common/api.dto';
import { CreateTaskDto, GetTaskResponseDto, UpdateTaskDto, taskSchema } from './task.dto';
import { TaskModel } from './task.model';

const taskModel = new TaskModel();

export const findAllTaskService: APIGatewayHandler = async () => {
  try {
    const tasks = await taskModel.model.scan().exec();
    if (!Array.isArray(tasks)) return tasks;

    const parsed = z.array(GetTaskResponseDto.schema).safeParse(tasks);
    if (!parsed.success) return badRequest(parsed.error.flatten());
    return createResponse(200, parsed.data);
  } catch (err: any) {
    return internalError(err);
  }
};

export const findOneTaskService: APIGatewayHandler = async event => {
  try {
    const pkResult = taskSchema.id.safeParse(event.pathParameters?.id);
    if (!pkResult.success) return badRequest(pkResult.error.flatten());

    const task = await taskModel.model.get({ id: pkResult.data });
    if (task && 'statusCode' in task) return task;
    if (!task) return createResponse(404, ErrorDto.create({ ...apiMessages.NOT_FOUND }));

    const parsed = GetTaskResponseDto.schema.safeParse(task);
    if (!parsed.success) return badRequest(parsed.error.flatten());
    return createResponse(200, parsed.data);
  } catch (err: any) {
    return internalError(err);
  }
};

export const createTaskService: APIGatewayHandler = async event => {
  try {
    const body = JSON.parse(event.body || '{}');
    const dtoResult = CreateTaskDto.schema.safeParse(body);
    if (!dtoResult.success) return badRequest(dtoResult.error.flatten());

    const task = await taskModel.model.create({ ...dtoResult.data });
    if ('statusCode' in task) return task;

    const parsed = GetTaskResponseDto.schema.safeParse(task);
    if (!parsed.success) return badRequest(parsed.error.flatten());
    return createResponse(200, parsed.data);
  } catch (err: any) {
    return internalError(err);
  }
};

export const updateTaskService: APIGatewayHandler = async event => {
  try {
    const pkResult = taskSchema.id.safeParse(event.pathParameters?.id);
    if (!pkResult.success) return badRequest(pkResult.error.flatten());

    const body = JSON.parse(event.body || '{}');
    const dtoResult = UpdateTaskDto.schema.safeParse(body);
    if (!dtoResult.success) return badRequest(dtoResult.error.flatten());

    const task = await taskModel.model.get({ id: pkResult.data });
    if (task && 'statusCode' in task) return task;
    if (!task) return createResponse(404, ErrorDto.create({ ...apiMessages.NOT_FOUND }));

    const updateData = Object.fromEntries(
      Object.entries(dtoResult.data).filter(([, v]) => v !== undefined),
    );

    const updateResult = await taskModel.model.update({ id: pkResult.data }, updateData);
    if (updateResult && 'statusCode' in updateResult) return updateResult;

    const updated = await taskModel.model.get({ id: pkResult.data });
    if (updated && 'statusCode' in updated) return updated;

    const parsed = GetTaskResponseDto.schema.safeParse(updated);
    if (!parsed.success) return badRequest(parsed.error.flatten());
    return createResponse(200, parsed.data);
  } catch (err: any) {
    return internalError(err);
  }
};

export const deleteTaskService: APIGatewayHandler = async event => {
  try {
    const pkResult = taskSchema.id.safeParse(event.pathParameters?.id);
    if (!pkResult.success) return badRequest(pkResult.error.flatten());

    const task = await taskModel.model.get({ id: pkResult.data });
    if (task && 'statusCode' in task) return task;
    if (!task) return createResponse(404, ErrorDto.create({ ...apiMessages.NOT_FOUND }));

    await taskModel.model.delete({ id: pkResult.data });
    return createResponse(200, SuccessDto.create({ ...apiMessages.DELETED_SUCCESSFULLY }));
  } catch (err: any) {
    return internalError(err);
  }
};
```

### `src/lib/lambda/<entity>/<entity>.lambda.ts`

```typescript
import * as cdk from 'aws-cdk-lib';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as nodeLambda from 'aws-cdk-lib/aws-lambda-nodejs';

import { resourceNames } from '../../../constants/resources';
import { environment } from '../../../utils/lambda';

const nodeModules = ['dynamoose', 'zod'];

const entry = __dirname + '/task.service.ts';
const runtime = lambda.Runtime.NODEJS_22_X;
const timeout = cdk.Duration.seconds(10);

export class FindAllTaskLambda extends nodeLambda.NodejsFunction {
  constructor(scope: cdk.Stack, environment: environment, logGroup?: cdk.aws_logs.LogGroup) {
    super(scope, resourceNames.findAllTaskLambda, {
      runtime, timeout, entry, environment, logGroup,
      handler: 'findAllTaskService',
      functionName: resourceNames.findAllTaskLambda,
      bundling: { environment, nodeModules },
    });
  }
}

export class FindOneTaskLambda extends nodeLambda.NodejsFunction {
  constructor(scope: cdk.Stack, environment: environment, logGroup?: cdk.aws_logs.LogGroup) {
    super(scope, resourceNames.findOneTaskLambda, {
      runtime, timeout, entry, environment, logGroup,
      handler: 'findOneTaskService',
      functionName: resourceNames.findOneTaskLambda,
      bundling: { environment, nodeModules },
    });
  }
}

export class CreateTaskLambda extends nodeLambda.NodejsFunction {
  constructor(scope: cdk.Stack, environment: environment, logGroup?: cdk.aws_logs.LogGroup) {
    super(scope, resourceNames.createTaskLambda, {
      runtime, timeout, entry, environment, logGroup,
      handler: 'createTaskService',
      functionName: resourceNames.createTaskLambda,
      bundling: { environment, nodeModules },
    });
  }
}

export class UpdateTaskLambda extends nodeLambda.NodejsFunction {
  constructor(scope: cdk.Stack, environment: environment, logGroup?: cdk.aws_logs.LogGroup) {
    super(scope, resourceNames.updateTaskLambda, {
      runtime, timeout, entry, environment, logGroup,
      handler: 'updateTaskService',
      functionName: resourceNames.updateTaskLambda,
      bundling: { environment, nodeModules },
    });
  }
}

export class DeleteTaskLambda extends nodeLambda.NodejsFunction {
  constructor(scope: cdk.Stack, environment: environment, logGroup?: cdk.aws_logs.LogGroup) {
    super(scope, resourceNames.deleteTaskLambda, {
      runtime, timeout, entry, environment, logGroup,
      handler: 'deleteTaskService',
      functionName: resourceNames.deleteTaskLambda,
      bundling: { environment, nodeModules },
    });
  }
}
```

### `src/lib/dynamodb/<entity>.table.ts`

```typescript
import * as cdk from 'aws-cdk-lib';
import * as dynamodb from 'aws-cdk-lib/aws-dynamodb';

import { resourceNames } from '../../constants/resources';

export class TaskTable {
  public table: dynamodb.Table;

  constructor(scope: cdk.Stack) {
    this.table = new dynamodb.Table(scope, resourceNames.taskTable, {
      tableName: resourceNames.taskTable,
      partitionKey: {
        name: 'id',
        type: dynamodb.AttributeType.STRING,
      },
      billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
      removalPolicy: cdk.RemovalPolicy.DESTROY,
    });
  }
}
```

### `src/lib/gateway/<entity>.gateway.ts`

```typescript
import * as cdk from 'aws-cdk-lib';
import * as gateway from 'aws-cdk-lib/aws-apigateway';

import { resourceNames } from '../../constants/resources';

export class TaskAPIGateway extends gateway.RestApi {
  constructor(scope: cdk.Stack) {
    super(scope, resourceNames.taskApiGateway, {
      restApiName: resourceNames.taskApiGateway,
    });
  }
}
```

---

## 11. Stack Entry Point

### `src/stack.ts`

```typescript
import './config/dotenv'; // sort-imports-ignore

import * as cdk from 'aws-cdk-lib';
import * as gateway from 'aws-cdk-lib/aws-apigateway';

import dotenv from './constants/environment';
import { resourceNames } from './constants/resources';
import { TaskTable } from './lib/dynamodb/task.table';
import { TaskAPIGateway } from './lib/gateway/task.gateway';
import * as TaskLambdas from './lib/lambda/tasks/task.lambda';
import { addCorsPreflight } from './utils/aws';
import { environment } from './utils/lambda';

export class NodeTemplateStack extends cdk.Stack {
  constructor(scope: cdk.App, props?: cdk.StackProps) {
    super(scope, dotenv.STACK_NAME, props);

    const taskTable = new TaskTable(this);

    const logGroup = new cdk.aws_logs.LogGroup(this, resourceNames.logGroup, {
      logGroupName: resourceNames.logGroup,
      retention: cdk.aws_logs.RetentionDays.ONE_WEEK,
      removalPolicy: cdk.RemovalPolicy.DESTROY,
    });

    const lambdaEnv: environment = {
      STACK_NAME: dotenv.STACK_NAME,
      TASK_TABLE_NAME: taskTable.table.tableName,
    };

    const createTaskLambda = new TaskLambdas.CreateTaskLambda(this, lambdaEnv, logGroup);
    const findAllTaskLambda = new TaskLambdas.FindAllTaskLambda(this, lambdaEnv, logGroup);
    const findOneTaskLambda = new TaskLambdas.FindOneTaskLambda(this, lambdaEnv, logGroup);
    const updateTaskLambda = new TaskLambdas.UpdateTaskLambda(this, lambdaEnv, logGroup);
    const deleteTaskLambda = new TaskLambdas.DeleteTaskLambda(this, lambdaEnv, logGroup);

    const taskApi = new TaskAPIGateway(this);

    // /tasks
    const taskRoute = taskApi.root.addResource('tasks');
    taskRoute.addMethod('POST', new gateway.LambdaIntegration(createTaskLambda));
    taskRoute.addMethod('GET', new gateway.LambdaIntegration(findAllTaskLambda));

    // /tasks/{id}
    const taskIdRoute = taskRoute.addResource('{id}');
    taskIdRoute.addMethod('GET', new gateway.LambdaIntegration(findOneTaskLambda));
    taskIdRoute.addMethod('PUT', new gateway.LambdaIntegration(updateTaskLambda));
    taskIdRoute.addMethod('DELETE', new gateway.LambdaIntegration(deleteTaskLambda));

    taskTable.table.grantReadWriteData(createTaskLambda);
    taskTable.table.grantReadWriteData(findAllTaskLambda);
    taskTable.table.grantReadWriteData(findOneTaskLambda);
    taskTable.table.grantReadWriteData(updateTaskLambda);
    taskTable.table.grantReadWriteData(deleteTaskLambda);

    addCorsPreflight(taskRoute);
    addCorsPreflight(taskIdRoute);
  }
}

const app = new cdk.App();
new NodeTemplateStack(app);
```

---

## 12. Local Dev Server

### `src/api.ts`

```typescript
import './config/dotenv'; // sort-imports-ignore

import * as bodyparser from 'body-parser';
import { expressToLambdaEvent } from './utils/express';
import * as taskService from './lib/lambda/tasks/task.service';

const express = require('express');
const cors = require('cors');

const localRoutes = () => {
  const router = express.Router();
  router.get('/tasks', expressToLambdaEvent(taskService.findAllTaskService));
  router.get('/tasks/:id', expressToLambdaEvent(taskService.findOneTaskService));
  router.post('/tasks', expressToLambdaEvent(taskService.createTaskService));
  router.put('/tasks/:id', expressToLambdaEvent(taskService.updateTaskService));
  router.delete('/tasks/:id', expressToLambdaEvent(taskService.deleteTaskService));
  return router;
};

const localApi = async () => {
  const app = express();
  const port = 3005;

  app.use(bodyparser.urlencoded({ extended: false }));
  app.use(bodyparser.json());
  app.use(cors({ credentials: true, origin: (_: any, callback: any) => callback(null, true) }));
  app.use(localRoutes());
  app.listen(port, () => console.log('Running at: http://localhost:' + port));
};

localApi();
```

---

## 13. Integration Tests

### `src/lib/lambda/<entity>/<entity>.spec.ts`

```typescript
import '../../../config/dotenv'; // sort-imports-ignore

import axios from 'axios';
import { randomUUID } from 'crypto';

import dotenv from '../../../constants/environment';
import { apiMessages } from '../../../constants/apiMessages';
import { CreateTaskDto, UpdateTaskDto } from './task.dto';

describe('Tasks API', () => {
  const baseURL = dotenv.TEST_API_URL;
  if (!baseURL) throw new Error('missing TEST_API_URL');
  const apiClient = axios.create({ baseURL });

  const testId = randomUUID().replace(/-/g, '');
  const createTaskDto: CreateTaskDto = {
    name: `Test Task ${testId}`,
    description: `Description for task ${testId}`,
    status: 'pending',
    priority: 'normal',
    dueDate: '2026-12-31',
    progressList: [],
  };

  let createdTaskId: string;

  it('should create a new task and return 200 status', async () => {
    const response = await apiClient.post('/tasks', createTaskDto);
    expect(response.status).toBe(200);
    expect(response.data).toMatchObject({
      name: createTaskDto.name,
      status: createTaskDto.status,
    });
    createdTaskId = response.data.id;
    expect(createdTaskId).toBeDefined();
  });

  it('should return 400 status for invalid task data', async () => {
    await expect(apiClient.post('/tasks', { name: '' })).rejects.toMatchObject({
      response: { status: 400 },
    });
  });

  it('should list all tasks and return 200 status', async () => {
    const response = await apiClient.get('/tasks');
    expect(response.status).toBe(200);
    expect(Array.isArray(response.data)).toBe(true);
  });

  it('should get a task by id and return 200 status', async () => {
    const response = await apiClient.get(`/tasks/${createdTaskId}`);
    expect(response.status).toBe(200);
    expect(response.data).toMatchObject({ id: createdTaskId });
  });

  it('should return 400 status for invalid task id', async () => {
    await expect(apiClient.get('/tasks/not-a-valid-uuid')).rejects.toMatchObject({
      response: { status: 400 },
    });
  });

  it('should return 404 status for non-existent task', async () => {
    await expect(apiClient.get(`/tasks/${randomUUID()}`)).rejects.toMatchObject({
      response: { status: 404 },
    });
  });

  it('should update a task and return 200 status', async () => {
    const updateDto: UpdateTaskDto = { name: `Updated Task ${testId}`, status: 'in_progress' };
    const response = await apiClient.put(`/tasks/${createdTaskId}`, updateDto);
    expect(response.status).toBe(200);
    expect(response.data).toMatchObject({ id: createdTaskId, ...updateDto });
  });

  it('should return 404 when updating a non-existent task', async () => {
    await expect(
      apiClient.put(`/tasks/${randomUUID()}`, { name: 'Ghost' }),
    ).rejects.toMatchObject({ response: { status: 404 } });
  });

  it('should delete a task and return 200 status', async () => {
    const response = await apiClient.delete(`/tasks/${createdTaskId}`);
    expect(response.status).toBe(200);
    expect(response.data).toEqual({ ...apiMessages.DELETED_SUCCESSFULLY });
  });

  it('should return 404 when deleting a non-existent task', async () => {
    await expect(apiClient.delete(`/tasks/${randomUUID()}`)).rejects.toMatchObject({
      response: { status: 404 },
    });
  });

  it('should clean up any test data', async () => {
    if (!createdTaskId) return;
    await apiClient.delete(`/tasks/${createdTaskId}`).catch(() => undefined);
  });
});
```

> Tests run against the deployed `TEST_API_URL`. Set it in `.env` after deploying.

---

## 14. CDK Bootstrap & Deploy

```bash
# First-time CDK bootstrap (once per AWS account/region)
cdk bootstrap aws://<account-id>/<region>

# Verify the stack compiles
yarn cdk-synt

# Deploy to AWS
yarn cdk-deploy
```

After deploy, copy the API Gateway URL from the output and set it as `TEST_API_URL` in `.env`.

---

## 15. Development Workflow

```bash
# Local dev (port 3005, uses .env credentials to hit real DynamoDB)
yarn dev

# Run integration tests against deployed API
yarn test

# Coverage report
yarn test-cover

# Format source files
yarn format
```

---

## 16. Conventions

| Rule | Detail |
|------|--------|
| Resource names | All derived from `STACK_NAME` prefix via `resourceNames` |
| Lambda handlers | Named exports in `<entity>.service.ts`, one per HTTP verb |
| Validation | All input/output validated through Zod schemas in `<entity>.dto.ts` |
| Error responses | Always use `badRequest()` or `internalError()` from `utils/lambda.ts` |
| Node modules bundled | List extra npm deps in `nodeModules` array inside `<entity>.lambda.ts` |
| Local dev | Express routes mirror Lambda routes via `expressToLambdaEvent()` adapter |
| Tests | Integration only — hit the real deployed API, no mocks |
| Log retention | 1 week (`RetentionDays.ONE_WEEK`) with auto-destroy |
| Table billing | On-demand (`PAY_PER_REQUEST`) |
| Runtime | Node.js 22.x |
| Timeout | 10 seconds per Lambda |

---

## 17. Adding a New Entity

1. Create `src/lib/lambda/<entity>/` with four files: `.dto.ts`, `.model.ts`, `.service.ts`, `.lambda.ts`, `.spec.ts`
2. Add resource names for the entity to `src/constants/resources.ts`
3. Add table name env var to `src/constants/environment.ts` and `.env`
4. Create `src/lib/dynamodb/<entity>.table.ts`
5. Create `src/lib/gateway/<entity>.gateway.ts` (or reuse existing gateway)
6. Wire lambdas, routes, permissions, and CORS in `src/stack.ts`
7. Add routes to `src/api.ts` for local dev
