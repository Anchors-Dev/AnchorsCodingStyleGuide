# TypeScript 스타일 가이드

## ESLint 설정

- 프로젝트에는 항상 ESLint를 사용합니다.
- 기본적으로 [typescript-eslint](https://typescript-eslint.io/)를 사용합니다.
- `.eslintrc.js` 파일 예시:

```javascript
module.exports = {
  parser: '@typescript-eslint/parser',
  plugins: ['@typescript-eslint'],
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:@typescript-eslint/recommended-requiring-type-checking',
  ],
  parserOptions: {
    project: './tsconfig.json',
  },
  rules: {
    // 커스텀 규칙 설정
  },
};
```

## 들여쓰기 (Indentation)

- 들여쓰기 단위를 `2칸`을 사용합니다.
- 들여쓰기에 탭을 사용하지 않고 스페이스(space)를 사용합니다.

## 사용하는 표기법

- 3가지의 표기법을 용도별로 사용합니다.

### 카멜 표기법 (camelCase)

* 첫문자를 제외하고 이후 나오는 단어의 첫문자를 대문자로 표기하고 붙여쓰는 표기법
* 띄어쓰기 대신 대문자로 단어를 구분하는 표기 방식

  > 예: backgroundColor, typeName, iPhone

#### 주요 용도
- 변수명
- 함수명
- 메서드명
- 프로퍼티명

### 파스칼 표기법 (PascalCase)

* 모든 단어의 첫문자를 대문자로 표기하고 붙여쓰는 표기법

  > 예: BackgroundColor, TypeName, PowerPoint

#### 주요 사용
- 클래스명
- 인터페이스명
- 타입명
- 열거형명
- 데코레이터명
- React/Vue 컴포넌트 파일명

### 상수 표기법 (CONSTANT_CASE)

- 모든 문자를 대문자로 작성하고 단어를 언더스코어(`_`)로 구분하는 표기법

  > 예: MAX_COUNT, API_KEY, ENVIRONMENT_VARIABLES

#### 주요 용도
- 전역 상수
- 환경 변수
- 매직 넘버

## 최신 JavaScript/TypeScript 기능 활용

### Optional Chaining

```typescript
// ❌ 나쁨
const streetName = user && user.address && user.address.street && user.address.street.name;

// 👍 좋음
const streetName = user?.address?.street?.name;
```

### Nullish Coalescing

```typescript
// ❌ 나쁨
const count = value || 0; // falsy 값(0, '', false)도 기본값으로 대체됨

// 👍 좋음
const count = value ?? 0; // null과 undefined일 때만 기본값으로 대체
```

### 타입 추론 활용

```typescript
// ❌ 나쁨 - 불필요한 타입 명시
const numbers: number[] = [1, 2, 3];
const user: User = new User();

// 👍 좋음 - 타입 추론 활용
const numbers = [1, 2, 3];
const user = new User();
```

### const assertions

```typescript
// 👍 좋음 - 리터럴 타입으로 추론
const config = {
  endpoint: 'https://api.example.com',
  timeout: 3000,
} as const;
```

## 비동기 처리

### async/await 사용

```typescript
// ❌ 나쁨
function fetchUser() {
  return fetch('/api/user')
    .then(response => response.json())
    .then(user => processUser(user))
    .catch(error => handleError(error));
}

// 👍 좋음
async function fetchUser() {
  try {
    const response = await fetch('/api/user');
    const user = await response.json();
    return processUser(user);
  } catch (error) {
    handleError(error);
  }
}
```

## 모듈 시스템

### Import/Export

```typescript
// ❌ 나쁨
import * as Everything from './module';

// 👍 좋음
import { specificFunction, SpecificType } from './module';

// 👍 좋음 - 모듈 전체를 import 해야 하는 경우
import * as UserModule from './userModule';
```

### 타입 Import

```typescript
// ❌ 나쁨
import { User } from './types';

// 👍 좋음
import type { User } from './types';
```

## 함수

### 함수 파라미터

```typescript
// ❌ 나쁨
function updateUser(name: string, age: number, email: string) {
  // ...
}

// 👍 좋음
interface UpdateUserParams {
  name: string;
  age: number;
  email: string;
}

function updateUser(params: UpdateUserParams) {
  // ...
}
```

### 화살표 함수

```typescript
// ❌ 나쁨
function (a: number, b: number): number {
  return a + b;
}

// 👍 좋음
const add = (a: number, b: number): number => a + b;
```

## 참고

* [typescript-eslint](https://typescript-eslint.io/)
* [ESLint](https://eslint.org/)
* [Google TypeScript Style Guide](https://google.github.io/styleguide/tsguide.html)
* [Microsoft TypeScript Coding Guidelines](https://github.com/microsoft/TypeScript/wiki/Coding-guidelines)
* [Clean Code TypeScript](https://github.com/labs42io/clean-code-typescript)
* [Angular Style Guide](https://angular.io/guide/styleguide)

