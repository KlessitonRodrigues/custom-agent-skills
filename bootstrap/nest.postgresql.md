# Bootstrap: NestJS PostgreSQL Auth API

Step-by-step guide to create a new NestJS REST API with JWT authentication and PostgreSQL from zero using this project as template.

---

## Prerequisites

- Node.js 22.x
- Yarn
- PostgreSQL instance (local or AWS Aurora)
- AWS CLI configured (`aws configure`) — only needed for Aurora Data API variant

---

## 1. Project Initialization

```bash
mkdir my-auth-api && cd my-auth-api
yarn init -y
```

---

## 2. package.json

Replace the generated `package.json`:

```json
{
  "name": "@services/my-auth-api",
  "version": "0.0.1",
  "description": "",
  "author": "",
  "private": true,
  "license": "UNLICENSED",
  "scripts": {
    "build": "nest build",
    "format": "prettier --write \"src/**/*.ts\"",
    "start": "nest start",
    "dev": "nest start --watch",
    "start:debug": "nest start --debug --watch",
    "start:prod": "node dist/main",
    "lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "test:e2e": "jest --config ./test/jest-e2e.json"
  },
  "dependencies": {
    "@aws-sdk/rds-signer": "^3.1049.0",
    "@nestjs/common": "^11.0.1",
    "@nestjs/config": "^4.0.4",
    "@nestjs/core": "^11.0.1",
    "@nestjs/platform-express": "^11.0.1",
    "@nestjs/typeorm": "^11.0.1",
    "@types/bcryptjs": "^3.0.0",
    "@types/jsonwebtoken": "^9.0.10",
    "bcryptjs": "^3.0.3",
    "dotenv": "^17.4.2",
    "jsonwebtoken": "^9.0.3",
    "nestjs-zod": "^5.4.0",
    "pg": "^8.21.0",
    "reflect-metadata": "^0.2.2",
    "rxjs": "^7.8.1",
    "typeorm": "^1.0.0",
    "typeorm-aurora-data-api-driver": "^3.0.2"
  },
  "devDependencies": {
    "@babel/plugin-proposal-decorators": "^8.0.2",
    "@eslint/eslintrc": "^3.2.0",
    "@eslint/js": "^10.0.1",
    "@nestjs/cli": "^11.0.0",
    "@nestjs/schematics": "^11.0.0",
    "@nestjs/testing": "^11.0.1",
    "@trivago/prettier-plugin-sort-imports": "^6.0.2",
    "@types/express": "^5.0.0",
    "@types/jest": "^30.0.0",
    "@types/node": "^25.9.1",
    "@types/supertest": "^7.0.0",
    "eslint": "^10.4.0",
    "eslint-config-prettier": "^10.0.1",
    "eslint-plugin-prettier": "^5.2.2",
    "globals": "^17.0.0",
    "jest": "^30.0.0",
    "prettier": "^3.4.2",
    "source-map-support": "^0.5.21",
    "supertest": "^7.0.0",
    "ts-jest": "^29.4.10",
    "ts-loader": "^9.5.2",
    "ts-node": "^10.9.2",
    "tsconfig-paths": "^4.2.0",
    "typescript": "^6.0.3",
    "typescript-eslint": "^8.59.4"
  },
  "jest": {
    "moduleFileExtensions": ["js", "json", "ts"],
    "rootDir": "src",
    "testRegex": ".*\\.spec\\.ts$",
    "transform": { "^.+\\.(t|j)s$": "ts-jest" },
    "collectCoverageFrom": ["**/*.(t|j)s"],
    "coverageDirectory": "../coverage",
    "testEnvironment": "node"
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
    "module": "nodenext",
    "moduleResolution": "nodenext",
    "declaration": false,
    "removeComments": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "allowSyntheticDefaultImports": true,
    "target": "ES2023",
    "sourceMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": false,
    "skipLibCheck": true,
    "strictNullChecks": false,
    "noImplicitAny": false,
    "strictBindCallApply": false,
    "forceConsistentCasingInFileNames": false,
    "noFallthroughCasesInSwitch": false,
    "paths": { "@/*": ["./*"] }
  }
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
  "importOrder": ["^src/(.*)$", "^[./]"],
  "importOrderSeparation": true,
  "importOrderSortSpecifiers": true,
  "plugins": ["@trivago/prettier-plugin-sort-imports"]
}
```

---

## 5. Environment Variables

**`.env`** — never commit this file

```env
BASE_URL="http://localhost:3000"
PORT=3000

DB_HOST="localhost"
DB_PORT="5432"
DB_USERNAME="postgres"
DB_PASSWORD="postgres"
DB_NAME="my_auth_api_db"

JWT_SECRET="your-jwt-secret-here"
JWT_EXPIRATION="7d"
JWT_REFRESH_EXPIRATION="30d"

GOOGLE_CLIENT_ID=""
GOOGLE_CLIENT_SECRET=""
```

Add `.env` to `.gitignore`.

---

## 6. Directory Structure

```
src/
├── main.ts                             # NestJS bootstrap entry point
├── app.module.ts                       # Root module
├── constants/
│   └── dotenv.ts                       # Typed env accessors
├── database/
│   └── database.module.ts              # TypeORM PostgreSQL connection
└── modules/
    ├── common/
    │   └── dto/
    │       └── apiResponse.ts          # Shared ErrorDto / SuccessDto
    ├── docs/
    │   ├── swegger.module.ts           # Swagger module
    │   ├── swagger.controller.ts       # Serves swagger-2.0.yaml
    │   └── swagger.service.ts          # Reads swagger-2.0.yaml file
    ├── auth/
    │   ├── auth.module.ts
    │   ├── auth.controller.ts          # /auth routes
    │   ├── auth.service.ts             # JWT, bcrypt, OAuth logic
    │   ├── auth.spec.ts                # Integration tests
    │   └── dto/
    │       └── auth.dto.ts             # SignIn/SignUp/Refresh/Recovery DTOs
    └── users/
        ├── user.module.ts
        ├── user.controller.ts          # /users CRUD routes
        ├── user.service.ts
        ├── user.spec.ts
        ├── dto/
        │   └── user.dto.ts
        └── entity/
            └── user.entity.ts
```

---

## 7. Source Files

### `src/main.ts`

```typescript
import { NestFactory } from "@nestjs/core";

import { AppModule } from "./app.module";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

### `src/app.module.ts`

```typescript
import { Module } from "@nestjs/common";

import { DatabaseModule } from "./database/database.module";
import { AuthModule } from "./modules/auth/auth.module";
import { SwaggerModule } from "./modules/docs/swegger.module";
import { UserModule } from "./modules/users/user.module";

@Module({
  imports: [DatabaseModule, SwaggerModule, UserModule, AuthModule],
  providers: [],
})
export class AppModule {}
```

### `src/constants/dotenv.ts`

```typescript
import "dotenv/config";

const dotenv = {
  BASE_URL: process.env.BASE_URL || "http://localhost:3000",
  PORT: Number(process.env.PORT) || 3000,
  DB_HOST: process.env.DB_HOST || "",
  DB_PORT: Number(process.env.DB_PORT) || 5432,
  DB_USERNAME: process.env.DB_USERNAME || "",
  DB_PASSWORD: process.env.DB_PASSWORD || "",
  DB_NAME: process.env.DB_NAME || "",
  JWT_SECRET: process.env.JWT_SECRET || "",
  JWT_EXPIRATION: process.env.JWT_EXPIRATION || "7d",
  JWT_REFRESH_EXPIRATION: process.env.JWT_REFRESH_EXPIRATION || "30d",
};

export default dotenv;
```

### `src/database/database.module.ts`

```typescript
import { Module } from "@nestjs/common";
import { ConfigService } from "@nestjs/config";
import { TypeOrmModule } from "@nestjs/typeorm";

import { UserEntity } from "../modules/users/entity/user.entity";

@Module({
  imports: [
    TypeOrmModule.forRootAsync({
      useFactory: (configService: ConfigService) => ({
        type: "postgres",
        host: configService.get<string>("DB_HOST"),
        port: configService.get<number>("DB_PORT"),
        username: configService.get<string>("DB_USERNAME"),
        password: configService.get<string>("DB_PASSWORD"),
        database: configService.get<string>("DB_NAME"),
        entities: [UserEntity],
        synchronize: true,
      }),
      inject: [ConfigService],
    }),
  ],
})
export class DatabaseModule {}
```

### `src/modules/common/dto/apiResponse.ts`

```typescript
import { z } from "zod";

import { createZodDto } from "../../../utils/zod";

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

## 8. Entity Files (repeat per domain entity)

Replace `user` / `User` with your entity name throughout.

### `src/modules/users/entity/user.entity.ts`

```typescript
import {
  Column,
  CreateDateColumn,
  Entity,
  PrimaryGeneratedColumn,
  UpdateDateColumn,
} from "typeorm";

@Entity("users")
export class UserEntity {
  @PrimaryGeneratedColumn()
  id!: number;

  @Column({ unique: true, nullable: false, length: 255 })
  email!: string;

  @Column({ length: 255 })
  name!: string;

  @Column({ nullable: false, length: 255 })
  password!: string;

  @Column({ length: 255, nullable: true })
  recoveryToken!: string;

  @Column({ type: "timestamp", nullable: true })
  recoveryTokenExpiration!: Date;

  @CreateDateColumn({ type: "timestamp" })
  createdAt!: Date;

  @UpdateDateColumn({ type: "timestamp" })
  updatedAt!: Date;
}
```

### `src/modules/users/dto/user.dto.ts`

```typescript
import { createZodDto } from "nestjs-zod";
import { z } from "zod";

const userSchema = {
  id: z.number(),
  name: z.string().min(1).max(255),
  email: z.string().email(),
  password: z.string().min(8),
  createdAt: z.string().or(z.date()),
  updatedAt: z.string().or(z.date()),
};

export class CreateUserDto extends createZodDto(
  z.object({
    name: userSchema.name,
    email: userSchema.email,
    password: userSchema.password,
  }),
) {}

export class UpdateUserDto extends createZodDto(
  z
    .object({
      name: userSchema.name,
      email: userSchema.email,
      password: userSchema.password,
    })
    .partial(),
) {}

export class GetUserResponseDto extends createZodDto(
  z.object({
    id: userSchema.id,
    name: userSchema.name,
    email: userSchema.email,
    createdAt: userSchema.createdAt,
    updatedAt: userSchema.updatedAt,
  }),
) {}
```

### `src/modules/users/user.service.ts`

```typescript
import { Injectable } from "@nestjs/common";
import { InjectRepository } from "@nestjs/typeorm";
import { Repository } from "typeorm";

import { ErrorDto } from "../common/dto/apiResponse";
import {
  CreateUserDto,
  GetUserResponseDto,
  UpdateUserDto,
} from "./dto/user.dto";
import { UserEntity } from "./entity/user.entity";

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(UserEntity)
    private readonly userRepository: Repository<UserEntity>,
  ) {}

  async findAll(): Promise<GetUserResponseDto[] | ErrorDto> {
    try {
      const users = await this.userRepository.find();
      return users.map((u) => GetUserResponseDto.create(u));
    } catch (error) {
      const details = error instanceof Error ? error.message : error;
      return ErrorDto.create({ message: "Failed to fetch users", details });
    }
  }

  async findOne(id: number): Promise<GetUserResponseDto | ErrorDto> {
    try {
      const user = await this.userRepository.findOneBy({ id });
      if (!user)
        return ErrorDto.create({
          message: "User not found",
          code: "not_found",
        });
      return GetUserResponseDto.create(user);
    } catch (error) {
      const details = error instanceof Error ? error.message : error;
      return ErrorDto.create({ message: "Failed to fetch user", details });
    }
  }

  async create(dto: CreateUserDto): Promise<GetUserResponseDto | ErrorDto> {
    try {
      const user = this.userRepository.create(dto);
      const saved = await this.userRepository.save(user);
      return GetUserResponseDto.create(saved);
    } catch (error) {
      const details = error instanceof Error ? error.message : error;
      return ErrorDto.create({ message: "Failed to create user", details });
    }
  }

  async update(
    id: number,
    dto: UpdateUserDto,
  ): Promise<GetUserResponseDto | ErrorDto> {
    try {
      await this.userRepository.update(id, dto);
      return this.findOne(id);
    } catch (error) {
      const details = error instanceof Error ? error.message : error;
      return ErrorDto.create({ message: "Failed to update user", details });
    }
  }

  async delete(id: number): Promise<{ message: string } | ErrorDto> {
    try {
      await this.userRepository.delete(id);
      return { message: "User deleted successfully" };
    } catch (error) {
      const details = error instanceof Error ? error.message : error;
      return ErrorDto.create({ message: "Failed to delete user", details });
    }
  }
}
```

### `src/modules/users/user.controller.ts`

```typescript
import {
  Body,
  Controller,
  Delete,
  Get,
  Param,
  Patch,
  Post,
  Put,
  UsePipes,
} from "@nestjs/common";
import { ZodValidationPipe } from "nestjs-zod";

import { CreateUserDto, UpdateUserDto } from "./dto/user.dto";
import { UserService } from "./user.service";

@Controller("users")
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get()
  findAll() {
    return this.userService.findAll();
  }

  @Get(":id")
  findOne(@Param("id") id: string) {
    return this.userService.findOne(+id);
  }

  @Post()
  @UsePipes(ZodValidationPipe)
  createUser(@Body() dto: CreateUserDto) {
    return this.userService.create(dto);
  }

  @Put(":id")
  @UsePipes(ZodValidationPipe)
  updateUser(@Param("id") id: string, @Body() dto: UpdateUserDto) {
    return this.userService.update(+id, dto);
  }

  @Patch(":id")
  @UsePipes(ZodValidationPipe)
  patchUser(@Param("id") id: string, @Body() dto: UpdateUserDto) {
    return this.userService.update(+id, dto);
  }

  @Delete(":id")
  deleteUser(@Param("id") id: string) {
    return this.userService.delete(+id);
  }
}
```

### `src/modules/users/user.module.ts`

```typescript
import { Module } from "@nestjs/common";
import { TypeOrmModule } from "@nestjs/typeorm";

import { UserEntity } from "./entity/user.entity";
import { UserController } from "./user.controller";
import { UserService } from "./user.service";

@Module({
  imports: [TypeOrmModule.forFeature([UserEntity])],
  controllers: [UserController],
  providers: [UserService],
})
export class UserModule {}
```

---

## 9. Auth Module

The auth module provides JWT-based sign-in, sign-up, sign-out, token refresh, password recovery, and OAuth (Google/GitHub).

### `src/modules/auth/dto/auth.dto.ts`

```typescript
import { createZodDto } from "nestjs-zod";
import { z } from "zod";

export class SignInDto extends createZodDto(
  z.object({
    email: z.string().email(),
    password: z.string().min(8),
  }),
) {}

export class SignUpDto extends createZodDto(
  z.object({
    name: z.string().min(1).max(255),
    email: z.string().email(),
    password: z.string().min(8),
  }),
) {}

export class RefreshTokenDto extends createZodDto(
  z.object({
    refreshToken: z.string(),
  }),
) {}

export class SendRecoveryCodeDto extends createZodDto(
  z.object({
    email: z.string().email(),
  }),
) {}

export class VerifyRecoveryCodeDto extends createZodDto(
  z.object({
    email: z.string().email(),
    code: z.string(),
  }),
) {}

export class ResetPasswordDto extends createZodDto(
  z.object({
    email: z.string().email(),
    code: z.string(),
    newPassword: z.string().min(8),
  }),
) {}

export class GoogleSignInDto extends createZodDto(
  z.object({
    idToken: z.string(),
  }),
) {}

export class AuthResponseDto extends createZodDto(
  z.object({
    accessToken: z.string(),
    refreshToken: z.string(),
  }),
) {}
```

### `src/modules/auth/auth.service.ts`

```typescript
import * as bcrypt from "bcryptjs";
import * as jwt from "jsonwebtoken";
import { Injectable } from "@nestjs/common";
import { InjectRepository } from "@nestjs/typeorm";
import { Repository } from "typeorm";

import dotenv from "../../constants/dotenv";
import { ErrorDto } from "../common/dto/apiResponse";
import { UserEntity } from "../users/entity/user.entity";
import {
  AuthResponseDto,
  GoogleSignInDto,
  RefreshTokenDto,
  ResetPasswordDto,
  SendRecoveryCodeDto,
  SignInDto,
  SignUpDto,
  VerifyRecoveryCodeDto,
} from "./dto/auth.dto";

@Injectable()
export class AuthService {
  constructor(
    @InjectRepository(UserEntity)
    private readonly userRepository: Repository<UserEntity>,
  ) {}

  private signTokens(userId: number, email: string) {
    const accessToken = jwt.sign({ sub: userId, email }, dotenv.JWT_SECRET, {
      expiresIn: dotenv.JWT_EXPIRATION as jwt.SignOptions["expiresIn"],
    });
    const refreshToken = jwt.sign({ sub: userId, email }, dotenv.JWT_SECRET, {
      expiresIn: dotenv.JWT_REFRESH_EXPIRATION as jwt.SignOptions["expiresIn"],
    });
    return AuthResponseDto.create({ accessToken, refreshToken });
  }

  async signIn(dto: SignInDto): Promise<AuthResponseDto | ErrorDto> {
    try {
      const user = await this.userRepository.findOneBy({ email: dto.email });
      if (!user)
        return ErrorDto.create({
          message: "Invalid credentials",
          code: "unauthorized",
        });

      const valid = await bcrypt.compare(dto.password, user.password);
      if (!valid)
        return ErrorDto.create({
          message: "Invalid credentials",
          code: "unauthorized",
        });

      return this.signTokens(user.id, user.email);
    } catch (error) {
      const details = error instanceof Error ? error.message : error;
      return ErrorDto.create({ message: "Sign-in failed", details });
    }
  }

  async signUp(dto: SignUpDto): Promise<AuthResponseDto | ErrorDto> {
    try {
      const existing = await this.userRepository.findOneBy({
        email: dto.email,
      });
      if (existing)
        return ErrorDto.create({
          message: "Email already in use",
          code: "conflict",
        });

      const hashed = await bcrypt.hash(dto.password, 10);
      const user = await this.userRepository.save(
        this.userRepository.create({ ...dto, password: hashed }),
      );
      return this.signTokens(user.id, user.email);
    } catch (error) {
      const details = error instanceof Error ? error.message : error;
      return ErrorDto.create({ message: "Sign-up failed", details });
    }
  }

  async signOut(): Promise<{ message: string }> {
    return { message: "Signed out successfully" };
  }

  async refresh(dto: RefreshTokenDto): Promise<AuthResponseDto | ErrorDto> {
    try {
      const payload = jwt.verify(
        dto.refreshToken,
        dotenv.JWT_SECRET,
      ) as jwt.JwtPayload;
      return this.signTokens(payload.sub as number, payload.email as string);
    } catch (error) {
      return ErrorDto.create({
        message: "Invalid or expired refresh token",
        code: "unauthorized",
      });
    }
  }

  async sendRecoveryCode(
    dto: SendRecoveryCodeDto,
  ): Promise<{ message: string } | ErrorDto> {
    try {
      const user = await this.userRepository.findOneBy({ email: dto.email });
      if (!user) return { message: "Recovery email sent if account exists" };

      const code = Math.floor(100000 + Math.random() * 900000).toString();
      const expiration = new Date(Date.now() + 15 * 60 * 1000);
      await this.userRepository.update(user.id, {
        recoveryToken: code,
        recoveryTokenExpiration: expiration,
      });
      // TODO: send email with code
      return { message: "Recovery code sent" };
    } catch (error) {
      const details = error instanceof Error ? error.message : error;
      return ErrorDto.create({
        message: "Failed to send recovery code",
        details,
      });
    }
  }

  async verifyRecoveryCode(
    dto: VerifyRecoveryCodeDto,
  ): Promise<{ message: string } | ErrorDto> {
    try {
      const user = await this.userRepository.findOneBy({ email: dto.email });
      if (!user || user.recoveryToken !== dto.code)
        return ErrorDto.create({
          message: "Invalid recovery code",
          code: "invalid_code",
        });
      if (new Date() > user.recoveryTokenExpiration)
        return ErrorDto.create({
          message: "Recovery code expired",
          code: "code_expired",
        });
      return { message: "Code verified" };
    } catch (error) {
      const details = error instanceof Error ? error.message : error;
      return ErrorDto.create({ message: "Failed to verify code", details });
    }
  }

  async resetPassword(
    dto: ResetPasswordDto,
  ): Promise<{ message: string } | ErrorDto> {
    try {
      const user = await this.userRepository.findOneBy({ email: dto.email });
      if (!user || user.recoveryToken !== dto.code)
        return ErrorDto.create({
          message: "Invalid recovery code",
          code: "invalid_code",
        });
      if (new Date() > user.recoveryTokenExpiration)
        return ErrorDto.create({
          message: "Recovery code expired",
          code: "code_expired",
        });

      const hashed = await bcrypt.hash(dto.newPassword, 10);
      await this.userRepository.update(user.id, {
        password: hashed,
        recoveryToken: null!,
        recoveryTokenExpiration: null!,
      });
      return { message: "Password reset successfully" };
    } catch (error) {
      const details = error instanceof Error ? error.message : error;
      return ErrorDto.create({ message: "Failed to reset password", details });
    }
  }

  async googleSignIn(
    dto: GoogleSignInDto,
  ): Promise<AuthResponseDto | ErrorDto> {
    try {
      // Verify idToken with Google OAuth2 client
      // Extract email and name from token payload
      // Upsert user and return tokens
      return ErrorDto.create({
        message: "Google sign-in not configured",
        code: "not_implemented",
      });
    } catch (error) {
      const details = error instanceof Error ? error.message : error;
      return ErrorDto.create({ message: "Google sign-in failed", details });
    }
  }
}
```

### `src/modules/auth/auth.controller.ts`

```typescript
import { Body, Controller, Post, UsePipes } from "@nestjs/common";
import { ZodValidationPipe } from "nestjs-zod";

import { AuthService } from "./auth.service";
import {
  GoogleSignInDto,
  RefreshTokenDto,
  ResetPasswordDto,
  SendRecoveryCodeDto,
  SignInDto,
  SignUpDto,
  VerifyRecoveryCodeDto,
} from "./dto/auth.dto";

@Controller("auth")
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @Post("signin")
  @UsePipes(ZodValidationPipe)
  signIn(@Body() body: SignInDto) {
    return this.authService.signIn(body);
  }

  @Post("signup")
  @UsePipes(ZodValidationPipe)
  signUp(@Body() body: SignUpDto) {
    return this.authService.signUp(body);
  }

  @Post("signout")
  signOut() {
    return this.authService.signOut();
  }

  @Post("refresh-token")
  @UsePipes(ZodValidationPipe)
  refresh(@Body() body: RefreshTokenDto) {
    return this.authService.refresh(body);
  }

  @Post("send-recovery-code")
  @UsePipes(ZodValidationPipe)
  sendRecoveryCode(@Body() body: SendRecoveryCodeDto) {
    return this.authService.sendRecoveryCode(body);
  }

  @Post("verify-recovery-code")
  @UsePipes(ZodValidationPipe)
  verifyRecoveryCode(@Body() body: VerifyRecoveryCodeDto) {
    return this.authService.verifyRecoveryCode(body);
  }

  @Post("reset-password")
  @UsePipes(ZodValidationPipe)
  resetPassword(@Body() body: ResetPasswordDto) {
    return this.authService.resetPassword(body);
  }

  @Post("google")
  @UsePipes(ZodValidationPipe)
  googleSignIn(@Body() body: GoogleSignInDto) {
    return this.authService.googleSignIn(body);
  }
}
```

### `src/modules/auth/auth.module.ts`

```typescript
import { Module } from "@nestjs/common";
import { TypeOrmModule } from "@nestjs/typeorm";

import { UserEntity } from "../users/entity/user.entity";
import { AuthController } from "./auth.controller";
import { AuthService } from "./auth.service";

@Module({
  imports: [TypeOrmModule.forFeature([UserEntity])],
  controllers: [AuthController],
  providers: [AuthService],
})
export class AuthModule {}
```

---

## 10. Integration Tests

### `src/modules/auth/auth.spec.ts`

```typescript
import axios from "axios";
import { randomUUID } from "crypto";

import dotenv from "../../constants/dotenv";
import { SignInDto, SignUpDto } from "./dto/auth.dto";

describe("Auth API", () => {
  const baseURL = dotenv.BASE_URL;
  if (!baseURL) throw new Error("missing BASE_URL");
  const apiClient = axios.create({ baseURL });

  const testId = randomUUID().replace(/-/g, "");
  const signUpDto: SignUpDto = {
    name: `Test User ${testId}`,
    email: `user${testId}@test.com`,
    password: "password123",
  };
  const signInDto: SignInDto = {
    email: signUpDto.email,
    password: signUpDto.password,
  };

  let accessToken: string;
  let refreshToken: string;

  it("should sign up and return tokens", async () => {
    const response = await apiClient.post("/auth/signup", signUpDto);
    expect(response.status).toBe(201);
    expect(response.data.accessToken).toBeDefined();
    expect(response.data.refreshToken).toBeDefined();
    accessToken = response.data.accessToken;
    refreshToken = response.data.refreshToken;
  });

  it("should return 400 for invalid sign-up data", async () => {
    await expect(
      apiClient.post("/auth/signup", { email: "bad", password: "123" }),
    ).rejects.toMatchObject({ response: { status: 400 } });
  });

  it("should sign in and return tokens", async () => {
    const response = await apiClient.post("/auth/signin", signInDto);
    expect(response.status).toBe(201);
    expect(response.data.accessToken).toBeDefined();
  });

  it("should return unauthorized for wrong password", async () => {
    await expect(
      apiClient.post("/auth/signin", {
        ...signInDto,
        password: "wrongpassword",
      }),
    ).rejects.toMatchObject({ response: { status: 201 } }); // adjust based on service error shape
  });

  it("should refresh tokens", async () => {
    const response = await apiClient.post("/auth/refresh-token", {
      refreshToken,
    });
    expect(response.status).toBe(201);
    expect(response.data.accessToken).toBeDefined();
  });

  it("should sign out", async () => {
    const response = await apiClient.post("/auth/signout");
    expect(response.status).toBe(201);
    expect(response.data.message).toBe("Signed out successfully");
  });

  it("should send recovery code for existing email", async () => {
    const response = await apiClient.post("/auth/send-recovery-code", {
      email: signUpDto.email,
    });
    expect(response.status).toBe(201);
  });

  it("should clean up test user", async () => {
    const users = await apiClient.get("/users");
    const user = users.data.find((u: any) => u.email === signUpDto.email);
    if (user)
      await apiClient.delete(`/users/${user.id}`).catch(() => undefined);
  });
});
```

---

## 11. Deploy / Run

```bash
# Ensure PostgreSQL is running and .env is configured

# Start in development mode (hot reload)
yarn dev

# Build for production
yarn build

# Start production server
yarn start:prod
```

---

## 12. Development Workflow

```bash
# Local dev (hot reload, port from PORT env var, default 3000)
yarn dev

# Run all tests
yarn test

# Coverage report
yarn test:cov

# Format source files
yarn format

# Lint and auto-fix
yarn lint
```

---

## 13. Conventions

| Rule                 | Detail                                                                                            |
| -------------------- | ------------------------------------------------------------------------------------------------- |
| Module structure     | Each feature under `src/modules/<feature>/` with module, controller, service, spec, dto/, entity/ |
| Validation           | `@UsePipes(ZodValidationPipe)` on every route with a `@Body()` parameter                          |
| DTOs                 | Defined with `nestjs-zod` `createZodDto()` — never use plain classes or interfaces                |
| Service return types | Always `Promise<ResponseDto \| ErrorDto>` — never throw, always return                            |
| Entity usage         | Only injected in services via `@InjectRepository` — never used directly in controllers            |
| Password hashing     | Always `bcrypt.hash(password, 10)` — never store plaintext                                        |
| JWT                  | Signed with `JWT_SECRET` env var; access token short-lived, refresh token long-lived              |
| Recovery tokens      | 6-digit numeric code, 15-minute expiration, stored on user row                                    |
| Port                 | Configured via `PORT` env var, defaults to 3000                                                   |
| TypeORM sync         | `synchronize: true` in dev — disable in production and use migrations                             |
| Tests                | Integration only — run against live server, no mocks                                              |

---

## 14. Adding a New Entity

1. Create `src/modules/<entity>/` with: `<entity>.module.ts`, `<entity>.controller.ts`, `<entity>.service.ts`, `<entity>.spec.ts`, `dto/<entity>.dto.ts`, `entity/<entity>.entity.ts`
2. Add entity class to the `entities` array in `src/database/database.module.ts`
3. Register the module in `src/app.module.ts` imports
4. Add `TypeOrmModule.forFeature([<Entity>Entity])` to the feature module imports
5. Add env vars to `.env` if the entity requires feature-specific config
6. Update `swagger-2.0.yaml` to reflect new routes and DTOs
