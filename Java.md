# Java 스타일 가이드 (Java 24 기준)

## 1. 개요

이 문서는 구글의 Java 언어 코딩 표준의 완벽한 정의 문서입니다. Java 소스 파일은 이 문서의 규칙을 준수해야만 Google Style이라 칭할 수 있습니다.

## 코드 포맷팅 도구

- 프로젝트에는 항상 Checkstyle을 사용합니다.
- Google Java Style을 기본으로 사용합니다.
- `checkstyle.xml` 파일 예시:

```xml
<?xml version="1.0"?>
<!DOCTYPE module PUBLIC
          "-//Checkstyle//DTD Checkstyle Configuration 1.3//EN"
          "https://checkstyle.org/dtds/configuration_1_3.dtd">
<module name="Checker">
    <property name="severity" value="error"/>
    <module name="TreeWalker">
        <module name="JavadocMethod"/>
        <module name="JavadocType"/>
        <module name="JavadocVariable"/>
        <module name="JavadocStyle"/>
    </module>
</module>
```

## 들여쓰기 (Indentation)

- 들여쓰기 단위를 `4칸`을 사용합니다.
- 들여쓰기에 탭을 사용하지 않고 스페이스(space)를 사용합니다.

## 사용하는 표기법

- 3가지의 표기법을 용도별로 사용합니다.

### 카멜 표기법 (camelCase)

* 첫문자를 제외하고 이후 나오는 단어의 첫문자를 대문자로 표기하고 붙여쓰는 표기법
* 띄어쓰기 대신 대문자로 단어를 구분하는 표기 방식

  > 예: backgroundColor, userName, phoneNumber

#### 주요 용도
- 변수명
- 메서드명
- 패키지명

### 파스칼 표기법 (PascalCase)

* 모든 단어의 첫문자를 대문자로 표기하고 붙여쓰는 표기법

  > 예: BackgroundColor, UserService, PowerPoint

#### 주요 사용
- 클래스명
- 인터페이스명
- 열거형명
- 애노테이션명

### 상수 표기법 (CONSTANT_CASE)

- 모든 문자를 대문자로 작성하고 단어를 언더스코어(`_`)로 구분하는 표기법

  > 예: MAX_COUNT, API_KEY, ENVIRONMENT_VARIABLES

#### 주요 용도
- static final 상수
- 열거형 상수
- 환경 변수

## 최신 Java 기능 활용

### Record 클래스

```java
// ❌ 나쁨
public class UserDto {
    private final String name;
    private final int age;
    
    public UserDto(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    // getters, equals, hashCode, toString
}

// 👍 좋음
public record UserDto(String name, int age) {}
```

### Pattern Matching

```java
// ❌ 나쁨
if (obj instanceof String) {
    String str = (String) obj;
    System.out.println(str.length());
}

// 👍 좋음
if (obj instanceof String str) {
    System.out.println(str.length());
}
```

### Switch Expressions

```java
// ❌ 나쁨
String result;
switch (day) {
    case MONDAY:
        result = "월요일";
        break;
    case TUESDAY:
        result = "화요일";
        break;
    default:
        result = "기타";
}

// 👍 좋음
String result = switch (day) {
    case MONDAY -> "월요일";
    case TUESDAY -> "화요일";
    default -> "기타";
};
```

## 비동기 처리

### CompletableFuture 사용

```java
// ❌ 나쁨
Future<User> future = executorService.submit(() -> findUser(userId));
User user = future.get(); // 블로킹

// 👍 좋음
CompletableFuture<User> future = CompletableFuture
    .supplyAsync(() -> findUser(userId))
    .thenApply(this::processUser)
    .exceptionally(this::handleError);
```

## 모듈 시스템

### Import 문

```java
// ❌ 나쁨
import java.util.*;

// 👍 좋음
import java.util.List;
import java.util.ArrayList;
import java.util.Map;
```

### Static Import

```java
// ❌ 나쁨
import static org.junit.jupiter.api.Assertions.*;

// 👍 좋음
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;
```

## 메서드

### 메서드 파라미터

```java
// ❌ 나쁨
public void updateUser(String name, int age, String email) {
    // ...
}

// 👍 좋음
public record UpdateUserParams(String name, int age, String email) {}

public void updateUser(UpdateUserParams params) {
    // ...
}
```

### 메서드 체이닝

```java
// ❌ 나쁨
StringBuilder sb = new StringBuilder();
sb.append("Hello");
sb.append(" ");
sb.append("World");

// 👍 좋음
String result = new StringBuilder()
    .append("Hello")
    .append(" ")
    .append("World")
    .toString();
```

## 참고

* [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)
* [Oracle Code Conventions](https://www.oracle.com/java/technologies/javase/codeconventions-contents.html)
* [Checkstyle](https://checkstyle.sourceforge.io/)
* [Clean Code](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)
* [Effective Java](https://www.amazon.com/Effective-Java-Joshua-Bloch/dp/0134685997)

## 6. 프로그래밍 연습

### 6.1 @Override: 항상 사용

@Override 애노테이션이 사용 가능할 때는 반드시 사용합니다. 이는 다음과 같은 경우를 포함합니다:
- 슈퍼클래스의 메서드를 오버라이딩할 때
- 인터페이스의 메서드를 구현할 때
- 슈퍼 인터페이스의 메서드를 재구현할 때

예외: 부모 메서드가 @Deprecated인 경우 @Override를 생략할 수 있습니다.

### 6.2 예외 처리: 생략 금지

예외를 잡았을 때는 반드시 적절한 처리를 해야 합니다. 일반적인 처리 방법은 다음과 같습니다:
- 로그 기록
- 불가능한 상황으로 판단되는 경우 AssertionError로 다시 던지기

예외를 무시해야 하는 경우라면, 반드시 주석으로 그 이유를 설명해야 합니다:

```java
try {
    int i = Integer.parseInt(response);
    return handleNumericResponse(i);
} catch (NumberFormatException ok) {
    // 숫자가 아닌 경우는 정상적인 흐름으로 처리
}
return handleTextResponse(response);
```

### 6.3 정적 멤버: 클래스로 참조

정적 멤버는 인스턴스가 아닌 클래스 이름으로 참조해야 합니다:

```java
// 👍 좋음
Foo.aStaticMethod();

// ❌ 나쁨
aFoo.aStaticMethod();
somethingThatYieldsAFoo().aStaticMethod();
```

## 7. Javadoc

### 7.1 포맷팅

#### 7.1.1 기본 형식

Javadoc의 기본 형식은 다음과 같습니다:

```java
/**
 * 여러 줄의 Javadoc 텍스트는
 * 이렇게 작성합니다...
 */
public int method(String p1) { ... }

/** 한 줄짜리 간단한 Javadoc. */
```

#### 7.1.2 문단

- 문단 사이에는 빈 줄을 넣습니다
- 블록 태그 그룹 앞에도 빈 줄을 넣습니다
- 첫 번째 문단을 제외한 각 문단은 <p>로 시작합니다

#### 7.1.3 블록 태그

주요 블록 태그(@param, @return, @throws, @deprecated)는 비어있지 않은 설명을 포함해야 합니다.

### 7.2 요약 설명

모든 Javadoc은 간단한 요약 설명으로 시작합니다:
- 명사구 또는 동사구로 작성
- "이 메서드는..."과 같은 문장으로 시작하지 않음
- 대문자로 시작하고 마침표로 끝남

예시:
```java
/** 
 * 고객 ID를 반환합니다.
 */
```

### 7.3 Javadoc 작성 대상

다음의 경우 반드시 Javadoc을 작성해야 합니다:
- 모든 public 클래스
- 모든 public 또는 protected 멤버

예외적인 경우:
1. 자명한 메서드 (예: 단순 getter)
2. 오버라이딩 메서드 (상위 문서로 충분한 경우)
