# Spring 스타일 가이드 (Spring Boot 3.x)

## 프로젝트 설정

### build.gradle.kts
```kotlin
plugins {
    id("org.springframework.boot") version "3.2.0"
    id("io.spring.dependency-management") version "1.1.4"
    kotlin("jvm") version "1.9.20"
    kotlin("plugin.spring") version "1.9.20"
    kotlin("plugin.jpa") version "1.9.20"
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    implementation("org.springframework.boot:spring-boot-starter-validation")
    implementation("org.springframework.boot:spring-boot-starter-security")
    
    // JWT
    implementation("io.jsonwebtoken:jjwt-api:0.12.3")
    runtimeOnly("io.jsonwebtoken:jjwt-impl:0.12.3")
    runtimeOnly("io.jsonwebtoken:jjwt-jackson:0.12.3")
    
    // Database
    runtimeOnly("com.h2database:h2")
    runtimeOnly("org.postgresql:postgresql")
    
    // Test
    testImplementation("org.springframework.boot:spring-boot-starter-test")
    testImplementation("org.springframework.security:spring-security-test")
}
```

## 디렉토리 구조

```
src/
├── main/
│   ├── kotlin/
│   │   └── com/
│   │       └── example/
│   │           └── demo/
│   │               ├── DemoApplication.kt
│   │               ├── common/                    # 공통 모듈
│   │               │   ├── annotation/           # 커스텀 어노테이션
│   │               │   ├── config/              # 설정 클래스
│   │               │   ├── exception/           # 예외 처리
│   │               │   ├── security/            # 보안 관련
│   │               │   └── utils/               # 유틸리티
│   │               │
│   │               ├── domain/                   # 도메인 계층
│   │               │   ├── user/                # 사용자 도메인
│   │               │   │   ├── domain/         # 도메인 모델
│   │               │   │   │   ├── User.kt
│   │               │   │   │   └── UserRole.kt
│   │               │   │   ├── repository/     # 레포지토리
│   │               │   │   ├── service/        # 도메인 서비스
│   │               │   │   └── exception/      # 도메인 예외
│   │               │   │
│   │               │   └── order/               # 주문 도메인
│   │               │       ├── domain/
│   │               │       ├── repository/
│   │               │       ├── service/
│   │               │       └── exception/
│   │               │
│   │               ├── infrastructure/           # 인프라스트럭처 계층
│   │               │   ├── persistence/        # 영속성 관리
│   │               │   └── external/           # 외부 서비스
│   │               │
│   │               ├── interfaces/              # 인터페이스 계층
│   │               │   ├── rest/               # REST API
│   │               │   │   ├── user/
│   │               │   │   │   ├── UserController.kt
│   │               │   │   │   ├── UserDto.kt
│   │               │   │   │   └── UserMapper.kt
│   │               │   │   └── order/
│   │               │   └── scheduler/          # 스케줄러
│   │               │
│   │               └── application/             # 응용 계층
│   │                   ├── user/               # 사용자 관련 유스케이스
│   │                   │   ├── UserService.kt
│   │                   │   ├── dto/
│   │                   │   └── mapper/
│   │                   └── order/              # 주문 관련 유스케이스
│   │
│   └── resources/
│       ├── application.yml                     # 기본 설정
│       ├── application-local.yml              # 로컬 환경
│       ├── application-dev.yml                # 개발 환경
│       └── application-prod.yml               # 운영 환경
│
└── test/
    └── kotlin/
        └── com/
            └── example/
                └── demo/
                    ├── domain/                # 도메인 테스트
                    ├── interfaces/            # 인터페이스 테스트
                    └── application/           # 응용 계층 테스트
```

## 레이어별 코드 스타일 가이드

### 도메인 계층 (Domain Layer)

```kotlin
// domain/user/domain/User.kt
@Entity
@Table(name = "users")
class User(
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long? = null,

    @Column(nullable = false, unique = true)
    val email: String,

    @Column(nullable = false)
    var password: String,

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    var role: UserRole = UserRole.USER
) {
    fun updatePassword(newPassword: String) {
        this.password = newPassword
    }

    // 도메인 로직 구현
}
```

### 응용 계층 (Application Layer)

```kotlin
// application/user/UserService.kt
@Service
@Transactional
class UserService(
    private val userRepository: UserRepository,
    private val passwordEncoder: PasswordEncoder
) {
    fun signup(command: SignupCommand): UserResponse {
        validateDuplicateEmail(command.email)
        
        val user = User(
            email = command.email,
            password = passwordEncoder.encode(command.password)
        )
        
        return UserMapper.toResponse(userRepository.save(user))
    }
    
    private fun validateDuplicateEmail(email: String) {
        if (userRepository.existsByEmail(email)) {
            throw DuplicateEmailException(email)
        }
    }
}
```

### 인터페이스 계층 (Interface Layer)

```kotlin
// interfaces/rest/user/UserController.kt
@RestController
@RequestMapping("/api/v1/users")
class UserController(
    private val userService: UserService
) {
    @PostMapping("/signup")
    fun signup(@Valid @RequestBody request: SignupRequest): ResponseEntity<UserResponse> {
        val command = SignupCommand(
            email = request.email,
            password = request.password
        )
        return ResponseEntity.ok(userService.signup(command))
    }
}

// interfaces/rest/user/UserDto.kt
data class SignupRequest(
    @field:Email
    @field:NotBlank
    val email: String,
    
    @field:NotBlank
    @field:Size(min = 8, max = 20)
    val password: String
)

data class UserResponse(
    val id: Long,
    val email: String,
    val role: UserRole
)
```

### 인프라스트럭처 계층 (Infrastructure Layer)

```kotlin
// infrastructure/persistence/UserRepositoryImpl.kt
@Repository
class UserRepositoryImpl(
    private val jpaRepository: UserJpaRepository
) : UserRepository {
    override fun findByEmail(email: String): User? {
        return jpaRepository.findByEmail(email)
    }
    
    override fun save(user: User): User {
        return jpaRepository.save(user)
    }
}
```

## 예외 처리

```kotlin
// common/exception/GlobalExceptionHandler.kt
@RestControllerAdvice
class GlobalExceptionHandler {
    @ExceptionHandler(DuplicateEmailException::class)
    fun handleDuplicateEmailException(e: DuplicateEmailException): ResponseEntity<ErrorResponse> {
        return ResponseEntity
            .status(HttpStatus.CONFLICT)
            .body(ErrorResponse(message = e.message))
    }
}

// common/exception/ErrorResponse.kt
data class ErrorResponse(
    val message: String,
    val timestamp: LocalDateTime = LocalDateTime.now()
)
```

## 보안 설정

```kotlin
// common/security/SecurityConfig.kt
@Configuration
@EnableWebSecurity
class SecurityConfig(
    private val jwtAuthenticationFilter: JwtAuthenticationFilter
) {
    @Bean
    fun securityFilterChain(http: HttpSecurity): SecurityFilterChain {
        return http
            .csrf { it.disable() }
            .authorizeHttpRequests {
                it.requestMatchers("/api/v1/auth/**").permitAll()
                  .anyRequest().authenticated()
            }
            .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter::class.java)
            .build()
    }
}
```

## 테스트 코드 작성

```kotlin
// test/kotlin/com/example/demo/domain/user/UserTest.kt
@DisplayName("User 도메인 테스트")
class UserTest {
    @Test
    fun `비밀번호 변경 테스트`() {
        // given
        val user = User(
            email = "test@example.com",
            password = "oldPassword"
        )
        val newPassword = "newPassword"
        
        // when
        user.updatePassword(newPassword)
        
        // then
        assertThat(user.password).isEqualTo(newPassword)
    }
}
```

## API 문서화 (Swagger)

```kotlin
// common/config/SwaggerConfig.kt
@Configuration
class SwaggerConfig {
    @Bean
    fun openAPI(): OpenAPI = OpenAPI()
        .info(
            Info()
                .title("Demo API")
                .version("1.0.0")
                .description("Demo API 문서")
        )
        .components(
            Components()
                .addSecuritySchemes(
                    "bearer-key",
                    SecurityScheme()
                        .type(SecurityScheme.Type.HTTP)
                        .scheme("bearer")
                        .bearerFormat("JWT")
                )
        )
}
```

## 참고

* [Spring 공식 문서](https://spring.io/projects/spring-boot)
* [Spring Security 공식 문서](https://spring.io/projects/spring-security)
* [Domain-Driven Design](https://www.domainlanguage.com/ddd/)
* [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
