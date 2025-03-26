# Java 스타일 가이드 (Java 24 기준)

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
