## jaws

> Grug brain development philosophy - keeping software simple and avoiding complexity demons


# Grug Brain Developer Rules

> "complexity very, very bad" - grug brain developer

## Core Philosophy

### Rule 1: Complexity is the Apex Predator
- **Complexity bad. Complexity very bad. Complexity VERY, VERY bad.**
- Complexity is invisible demon that enters codebase through well-meaning developers
- When you sense complexity demon, reach for magic word: "no"
- Simple code that works > complex code that might work better

```java
// BAD: Complexity demon loves this
public class AbstractSingletonProxyFactoryBean<T> implements 
    FactoryBean<T>, BeanClassLoaderAware, BeanFactoryAware {
    // 200 lines of abstraction nobody understands
}

// GOOD: Grug understand this
public class UserService {
    private final UserRepository userRepo;
    
    public User findUser(Long id) {
        return userRepo.findById(id);
    }
}
```

### Rule 2: Magic Word "No"
- Best weapon against complexity: "no"
- "no, grug not build that feature"
- "no, grug not build that abstraction"
- "no, grug not need enterprise pattern for simple task"

### Rule 3: Fear of Looking Dumb is Normal
- All developers feel like impostor sometimes
- Senior grug should say "this too complex for grug"
- Makes it OK for junior grugs to admit confusion
- Nobody impostor if everybody impostor

### Rule 4: Saying "Ok" When Necessary
- Sometimes must compromise for shiney rocks (salary)
- Build 80/20 solution - 80% value with 20% code
- Don't tell project manager, just do it simple way

## Code Structure & Design

### Rule 5: Don't Factor Code Too Early
- Early in project everything like water - very abstract
- Wait for natural cut-points to emerge
- Good cut-point has narrow interface, hides complexity demon inside

```java
// TOO EARLY: Don't abstract before you understand
public abstract class AbstractPaymentProcessor<T extends PaymentMethod> {
    // Complex generics before knowing what you're doing
}

// BETTER: Simple concrete implementation first
public class CreditCardPaymentProcessor {
    public PaymentResult processPayment(CreditCard card, Amount amount) {
        // Simple, clear implementation
        return paymentGateway.charge(card, amount);
    }
}
```

### Rule 6: Break Down Complex Expressions
- Don't minimize lines of code at expense of debuggability
- Use intermediate variables with good names
- Easier to debug when each step visible

```java
// BAD: Hard to debug
if (user != null && !user.isActive() && 
    (user.hasRole("ADMIN") || user.hasRole("MANAGER")) &&
    user.getLastLoginDate().isBefore(LocalDate.now().minusDays(30))) {
    // Do something
}

// GOOD: Easy to debug each part
if (user != null) {
    boolean userIsInactive = !user.isActive();
    boolean userIsAdminOrManager = user.hasRole("ADMIN") || user.hasRole("MANAGER");
    boolean userLastLoginOld = user.getLastLoginDate().isBefore(LocalDate.now().minusDays(30));
    
    if (userIsInactive && userIsAdminOrManager && userLastLoginOld) {
        // Do something
    }
}
```

### Rule 7: DRY Balance
- Don't Repeat Yourself, but don't over-abstract
- Sometimes copy-paste with small variation better than complex abstraction
- Repeat code obvious > complex DRY solution

```java
// SOMETIMES OK: Simple repeated code
public void sendWelcomeEmail(User user) {
    String subject = "Welcome " + user.getName();
    String body = "Welcome to our app, " + user.getName();
    emailService.send(user.getEmail(), subject, body);
}

public void sendGoodbyeEmail(User user) {
    String subject = "Goodbye " + user.getName();
    String body = "Sorry to see you go, " + user.getName();
    emailService.send(user.getEmail(), subject, body);
}

// AVOID: Over-abstraction for simple cases
public void sendTemplatedEmail(User user, EmailTemplate template, 
    Map<String, Object> context, EmailConfiguration config) {
    // Complex template system for 2 simple emails
}
```

### Rule 8: Locality of Behavior Over Separation of Concerns
- Put code on thing that does the thing
- When grug look at thing, grug know what thing does
- Don't separate related code across many files

```java
// GOOD: Everything about User validation in one place
public class User {
    private String email;
    private String name;
    
    public void setEmail(String email) {
        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("Invalid email");
        }
        this.email = email;
    }
    
    public boolean canLogin() {
        return email != null && name != null;
    }
}

// AVOID: Spreading User logic across multiple validators
// UserEmailValidator, UserNameValidator, UserLoginValidator, etc.
```

## Development Practices

### Rule 9: Testing Philosophy
- Tests save grug many times, grug love tests
- But avoid test shamans who demand "test first" before understanding problem
- Write tests after prototype phase when code firms up
- Focus on integration tests - sweet spot between unit and e2e

```java
// GOOD: Integration test - tests real behavior
@Test
public void shouldCreateUserAndSendWelcomeEmail() {
    // Given
    CreateUserRequest request = new CreateUserRequest("john@example.com", "John");
    
    // When
    User user = userService.createUser(request);
    
    // Then
    assertThat(user.getEmail()).isEqualTo("john@example.com");
    assertThat(emailService.getSentEmails()).hasSize(1);
    assertThat(emailService.getSentEmails().get(0).getSubject()).contains("Welcome");
}
```

### Rule 10: Logging is Crucial
- Log all major logical branches (if/for statements)
- Include request ID for distributed systems
- Make log levels dynamically controlled
- Logging infrastructure very important, invest time

```java
// GOOD: Comprehensive logging
public class PaymentService {
    private static final Logger logger = LoggerFactory.getLogger(PaymentService.class);
    
    public PaymentResult processPayment(PaymentRequest request) {
        logger.info("Processing payment for user {} amount {}", request.getUserId(), request.getAmount());
        
        if (request.getAmount().compareTo(BigDecimal.ZERO) <= 0) {
            logger.warn("Invalid payment amount {} for user {}", request.getAmount(), request.getUserId());
            return PaymentResult.failed("Invalid amount");
        }
        
        try {
            PaymentResult result = paymentGateway.charge(request);
            logger.info("Payment {} for user {} result: {}", 
                request.getAmount(), request.getUserId(), result.getStatus());
            return result;
        } catch (Exception e) {
            logger.error("Payment failed for user {} amount {}", 
                request.getUserId(), request.getAmount(), e);
            return PaymentResult.failed("Payment processing error");
        }
    }
}
```

### Rule 11: Tools Are Crucial
- Good debugger worth all shiney rocks
- IDE with code completion essential
- Type systems help when they show what grug can do (dot completion)
- Learn debugger deeply - conditional breakpoints, expression evaluation

```java
// GOOD: Use type system for discoverability
public class PaymentService {
    // When grug types "payment." IDE shows all options
    public PaymentResult processPayment(PaymentRequest request) {
        // Type safety prevents many bugs
        return request.getPaymentMethod().process(request.getAmount());
    }
}

// AVOID: Complex generics that confuse IDE and grug
public class ComplexGenericProcessor<T extends Comparable<T> & Serializable> {
    // Grug confused, IDE confused, everyone confused
}
```

### Rule 12: Refactoring Carefully
- Refactoring fine, but keep refactors small
- Don't be "too far from shore" - system should work at each step
- Respect existing code (Chesterton's Fence)
- Understand why code exists before changing

```java
// GOOD: Small refactoring step
// Before
public void processOrder(Order order) {
    validateOrder(order);
    calculateTotal(order);
    chargePayment(order);
    sendConfirmation(order);
}

// After (one small step)
public void processOrder(Order order) {
    validateOrder(order);
    PaymentResult result = processPayment(order);
    if (result.isSuccessful()) {
        sendConfirmation(order);
    }
}

private PaymentResult processPayment(Order order) {
    calculateTotal(order);
    return chargePayment(order);
}
```

## Architecture & APIs

### Rule 13: API Simplicity
- APIs should not make grug think too much
- Simple cases should be simple, complex cases should be possible
- Put methods on the thing, not elsewhere

```java
// GOOD: Simple API
public class UserList {
    private List<User> users;
    
    // Grug just wants to filter list
    public List<User> getActiveUsers() {
        return users.stream()
            .filter(User::isActive)
            .collect(Collectors.toList());
    }
}

// BAD: Over-engineered API
public class UserList {
    // Grug must understand streams, collectors, predicates just to filter
    public <T> Stream<T> transform(Function<User, T> mapper, Predicate<User> filter) {
        return users.stream().filter(filter).map(mapper);
    }
}
```

### Rule 14: Avoid Microservices Complexity
- Why take hardest problem (factoring system) and add network calls?
- Network calls equivalent to millions of CPU cycles
- Keep things simple until scale demands otherwise

```java
// GOOD: Simple service
@Service
public class OrderService {
    private final PaymentService paymentService;
    private final InventoryService inventoryService;
    
    public OrderResult createOrder(OrderRequest request) {
        // Simple method calls, no network complexity
        inventoryService.reserveItems(request.getItems());
        PaymentResult payment = paymentService.processPayment(request.getPayment());
        return new OrderResult(payment.getTransactionId());
    }
}

// AVOID: Premature microservices
// OrderService -> HTTP call -> PaymentService -> HTTP call -> InventoryService
// Much complexity, network failures, timeouts, circuit breakers, etc.
```

### Rule 15: Concurrency Caution
- Grug fear concurrency, as should all sane developers
- Use simple models: stateless request handlers, simple job queues
- Thread-local variables for framework code
- Avoid complex concurrent data structures unless necessary

```java
// GOOD: Simple concurrency model
@RestController
public class UserController {
    private final UserService userService; // Stateless service
    
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        // Simple stateless request handling
        return userService.findById(id);
    }
}

// AVOID: Complex concurrency without clear need
public class ComplexConcurrentUserProcessor {
    private final ConcurrentHashMap<Long, CompletableFuture<User>> futures = new ConcurrentHashMap<>();
    // Complex threading logic that probably not needed
}
```

### Rule 16: Parsing with Recursive Descent
- Recursive descent most beautiful way to parse
- Parser generators create unreadable spaghetti code
- Hand-written parsers easier to understand and debug

```java
// GOOD: Simple recursive descent parser
public class JsonParser {
    private final String json;
    private int position = 0;
    
    public JsonValue parseValue() {
        skipWhitespace();
        char c = peek();
        
        if (c == '"') return parseString();
        if (c == '{') return parseObject();
        if (c == '[') return parseArray();
        if (Character.isDigit(c)) return parseNumber();
        
        throw new ParseException("Unexpected character: " + c);
    }
    
    private JsonString parseString() {
        expect('"');
        String value = readUntil('"');
        expect('"');
        return new JsonString(value);
    }
}
```

## Performance & Optimization

### Rule 17: Avoid Premature Optimization
- "Premature optimization is root of all evil" - wise saying
- Always profile before optimizing
- Often network, not CPU, is bottleneck
- Grug often surprised by what actually slow

```java
// GOOD: Profile first, optimize second
@Service
public class UserService {
    @Timed // Measure actual performance
    public List<User> findActiveUsers() {
        // Simple implementation first
        return userRepository.findAll()
            .stream()
            .filter(User::isActive)
            .collect(Collectors.toList());
    }
}

// If profiling shows this is slow, THEN optimize:
// @Query("SELECT u FROM User u WHERE u.active = true")
// List<User> findActiveUsers();
```

### Rule 18: Network Calls Are Expensive
- One network call = millions of CPU cycles
- Minimize network calls, especially in loops
- Batch operations when possible

```java
// BAD: Network call in loop
public void updateUserStatuses(List<Long> userIds) {
    for (Long userId : userIds) {
        User user = userService.findById(userId); // Network call each time
        user.setLastSeen(LocalDateTime.now());
        userService.save(user); // Another network call
    }
}

// GOOD: Batch operations
public void updateUserStatuses(List<Long> userIds) {
    List<User> users = userService.findAllById(userIds); // One network call
    users.forEach(user -> user.setLastSeen(LocalDateTime.now()));
    userService.saveAll(users); // One network call
}
```

## Front-end Development

### Rule 19: Avoid Over-Complex SPAs
- Many developers say "I know, I'll use React/Angular/Vue for everything"
- Now you have two complexity demons: backend AND frontend
- Simple HTML forms often sufficient
- JavaScript should enhance, not replace, basic functionality

```java
// GOOD: Simple server-side rendering
@Controller
public class UserController {
    @GetMapping("/users")
    public String listUsers(Model model) {
        List<User> users = userService.findAll();
        model.addAttribute("users", users);
        return "users/list"; // Simple HTML template
    }
    
    @PostMapping("/users")
    public String createUser(@ModelAttribute User user) {
        userService.save(user);
        return "redirect:/users";
    }
}
```

### Rule 20: Keep JavaScript Simple
- JavaScript natural habitat of complexity demon
- Use server-side rendering with progressive enhancement
- Avoid callback hell, promise chains, and complex state management

## Practical Examples

### Example 1: Simple Service Layer
```java
@Service
public class OrderService {
    private static final Logger logger = LoggerFactory.getLogger(OrderService.class);
    
    private final OrderRepository orderRepository;
    private final PaymentService paymentService;
    private final EmailService emailService;
    
    public OrderResult createOrder(CreateOrderRequest request) {
        logger.info("Creating order for user {}", request.getUserId());
        
        // Simple validation
        if (request.getItems().isEmpty()) {
            logger.warn("Empty order attempted by user {}", request.getUserId());
            return OrderResult.failed("Order must have items");
        }
        
        // Simple flow
        Order order = new Order(request.getUserId(), request.getItems());
        BigDecimal total = calculateTotal(order);
        
        PaymentResult payment = paymentService.processPayment(
            new PaymentRequest(request.getUserId(), total));
        
        if (payment.isSuccessful()) {
            order.setStatus(OrderStatus.CONFIRMED);
            order.setPaymentId(payment.getTransactionId());
            orderRepository.save(order);
            
            emailService.sendOrderConfirmation(order);
            logger.info("Order {} created successfully", order.getId());
            return OrderResult.success(order.getId());
        } else {
            logger.warn("Payment failed for order user {}", request.getUserId());
            return OrderResult.failed("Payment failed");
        }
    }
    
    private BigDecimal calculateTotal(Order order) {
        // Simple calculation, no complex pricing engine
        return order.getItems().stream()
            .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}
```

### Example 2: Simple Error Handling
```java
@RestController
public class UserController {
    private static final Logger logger = LoggerFactory.getLogger(UserController.class);
    
    @GetMapping("/users/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        logger.info("Fetching user {}", id);
        
        try {
            User user = userService.findById(id);
            return ResponseEntity.ok(user);
        } catch (UserNotFoundException e) {
            logger.warn("User {} not found", id);
            return ResponseEntity.notFound().build();
        } catch (Exception e) {
            logger.error("Error fetching user {}", id, e);
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }
}
```

## Remember These Grug Wisdoms

1. **Complexity bad** - Always choose simple solution
2. **"No" is magic word** - Reject unnecessary complexity
3. **Grug not ashamed of simple brain** - Simple often better
4. **Tools important** - Invest in good debugging tools
5. **Test important** - But don't let test shamans take over
6. **Log everything** - Future grug will thank present grug
7. **Profile before optimize** - Grug often wrong about performance
8. **Network calls expensive** - Minimize them
9. **Understand before change** - Respect existing code
10. **APIs should be simple** - Don't make grug think too much

> "grug say again and say often: complexity very, very bad"

When in doubt, choose the simple solution. Complexity demon always lurking, waiting to make code unmaintainable. Stay strong, fellow grug!
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rui-tx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
