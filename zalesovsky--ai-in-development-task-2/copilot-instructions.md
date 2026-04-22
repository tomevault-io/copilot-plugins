## ai-in-development-task-2

> // Prefer constructor injection

# Spring Framework Guidelines

## Dependency Injection
```java
// Prefer constructor injection
@Service
public class UserService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    public UserService(UserRepository userRepository, 
                      PasswordEncoder passwordEncoder) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }
}
```

## Configuration
```yaml
# application.yml
spring:
  profiles:
    active: dev
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: ${DB_USER}
    password: ${DB_PASS}
```

## Bean Management
```java
@Configuration
public class AppConfig {
    @Bean
    @Profile("prod")
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager();
    }
}
```

## Security
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) {
        return http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .build();
    }
}
```

## Testing
```java
@SpringBootTest
class UserServiceTest {
    @Autowired
    private UserService userService;
    
    @MockBean
    private UserRepository userRepository;
    
    @Test
    void whenValidUser_thenSaveSucceeds() {
        // test implementation
    }
}
```

@example/UserService.java
@example/SecurityConfig.java
@example/application.yml

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Zalesovsky) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
