# JavaScript 스타일 가이드

## ESLint 설정

- 프로젝트에는 항상 ESLint를 사용합니다.
- `.eslintrc.js` 파일 예시:

```javascript
module.exports = {
    "env": {
        "browser": true,
        "es2024": true,
        "jquery": true
    },
    "extends": [
        "eslint:recommended",
        "airbnb-base"
    ],
    "rules": {
        "no-console": "warn",
        "indent": ["error", 2],
        "quotes": ["error", "single"],
        "semi": ["error", "always"]
    }
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

  > 예: backgroundColor, userName, phoneNumber

#### 주요 용도
- 변수명
- 함수명
- 메서드명
- 프로퍼티명

### 파스칼 표기법 (PascalCase)

* 모든 단어의 첫문자를 대문자로 표기하고 붙여쓰는 표기법

  > 예: BackgroundColor, UserService, PowerPoint

#### 주요 사용
- 클래스명
- 생성자 함수명
- 컴포넌트명

### 상수 표기법 (CONSTANT_CASE)

- 모든 문자를 대문자로 작성하고 단어를 언더스코어(`_`)로 구분하는 표기법

  > 예: MAX_COUNT, API_KEY, ENVIRONMENT_VARIABLES

#### 주요 용도
- 전역 상수
- 환경 변수
- 매직 넘버

## 최신 JavaScript 기능 활용

### Optional Chaining

```javascript
// ❌ 나쁨
const streetName = user && user.address && user.address.street && user.address.street.name;

// 👍 좋음
const streetName = user?.address?.street?.name;
```

### Nullish Coalescing

```javascript
// ❌ 나쁨
const count = value || 0;

// 👍 좋음
const count = value ?? 0;
```

### 구조 분해 할당

```javascript
// ❌ 나쁨
const firstName = user.firstName;
const lastName = user.lastName;

// 👍 좋음
const { firstName, lastName } = user;
```

## jQuery 사용 가이드

### 셀렉터 최적화

```javascript
// ❌ 나쁨
$('div.myClass');
$('div#myId');

// 👍 좋음
$('.myClass');
$('#myId');
```

### 체이닝

```javascript
// ❌ 나쁨
const $elem = $('#myElement');
$elem.addClass('active');
$elem.show();
$elem.text('Hello');

// 👍 좋음
$('#myElement')
  .addClass('active')
  .show()
  .text('Hello');
```

### 이벤트 위임

```javascript
// ❌ 나쁨
$('.dynamic-item').on('click', function() {
  // 동적으로 추가되는 요소에 대해 작동하지 않음
});

// 👍 좋음
$(document).on('click', '.dynamic-item', function() {
  // 동적으로 추가되는 요소에도 이벤트 적용
});
```

### 캐싱

```javascript
// ❌ 나쁨
$('#myElement').addClass('active');
$('#myElement').show();
$('#myElement').text('Hello');

// 👍 좋음
const $myElement = $('#myElement');
$myElement.addClass('active')
         .show()
         .text('Hello');
```

### AJAX 호출

```javascript
// 👍 좋음
$.ajax({
  url: '/api/users',
  method: 'GET',
  dataType: 'json',
  success: function(response) {
    // 성공 처리
  },
  error: function(xhr, status, error) {
    // 에러 처리
  }
});

// 더 좋음 (Promise 사용)
$.ajax({
  url: '/api/users',
  method: 'GET',
  dataType: 'json'
})
.then(response => {
  // 성공 처리
})
.catch(error => {
  // 에러 처리
});
```

## 비동기 처리

### Promise 사용

```javascript
// ❌ 나쁨
function fetchUser(callback) {
  $.get('/api/user', function(response) {
    callback(null, response);
  }).fail(function(error) {
    callback(error);
  });
}

// 👍 좋음
async function fetchUser() {
  try {
    const response = await $.ajax({
      url: '/api/user',
      method: 'GET'
    });
    return response;
  } catch (error) {
    console.error('Error fetching user:', error);
    throw error;
  }
}
```

### async/await 사용

```javascript
// ❌ 나쁨
function processUser() {
  return fetchUser()
    .then(user => updateUser(user))
    .then(result => saveUser(result))
    .catch(error => handleError(error));
}

// 👍 좋음
async function processUser() {
  try {
    const user = await fetchUser();
    const result = await updateUser(user);
    return await saveUser(result);
  } catch (error) {
    handleError(error);
  }
}
```

## 모듈 시스템

### Import/Export

```javascript
// ❌ 나쁨
const utils = require('./utils');

// 👍 좋음
import { specificFunction } from './utils';
import * as Utils from './utils';
```

## 참고

* [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)
* [Google JavaScript Style Guide](https://google.github.io/styleguide/jsguide.html)
* [jQuery Best Practices](https://www.w3schools.com/jquery/jquery_best_practices.asp)
* [ESLint](https://eslint.org/)
* [Clean Code JavaScript](https://github.com/ryanmcdermott/clean-code-javascript)
