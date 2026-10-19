# Node.js Lambda (CDK) Development Guidelines

Keep development changes focused on the JSON translation API. This service uses
AWS Lambda, API Gateway, Express for local development, Zod for validation, and
AWS Translate; it does not currently use DynamoDB or CRUD resources.

## Project Structure

```text
src/
├── api.ts
├── stack.ts
├── aws/
│   ├── gateway/translate.gateway.ts
│   └── lambda/
│       ├── common/api.dto.ts
│       └── translate/
│           ├── translate.dto.ts
│           ├── translate.lambda.ts
│           ├── translate.service.ts
│           └── translate.spec.ts
├── config/
│   ├── config.dotenv.ts
│   └── config.express.ts
├── constants/
│   ├── constants.environment.ts
│   ├── constants.messages.ts
│   └── constants.resources.ts
└── utils/
    ├── utils.aws.ts
    ├── utils.express.ts
    ├── utils.lambda.ts
    └── utils.zod.ts
```

## Development Rules

- Keep translation request schemas in `translate.dto.ts` and validate untrusted
  input with Zod before calling AWS Translate.
- Preserve the `POST /translate` route in both `src/stack.ts` (API Gateway) and
  `src/api.ts` (local Express), using `expressToLambdaEvent` for local parity.
- Keep API response messages in `constants.messages.ts` as strings. Place the
  selected value in the response DTO's `message` field; do not spread it as an
  object or assume it contains a `code` field.
- Use the response DTOs and helpers in `aws/lambda/common/api.dto.ts` and
  `utils/utils.lambda.ts` for consistent response bodies.
- When changing AWS Translate usage, update the Lambda IAM permissions in
  `src/stack.ts` and use the configured AWS region.
- Keep runtime configuration in `constants/constants.environment.ts`; do not
  commit `.env` or credentials.
- Keep changes scoped to the translator. Do not introduce CRUD, DynamoDB, or
  entity-model conventions unless the feature explicitly requires them.

## Tests and Quality

- Maintain Axios integration coverage in
  `src/aws/lambda/translate/translate.spec.ts` for successful translation and
  invalid request cases.
- Use `randomUUID()` in the submitted JSON values when test input needs to be
  unique. Assert the HTTP status and response shape rather than exact machine
  translation output.
- `TEST_API_URL` defaults to `http://localhost:3005`. Start the local API with
  `yarn dev`; integration tests need valid AWS credentials and Translate access.
  Set `TEST_API_URL` to the deployed API URL to test the deployed service.
- Run the relevant project checks after changes:

  ```bash
  yarn test
  yarn cdk-synt
  yarn format
  ```
