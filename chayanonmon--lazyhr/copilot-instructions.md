## lazyhr

> This guide helps you build a comprehensive HR management system using Spring Boot with **Unix timestamp-based architecture** and the following features:

# Employee Attendance and Leave Management System - Implementation Guide

## Overview
This guide helps you build a comprehensive HR management system using Spring Boot with **Unix timestamp-based architecture** and the following features:
- Employee Attendance Management (Clock-in, Clock-out, timestamp precision)
- Leave Management (AM/PM/Full day periods, multiple categories)
- User Authentication and Management  
- Persistent Database Integration with Long integer timestamp fields
- Unix Timestamp Architecture for timezone-independent operations

## Key Features

### 🕒 Unix Timestamp Architecture
- **Timezone-independent**: All date/time fields use Unix timestamps (milliseconds)
- **Mobile-friendly**: Better compatibility with mobile and web clients
- **Consistent formatting**: JavaScript utilities for client-side timestamp handling
- **Database efficiency**: Long integer storage for optimal performance

## System Requirements

### Attendance System
- **Clock-in/Clock-out**: Employees can record their daily attendance
- **Overtime (OT) Requests**: Request and manage overtime hours
- **Attendance Tracking**: View daily, weekly, and monthly attendance records

### Leave Management System
- **Leave Periods**: 
  - AM (Morning - Half day)
  - PM (Afternoon - Half day) 
  - Full Day
- **Leave Categories**:
  - Annual Leave
  - Private Leave
  - Sick Leave
  - Special Holiday
- **Leave Workflow**: Apply, approve/reject leave requests

### Core Features
- User Authentication (Login/Logout)
- User Management (Add/Remove employees)
- Role-based Access Control (Admin, Manager, Employee)
- Persistent Database Storage
- RESTful API endpoints
- Web Interface

## Technology Stack

### Backend
- **Spring Boot 3.x** - Main framework
- **Spring Security** - Authentication & Authorization
- **Spring Data JPA** - Database operations
- **Spring Web** - REST API
- **MySQL/PostgreSQL** - Database
- **Maven/Gradle** - Build tool

### Frontend (Optional)
- **Thymeleaf** - Server-side templating
- **Bootstrap** - CSS framework
- **JavaScript** - Client-side interactions

## Database Schema

### Users Table
```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    employee_id VARCHAR(20) UNIQUE NOT NULL,
    department VARCHAR(100),
    position VARCHAR(100),
    hire_date BIGINT, -- Unix timestamp in milliseconds
    salary DECIMAL(10,2),
    is_active BOOLEAN DEFAULT true,
    role ENUM('ADMIN', 'MANAGER', 'EMPLOYEE') DEFAULT 'EMPLOYEE',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### Attendance Table
```sql
CREATE TABLE attendance (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    attendance_date BIGINT NOT NULL, -- Unix timestamp in milliseconds
    clock_in_time BIGINT, -- Unix timestamp in milliseconds
    clock_out_time BIGINT, -- Unix timestamp in milliseconds
    break_duration_minutes INT DEFAULT 0,
    total_hours DECIMAL(4,2),
    overtime_hours DECIMAL(4,2) DEFAULT 0,
    status ENUM('PRESENT', 'ABSENT', 'LATE', 'HALF_DAY') DEFAULT 'PRESENT',
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id),
    UNIQUE KEY unique_user_date (user_id, attendance_date)
);
```

### Overtime Requests Table
```sql
CREATE TABLE overtime_requests (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    request_date DATE NOT NULL,
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP NOT NULL,
    hours_requested DECIMAL(4,2) NOT NULL,
    reason TEXT NOT NULL,
    status ENUM('PENDING', 'APPROVED', 'REJECTED') DEFAULT 'PENDING',
    approved_by BIGINT,
    approved_at TIMESTAMP NULL,
    comments TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (approved_by) REFERENCES users(id)
);
```

### Leave Requests Table
```sql
CREATE TABLE leave_requests (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    leave_category ENUM('ANNUAL', 'PRIVATE', 'SICK', 'SPECIAL_HOLIDAY') NOT NULL,
    start_date BIGINT NOT NULL, -- Unix timestamp in milliseconds
    end_date BIGINT NOT NULL, -- Unix timestamp in milliseconds
    leave_period ENUM('AM', 'PM', 'FULL_DAY') NOT NULL,
    total_days DECIMAL(3,1) NOT NULL,
    reason TEXT NOT NULL,
    status ENUM('PENDING', 'APPROVED', 'REJECTED') DEFAULT 'PENDING',
    applied_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    approved_by BIGINT,
    approved_date TIMESTAMP NULL,
    comments TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (approved_by) REFERENCES users(id)
);
```

### Leave Balance Table
```sql
CREATE TABLE leave_balance (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    year INT NOT NULL,
    annual_leave_allocated DECIMAL(4,1) DEFAULT 21.0,
    annual_leave_used DECIMAL(4,1) DEFAULT 0.0,
    sick_leave_allocated DECIMAL(4,1) DEFAULT 14.0,
    sick_leave_used DECIMAL(4,1) DEFAULT 0.0,
    private_leave_allocated DECIMAL(4,1) DEFAULT 5.0,
    private_leave_used DECIMAL(4,1) DEFAULT 0.0,
    special_holiday_used DECIMAL(4,1) DEFAULT 0.0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id),
    UNIQUE KEY unique_user_year (user_id, year)
);
```

## Implementation Steps

### Step 1: Project Setup

1. **Update build.gradle** with required dependencies:
```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    implementation 'mysql:mysql-connector-java'
    implementation 'org.springframework.boot:spring-boot-starter-mail'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.springframework.security:spring-security-test'
}
```

2. **Configure application.properties**:
```properties
# Database Configuration
spring.datasource.url=jdbc:mysql://localhost:3306/lazyhr_db
spring.datasource.username=your_username
spring.datasource.password=your_password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA Configuration
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

# Security Configuration
spring.security.user.name=admin
spring.security.user.password=admin123
spring.security.user.roles=ADMIN

# Server Configuration
server.port=8080
server.servlet.context-path=/lazyhr

# Thymeleaf Configuration
spring.thymeleaf.cache=false
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
```

### Step 2: Create Entity Classes

#### User Entity
```java
@Entity
@Table(name = "users")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true, nullable = false, length = 50)
    private String username;
    
    @Column(nullable = false)
    private String password;
    
    @Column(unique = true, nullable = false, length = 100)
    private String email;
    
    @Column(name = "first_name", nullable = false, length = 50)
    private String firstName;
    
    @Column(name = "last_name", nullable = false, length = 50)
    private String lastName;
    
    @Column(name = "employee_id", unique = true, nullable = false, length = 20)
    private String employeeId;
    
    @Column(length = 100)
    private String department;
    
    @Column(length = 100)
    private String position;
    
    @Column(name = "hire_date")
    private Long hireDate; // Unix timestamp in milliseconds
    
    @Column(precision = 10, scale = 2)
    private BigDecimal salary;
    
    @Column(name = "is_active")
    private boolean isActive = true;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private Role role = Role.EMPLOYEE;
    
    @CreationTimestamp
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    // One-to-many relationships
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JsonManagedReference
    private List<Attendance> attendances;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JsonManagedReference
    private List<LeaveRequest> leaveRequests;
    
    // Helper methods
    public String getFullName() {
        return firstName + " " + lastName;
    }
    
    public String getDisplayName() {
        return getFullName() + " (" + employeeId + ")";
    }
    
    // Constructors, getters, setters
}

enum Role {
    ADMIN, MANAGER, EMPLOYEE
}
```

#### Attendance Entity
```java
@Entity
@Table(name = "attendance")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Attendance {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    @JsonBackReference
    private User user;
    
    @Column(name = "attendance_date", nullable = false)
    private Long attendanceDate; // Unix timestamp in milliseconds
    
    @Column(name = "clock_in_time")
    private Long clockInTime; // Unix timestamp in milliseconds
    
    @Column(name = "clock_out_time")
    private Long clockOutTime; // Unix timestamp in milliseconds
    
    @Column(name = "break_duration_minutes")
    private Integer breakDurationMinutes = 0;
    
    @Column(name = "total_hours", precision = 4, scale = 2)
    private BigDecimal totalHours;
    
    @Column(name = "overtime_hours", precision = 4, scale = 2)
    private BigDecimal overtimeHours = BigDecimal.ZERO;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private AttendanceStatus status = AttendanceStatus.PRESENT;
    
    @Column(columnDefinition = "TEXT")
    private String notes;
    
    @CreationTimestamp
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    // Helper methods for timestamp conversion
    public LocalDate getAttendanceDateAsLocalDate() {
        return attendanceDate != null ? 
            Instant.ofEpochMilli(attendanceDate).atZone(ZoneId.systemDefault()).toLocalDate() : null;
    }
    
    public LocalDateTime getClockInTimeAsLocalDateTime() {
        return clockInTime != null ? 
            Instant.ofEpochMilli(clockInTime).atZone(ZoneId.systemDefault()).toLocalDateTime() : null;
    }
    
    public LocalDateTime getClockOutTimeAsLocalDateTime() {
        return clockOutTime != null ? 
            Instant.ofEpochMilli(clockOutTime).atZone(ZoneId.systemDefault()).toLocalDateTime() : null;
    }
    
    // Constructors, getters, setters
}

enum AttendanceStatus {
    PRESENT, ABSENT, LATE, HALF_DAY
}
```

#### Leave Request Entity
```java
@Entity
@Table(name = "leave_requests")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class LeaveRequest {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    @JsonBackReference
    private User user;
    
    @Column(name = "leave_category", nullable = false)
    @Enumerated(EnumType.STRING)
    private LeaveCategory leaveCategory;
    
    @Column(name = "start_date", nullable = false)
    private Long startDate; // Unix timestamp in milliseconds
    
    @Column(name = "end_date", nullable = false)
    private Long endDate; // Unix timestamp in milliseconds
    
    @Column(name = "leave_period", nullable = false)
    @Enumerated(EnumType.STRING)
    private LeavePeriod leavePeriod;
    
    @Column(name = "total_days", nullable = false, precision = 3, scale = 1)
    private BigDecimal totalDays;
    
    @Column(nullable = false, columnDefinition = "TEXT")
    private String reason;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private LeaveStatus status = LeaveStatus.PENDING;
    
    @Column(name = "applied_date")
    @CreationTimestamp
    private LocalDateTime appliedDate;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "approved_by")
    private User approvedBy;
    
    @Column(name = "approved_date")
    private LocalDateTime approvedDate;
    
    @Column(columnDefinition = "TEXT")
    private String comments;
    
    @CreationTimestamp
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    // Helper methods for timestamp conversion
    public LocalDate getStartDateAsLocalDate() {
        return startDate != null ? 
            Instant.ofEpochMilli(startDate).atZone(ZoneId.systemDefault()).toLocalDate() : null;
    }
    
    public LocalDate getEndDateAsLocalDate() {
        return endDate != null ? 
            Instant.ofEpochMilli(endDate).atZone(ZoneId.systemDefault()).toLocalDate() : null;
    }
    
    public void setStartDateFromLocalDate(LocalDate date) {
        this.startDate = date != null ? 
            date.atStartOfDay(ZoneId.systemDefault()).toInstant().toEpochMilli() : null;
    }
    
    public void setEndDateFromLocalDate(LocalDate date) {
        this.endDate = date != null ? 
            date.atStartOfDay(ZoneId.systemDefault()).toInstant().toEpochMilli() : null;
    }
    
    // Constructors, getters, setters
}

enum LeaveCategory {
    ANNUAL, PRIVATE, SICK, SPECIAL_HOLIDAY
}

enum LeavePeriod {
    AM, PM, FULL_DAY
}

enum LeaveStatus {
    PENDING, APPROVED, REJECTED
}
```

### Step 3: Create Repository Interfaces

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
    Optional<User> findByEmail(String email);
    Optional<User> findByEmployeeId(String employeeId);
    List<User> findByIsActiveTrue();
    List<User> findByDepartment(String department);
}

@Repository
public interface AttendanceRepository extends JpaRepository<Attendance, Long> {
    Optional<Attendance> findByUserAndAttendanceDate(User user, Long timestamp);
    List<Attendance> findByUserAndAttendanceDateBetween(User user, Long startTimestamp, Long endTimestamp);
    List<Attendance> findByAttendanceDateBetween(Long startTimestamp, Long endTimestamp);
    List<Attendance> findByUserOrderByAttendanceDateDesc(User user);
    
    @Query("SELECT a FROM Attendance a WHERE a.user.id = :userId AND a.clockOutTime IS NULL")
    Optional<Attendance> findActiveAttendance(@Param("userId") Long userId);
    
    @Query("SELECT a FROM Attendance a WHERE a.user.id = :userId AND a.attendanceDate = :timestamp")
    Optional<Attendance> findByUserIdAndAttendanceDate(@Param("userId") Long userId, @Param("timestamp") Long timestamp);
}

@Repository
public interface LeaveRequestRepository extends JpaRepository<LeaveRequest, Long> {
    List<LeaveRequest> findByUserOrderByAppliedDateDesc(User user);
    List<LeaveRequest> findByStatusOrderByAppliedDateDesc(LeaveStatus status);
    List<LeaveRequest> findByUserAndStartDateBetween(User user, Long startTimestamp, Long endTimestamp);
    
    @Query("SELECT lr FROM LeaveRequest lr WHERE lr.startDate <= :endTimestamp AND lr.endDate >= :startTimestamp AND lr.user = :user AND lr.status = 'APPROVED'")
    List<LeaveRequest> findOverlappingLeaves(@Param("user") User user, @Param("startTimestamp") Long startTimestamp, @Param("endTimestamp") Long endTimestamp);
    
    @Query("SELECT lr FROM LeaveRequest lr WHERE lr.user.id = :userId")
    List<LeaveRequest> findByUserId(@Param("userId") Long userId);
}
```

### Step 4: Create Service Classes

#### User Service
```java
@Service
@Transactional
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    public User createUser(User user) {
        user.setPassword(passwordEncoder.encode(user.getPassword()));
        return userRepository.save(user);
    }
    
    public User findByUsername(String username) {
        return userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));
    }
    
    public List<User> getAllActiveUsers() {
        return userRepository.findByIsActiveTrue();
    }
    
    public User updateUser(User user) {
        return userRepository.save(user);
    }
    
    public void deactivateUser(Long userId) {
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new EntityNotFoundException("User not found"));
        user.setActive(false);
        userRepository.save(user);
    }
}
```

#### Attendance Service
```java
@Service
@Transactional
public class AttendanceService {
    
    @Autowired
    private AttendanceRepository attendanceRepository;
    
    @Autowired
    private UserRepository userRepository;
    
    public Attendance clockIn(Long userId) {
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new EntityNotFoundException("User not found"));
        
        // Get today's timestamp (start of day)
        Long todayTimestamp = LocalDate.now().atStartOfDay(ZoneId.systemDefault()).toInstant().toEpochMilli();
        Optional<Attendance> existingAttendance = attendanceRepository.findByUserAndAttendanceDate(user, todayTimestamp);
        
        if (existingAttendance.isPresent()) {
            throw new IllegalStateException("Already clocked in today");
        }
        
        Long currentTimestamp = Instant.now().toEpochMilli();
        
        Attendance attendance = new Attendance();
        attendance.setUser(user);
        attendance.setAttendanceDate(todayTimestamp);
        attendance.setClockInTime(currentTimestamp);
        attendance.setStatus(AttendanceStatus.PRESENT);
        
        return attendanceRepository.save(attendance);
    }
    
    public Attendance clockOut(Long userId) {
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new EntityNotFoundException("User not found"));
        
        Attendance attendance = attendanceRepository.findActiveAttendance(userId)
            .orElseThrow(() -> new IllegalStateException("No active clock-in found"));
        
        Long currentTimestamp = Instant.now().toEpochMilli();
        attendance.setClockOutTime(currentTimestamp);
        
        // Calculate total hours using timestamps
        if (attendance.getClockInTime() != null) {
            long totalMinutes = (currentTimestamp - attendance.getClockInTime()) / (1000 * 60);
            totalMinutes -= attendance.getBreakDurationMinutes();
            BigDecimal totalHours = BigDecimal.valueOf(totalMinutes).divide(BigDecimal.valueOf(60), 2, RoundingMode.HALF_UP);
            attendance.setTotalHours(totalHours);
            
            // Calculate overtime (assuming 8 hours is standard work day)
            BigDecimal standardHours = BigDecimal.valueOf(8);
            if (totalHours.compareTo(standardHours) > 0) {
                attendance.setOvertimeHours(totalHours.subtract(standardHours));
            }
        }
        
        return attendanceRepository.save(attendance);
    }
    
    public List<Attendance> getAttendanceByUser(Long userId, Long startTimestamp, Long endTimestamp) {
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new EntityNotFoundException("User not found"));
        return attendanceRepository.findByUserAndAttendanceDateBetween(user, startTimestamp, endTimestamp);
    }
    
    // Helper method to get today's attendance
    public Optional<Attendance> getTodayAttendance(Long userId) {
        Long todayTimestamp = LocalDate.now().atStartOfDay(ZoneId.systemDefault()).toInstant().toEpochMilli();
        return attendanceRepository.findByUserIdAndAttendanceDate(userId, todayTimestamp);
    }
}
```

#### Leave Service
```java
@Service
@Transactional
public class LeaveService {
    
    @Autowired
    private LeaveRequestRepository leaveRequestRepository;
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private LeaveBalanceService leaveBalanceService;
    
    public LeaveRequest applyLeave(LeaveRequest leaveRequest) {
        // Validate leave balance
        if (!leaveBalanceService.hasEnoughBalance(leaveRequest)) {
            throw new IllegalStateException("Insufficient leave balance");
        }
        
        // Check for overlapping leaves using timestamps
        List<LeaveRequest> overlappingLeaves = leaveRequestRepository.findOverlappingLeaves(
            leaveRequest.getUser(), leaveRequest.getStartDate(), leaveRequest.getEndDate());
        
        if (!overlappingLeaves.isEmpty()) {
            throw new IllegalStateException("Leave request overlaps with existing approved leave");
        }
        
        // Calculate total days using timestamp helper methods
        BigDecimal totalDays = calculateLeaveDays(leaveRequest);
        leaveRequest.setTotalDays(totalDays);
        
        return leaveRequestRepository.save(leaveRequest);
    }
    
    public LeaveRequest approveLeave(Long leaveId, Long approverId, String comments) {
        LeaveRequest leaveRequest = leaveRequestRepository.findById(leaveId)
            .orElseThrow(() -> new EntityNotFoundException("Leave request not found"));
        
        User approver = userRepository.findById(approverId)
            .orElseThrow(() -> new EntityNotFoundException("Approver not found"));
        
        leaveRequest.setStatus(LeaveStatus.APPROVED);
        leaveRequest.setApprovedBy(approver);
        leaveRequest.setApprovedDate(LocalDateTime.now());
        leaveRequest.setComments(comments);
        
        // Update leave balance
        leaveBalanceService.updateLeaveBalance(leaveRequest);
        
        return leaveRequestRepository.save(leaveRequest);
    }
    
    private BigDecimal calculateLeaveDays(LeaveRequest leaveRequest) {
        // Convert timestamps to LocalDate for calculation
        LocalDate startDate = leaveRequest.getStartDateAsLocalDate();
        LocalDate endDate = leaveRequest.getEndDateAsLocalDate();
        
        if (startDate == null || endDate == null) {
            throw new IllegalArgumentException("Invalid start or end date");
        }
        
        long daysBetween = ChronoUnit.DAYS.between(startDate, endDate) + 1;
        
        if (leaveRequest.getLeavePeriod() == LeavePeriod.FULL_DAY) {
            return BigDecimal.valueOf(daysBetween);
        } else {
            // AM or PM is half day
            return BigDecimal.valueOf(daysBetween * 0.5);
        }
    }
}
```

### Step 5: Create REST Controllers

#### Attendance Controller
```java
@RestController
@RequestMapping("/api/attendance")
@CrossOrigin(origins = "*")
public class AttendanceController {
    
    @Autowired
    private AttendanceService attendanceService;
    
    @PostMapping("/clock-in")
    public ResponseEntity<Attendance> clockIn(@RequestParam Long userId) {
        try {
            Attendance attendance = attendanceService.clockIn(userId);
            return ResponseEntity.ok(attendance);
        } catch (Exception e) {
            return ResponseEntity.badRequest().build();
        }
    }
    
    @PostMapping("/clock-out")
    public ResponseEntity<Attendance> clockOut(@RequestParam Long userId) {
        try {
            Attendance attendance = attendanceService.clockOut(userId);
            return ResponseEntity.ok(attendance);
        } catch (Exception e) {
            return ResponseEntity.badRequest().build();
        }
    }
    
    @GetMapping("/user/{userId}")
    public ResponseEntity<List<Attendance>> getUserAttendance(
            @PathVariable Long userId,
            @RequestParam Long startTimestamp,
            @RequestParam Long endTimestamp) {
        
        List<Attendance> attendance = attendanceService.getAttendanceByUser(userId, startTimestamp, endTimestamp);
        return ResponseEntity.ok(attendance);
    }
    
    @GetMapping("/today/{userId}")
    public ResponseEntity<Attendance> getTodayAttendance(@PathVariable Long userId) {
        Optional<Attendance> attendance = attendanceService.getTodayAttendance(userId);
        return attendance.map(ResponseEntity::ok).orElse(ResponseEntity.notFound().build());
    }
}
```

#### Leave Controller
```java
@RestController
@RequestMapping("/api/leave")
@CrossOrigin(origins = "*")
public class LeaveController {
    
    @Autowired
    private LeaveService leaveService;
    
    @PostMapping("/apply")
    public ResponseEntity<LeaveRequest> applyLeave(@RequestBody LeaveRequest leaveRequest) {
        try {
            LeaveRequest savedRequest = leaveService.applyLeave(leaveRequest);
            return ResponseEntity.ok(savedRequest);
        } catch (Exception e) {
            return ResponseEntity.badRequest().build();
        }
    }
    
    @PostMapping("/{leaveId}/approve")
    public ResponseEntity<LeaveRequest> approveLeave(
            @PathVariable Long leaveId,
            @RequestParam Long approverId,
            @RequestParam(required = false) String comments) {
        
        try {
            LeaveRequest approvedRequest = leaveService.approveLeave(leaveId, approverId, comments);
            return ResponseEntity.ok(approvedRequest);
        } catch (Exception e) {
            return ResponseEntity.badRequest().build();
        }
    }
    
    @GetMapping("/user/{userId}")
    public ResponseEntity<List<LeaveRequest>> getUserLeaves(@PathVariable Long userId) {
        try {
            List<LeaveRequest> leaves = leaveService.getUserLeaves(userId);
            return ResponseEntity.ok(leaves);
        } catch (Exception e) {
            return ResponseEntity.badRequest().build();
        }
    }
    
    @GetMapping("/user/{userId}/range")
    public ResponseEntity<List<LeaveRequest>> getUserLeavesInRange(
            @PathVariable Long userId,
            @RequestParam Long startTimestamp,
            @RequestParam Long endTimestamp) {
        
        try {
            List<LeaveRequest> leaves = leaveService.getUserLeavesInRange(userId, startTimestamp, endTimestamp);
            return ResponseEntity.ok(leaves);
        } catch (Exception e) {
            return ResponseEntity.badRequest().build();
        }
    }
}
```

### Step 6: Security Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/manager/**").hasAnyRole("ADMIN", "MANAGER")
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            );
        
        return http.build();
    }
}
```

### Step 7: JavaScript Timestamp Utilities

Since the system uses Unix timestamps throughout, here are essential JavaScript utilities for handling timestamps in the frontend:

#### timestamp-utils.js
```javascript
/**
 * Unix Timestamp Utility Functions for LazyHR
 * All timestamps are in milliseconds
 */

class TimestampUtils {
    
    // Convert timestamp to readable date string
    static formatDate(timestamp) {
        if (!timestamp) return '';
        const date = new Date(timestamp);
        return date.toLocaleDateString();
    }
    
    // Convert timestamp to readable datetime string
    static formatDateTime(timestamp) {
        if (!timestamp) return '';
        const date = new Date(timestamp);
        return date.toLocaleString();
    }
    
    // Convert timestamp to time string (HH:MM)
    static formatTime(timestamp) {
        if (!timestamp) return '';
        const date = new Date(timestamp);
        return date.toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'});
    }
    
    // Get start of day timestamp
    static getStartOfDay(date = new Date()) {
        const startOfDay = new Date(date);
        startOfDay.setHours(0, 0, 0, 0);
        return startOfDay.getTime();
    }
    
    // Get end of day timestamp
    static getEndOfDay(date = new Date()) {
        const endOfDay = new Date(date);
        endOfDay.setHours(23, 59, 59, 999);
        return endOfDay.getTime();
    }
    
    // Convert date input value to timestamp
    static dateInputToTimestamp(dateString) {
        if (!dateString) return null;
        const date = new Date(dateString);
        return this.getStartOfDay(date);
    }
    
    // Convert timestamp to date input value (YYYY-MM-DD)
    static timestampToDateInput(timestamp) {
        if (!timestamp) return '';
        const date = new Date(timestamp);
        return date.toISOString().split('T')[0];
    }
    
    // Get current timestamp
    static now() {
        return Date.now();
    }
    
    // Calculate duration between timestamps in hours
    static calculateHours(startTimestamp, endTimestamp) {
        if (!startTimestamp || !endTimestamp) return 0;
        const durationMs = endTimestamp - startTimestamp;
        return (durationMs / (1000 * 60 * 60)).toFixed(2);
    }
    
    // Get timestamp for number of days from now
    static daysFromNow(days) {
        const date = new Date();
        date.setDate(date.getDate() + days);
        return date.getTime();
    }
    
    // Check if timestamp is today
    static isToday(timestamp) {
        if (!timestamp) return false;
        const today = new Date();
        const timestampDate = new Date(timestamp);
        return today.toDateString() === timestampDate.toDateString();
    }
}

// Export for use in other files
window.TimestampUtils = TimestampUtils;
```

### Step 8: Frontend Templates (Thymeleaf)

#### Main Dashboard (dashboard.html)
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>LazyHR - Dashboard</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
        <div class="container">
            <a class="navbar-brand" href="#">LazyHR</a>
            <div class="navbar-nav ms-auto">
                <span class="navbar-text me-3" th:text="'Welcome, ' + ${user.firstName}"></span>
                <a class="nav-link" href="/logout">Logout</a>
            </div>
        </div>
    </nav>

    <div class="container mt-4">
        <div class="row">
            <!-- Attendance Card -->
            <div class="col-md-6">
                <div class="card">
                    <div class="card-header">
                        <h5>Attendance</h5>
                    </div>
                    <div class="card-body">
                        <div class="d-grid gap-2">
                            <button class="btn btn-success" id="clockInBtn">Clock In</button>
                            <button class="btn btn-danger" id="clockOutBtn">Clock Out</button>
                            <button class="btn btn-warning">Request OT</button>
                        </div>
                        <div class="mt-3">
                            <small class="text-muted">Today's Status: <span id="attendanceStatus">Not Clocked In</span></small>
                        </div>
                    </div>
                </div>
            </div>
            
            <!-- Leave Card -->
            <div class="col-md-6">
                <div class="card">
                    <div class="card-header">
                        <h5>Leave Management</h5>
                    </div>
                    <div class="card-body">
                        <div class="d-grid gap-2">
                            <button class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#leaveModal">Apply Leave</button>
                            <button class="btn btn-info">View Leave History</button>
                            <button class="btn btn-secondary">Leave Balance</button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        
        <!-- Recent Activities -->
        <div class="row mt-4">
            <div class="col-12">
                <div class="card">
                    <div class="card-header">
                        <h5>Recent Activities</h5>
                    </div>
                    <div class="card-body">
                        <div class="table-responsive">
                            <table class="table table-striped">
                                <thead>
                                    <tr>
                                        <th>Date</th>
                                        <th>Type</th>
                                        <th>Details</th>
                                        <th>Status</th>
                                    </tr>
                                </thead>
                                <tbody id="activitiesTable">
                                    <!-- Dynamic content will be loaded here -->
                                </tbody>
                            </table>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Leave Application Modal -->
    <div class="modal fade" id="leaveModal" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title">Apply for Leave</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
                </div>
                <form id="leaveForm">
                    <div class="modal-body">
                        <div class="mb-3">
                            <label for="leaveCategory" class="form-label">Leave Category</label>
                            <select class="form-select" id="leaveCategory" required>
                                <option value="">Select Category</option>
                                <option value="ANNUAL">Annual Leave</option>
                                <option value="PRIVATE">Private Leave</option>
                                <option value="SICK">Sick Leave</option>
                                <option value="SPECIAL_HOLIDAY">Special Holiday</option>
                            </select>
                        </div>
                        <div class="mb-3">
                            <label for="leavePeriod" class="form-label">Leave Period</label>
                            <select class="form-select" id="leavePeriod" required>
                                <option value="">Select Period</option>
                                <option value="FULL_DAY">Full Day</option>
                                <option value="AM">Morning (AM)</option>
                                <option value="PM">Afternoon (PM)</option>
                            </select>
                        </div>
                        <div class="mb-3">
                            <label for="startDate" class="form-label">Start Date</label>
                            <input type="date" class="form-control" id="startDate" required>
                        </div>
                        <div class="mb-3">
                            <label for="endDate" class="form-label">End Date</label>
                            <input type="date" class="form-control" id="endDate" required>
                        </div>
                        <div class="mb-3">
                            <label for="reason" class="form-label">Reason</label>
                            <textarea class="form-control" id="reason" rows="3" required></textarea>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
                        <button type="submit" class="btn btn-primary">Submit Application</button>
                    </div>
                </form>
            </div>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
    <script src="/js/timestamp-utils.js"></script>
    <script src="/js/dashboard.js"></script>
</body>
</html>
```

### Step 9: JavaScript for Frontend Interactions

#### dashboard.js
```javascript
document.addEventListener('DOMContentLoaded', function() {
    // Load today's attendance status
    loadTodayAttendance();
    
    // Clock In functionality
    document.getElementById('clockInBtn').addEventListener('click', function() {
        fetch('/api/attendance/clock-in', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({ userId: getCurrentUserId() })
        })
        .then(response => response.json())
        .then(data => {
            alert('Successfully clocked in at ' + TimestampUtils.formatTime(data.clockInTime));
            updateAttendanceStatus('Clocked In');
            loadTodayAttendance();
        })
        .catch(error => {
            alert('Error clocking in: ' + error.message);
        });
    });

    // Clock Out functionality
    document.getElementById('clockOutBtn').addEventListener('click', function() {
        fetch('/api/attendance/clock-out', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({ userId: getCurrentUserId() })
        })
        .then(response => response.json())
        .then(data => {
            alert('Successfully clocked out at ' + TimestampUtils.formatTime(data.clockOutTime));
            updateAttendanceStatus('Clocked Out - Total: ' + data.totalHours + ' hours');
            loadTodayAttendance();
        })
        .catch(error => {
            alert('Error clocking out: ' + error.message);
        });
    });

    // Leave application form
    document.getElementById('leaveForm').addEventListener('submit', function(e) {
        e.preventDefault();
        
        // Convert date inputs to timestamps
        const startDateInput = document.getElementById('startDate').value;
        const endDateInput = document.getElementById('endDate').value;
        
        const leaveData = {
            userId: getCurrentUserId(),
            leaveCategory: document.getElementById('leaveCategory').value,
            leavePeriod: document.getElementById('leavePeriod').value,
            startDate: TimestampUtils.dateInputToTimestamp(startDateInput),
            endDate: TimestampUtils.dateInputToTimestamp(endDateInput),
            reason: document.getElementById('reason').value
        };

        fetch('/api/leave/apply', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(leaveData)
        })
        .then(response => response.json())
        .then(data => {
            alert('Leave application submitted successfully!');
            document.getElementById('leaveForm').reset();
            bootstrap.Modal.getInstance(document.getElementById('leaveModal')).hide();
        })
        .catch(error => {
            alert('Error submitting leave application: ' + error.message);
        });
    });
});

function loadTodayAttendance() {
    fetch(`/api/attendance/today/${getCurrentUserId()}`)
        .then(response => {
            if (response.ok) {
                return response.json();
            }
            return null;
        })
        .then(data => {
            if (data) {
                let status = 'Clocked In at ' + TimestampUtils.formatTime(data.clockInTime);
                if (data.clockOutTime) {
                    status = 'Completed - Total: ' + data.totalHours + ' hours';
                }
                updateAttendanceStatus(status);
            } else {
                updateAttendanceStatus('Not Clocked In');
            }
        })
        .catch(error => {
            console.error('Error loading today\'s attendance:', error);
        });
}

function getCurrentUserId() {
    // This should be implemented based on your authentication system
    return 1; // Placeholder
}

function updateAttendanceStatus(status) {
    document.getElementById('attendanceStatus').textContent = status;
}
```

## Testing Strategy

### Unit Tests
- Test all service methods
- Test repository queries
- Test business logic

### Integration Tests
- Test REST API endpoints
- Test database operations
- Test authentication flows

### Example Test Class
```java
@SpringBootTest
@TestPropertySource(locations = "classpath:application-test.properties")
public class AttendanceServiceTest {
    
    @Autowired
    private AttendanceService attendanceService;
    
    @MockBean
    private UserRepository userRepository;
    
    @MockBean
    private AttendanceRepository attendanceRepository;
    
    @Test
    public void testClockIn_Success() {
        // Test implementation
    }
    
    @Test
    public void testClockIn_AlreadyClockedIn() {
        // Test implementation
    }
}
```

## Deployment Instructions

### Development Environment
1. Install MySQL/PostgreSQL
2. Create database `lazyhr_db`
3. Update `application.properties` with database credentials
4. Run `./gradlew bootRun`

### Production Deployment
1. Configure production database
2. Set environment variables for sensitive data
3. Build with `./gradlew build`
4. Deploy JAR file to server
5. Configure reverse proxy (nginx/Apache)

## API Documentation

The LazyHR system provides a comprehensive REST API with Unix timestamp-based endpoints for all date/time operations.

### Attendance Endpoints
- `POST /api/attendance/clock-in` - Clock in with current timestamp
  - Request: `{ "userId": 1 }`
  - Response: Attendance object with clockInTime as Unix timestamp
- `POST /api/attendance/clock-out` - Clock out with current timestamp
  - Request: `{ "userId": 1 }`
  - Response: Attendance object with clockOutTime and calculated hours
- `GET /api/attendance/user/{userId}?startTimestamp={timestamp}&endTimestamp={timestamp}` - Get user attendance in date range
  - Parameters: startTimestamp and endTimestamp as Unix timestamps in milliseconds
- `GET /api/attendance/today/{userId}` - Get today's attendance record
- `GET /api/attendance/history/{userId}` - Get attendance history
- `GET /api/attendance/status/{userId}` - Get current attendance status
- `PUT /api/attendance/{attendanceId}/notes` - Update attendance notes
- `POST /api/attendance/{attendanceId}/break` - Add break duration
- `GET /api/attendance/stats/today` - Get today's attendance statistics

### Leave Endpoints
- `POST /api/leave/apply` - Apply for leave with timestamp dates
  - Request body includes startDate and endDate as Unix timestamps
- `GET /api/leave/user/{userId}` - Get user's leave requests
- `GET /api/leave/user/{userId}/range?startTimestamp={timestamp}&endTimestamp={timestamp}` - Get leaves in date range
- `POST /api/leave/{leaveId}/approve` - Approve leave request
- `POST /api/leave/{leaveId}/reject` - Reject leave request
- `PUT /api/leave/{leaveId}` - Update leave request
- `DELETE /api/leave/{leaveId}` - Cancel leave request
- `GET /api/leave/pending` - Get pending leave requests
- `GET /api/leave/balance/{userId}` - Get user's leave balance
- `GET /api/leave/holidays` - Get company holidays

### User Management Endpoints
- `POST /api/users` - Create new user
  - hireDate field accepts Unix timestamp
- `GET /api/users` - Get all active users
- `GET /api/users/{userId}` - Get user by ID
- `PUT /api/users/{userId}` - Update user information
- `DELETE /api/users/{userId}` - Deactivate user
- `GET /api/users/search?query={searchTerm}` - Search users
- `GET /api/users/check-username?username={username}` - Check username availability
- `GET /api/users/check-email?email={email}` - Check email availability
- `GET /api/users/departments` - Get all departments
- `POST /api/users/{userId}/reset-password` - Reset user password

### Timestamp Format
All timestamps in the API are Unix timestamps in milliseconds (Long values). Use the JavaScript `Date.now()` or `new Date().getTime()` methods to generate compatible timestamps.

## Security Considerations

1. **Input Validation**: Validate all user inputs
2. **Authentication**: Implement JWT or session-based auth
3. **Authorization**: Role-based access control
4. **Data Protection**: Encrypt sensitive data
5. **Audit Trail**: Log all critical operations
6. **Rate Limiting**: Prevent API abuse

## Performance Optimization

1. **Database Indexing**: Add indexes on frequently queried columns
2. **Caching**: Implement Redis for session management
3. **Connection Pooling**: Configure database connection pooling
4. **Pagination**: Implement pagination for large datasets
5. **Lazy Loading**: Use JPA lazy loading appropriately

## Monitoring and Logging

1. **Application Logs**: Use SLF4J with Logback
2. **Database Monitoring**: Monitor query performance
3. **Health Checks**: Implement actuator endpoints
4. **Metrics**: Use Micrometer for application metrics

## Future Enhancements

1. **Mobile App**: React Native/Flutter mobile application
2. **Reports**: Advanced reporting and analytics
3. **Notifications**: Email/SMS notifications
4. **Integration**: LDAP/Active Directory integration
5. **Payroll**: Integration with payroll systems
6. **Biometric**: Fingerprint/face recognition for attendance
7. **Geofencing**: Location-based attendance tracking

## Conclusion

This comprehensive guide provides you with everything needed to build a robust employee attendance and leave management system with **Unix timestamp architecture**. The system is designed for modern web and mobile applications with timezone-independent operations.

### Key Implementation Highlights
- **Unix Timestamp Fields**: All date/time data stored as Long (milliseconds) for consistency
- **Helper Methods**: Convert between timestamps and Java date objects when needed
- **JavaScript Utilities**: Frontend timestamp formatting and conversion functions
- **API Consistency**: All endpoints use timestamp parameters for date ranges
- **Database Efficiency**: Long integer storage for optimal query performance

### Development Best Practices
- Use the provided TimestampUtils class for all frontend date operations
- Always validate timestamp inputs in controllers
- Implement proper timezone handling for display purposes
- Use helper methods in entities for LocalDate/LocalDateTime conversions
- Follow Spring Boot best practices for REST API design
- Write comprehensive tests covering timestamp edge cases
- Implement proper security measures with role-based access
- Monitor application performance, especially timestamp queries
- Keep user experience simple with readable date formats

### Mobile App Considerations
The Unix timestamp architecture makes this system particularly suitable for mobile app integration:
- Consistent timestamp handling across platforms
- No timezone conversion issues between client and server
- Simple API contracts with Long timestamp parameters
- Better offline synchronization capabilities

For any specific implementation questions about the timestamp architecture or other features, refer to the Spring Boot documentation or create specific tickets for detailed assistance.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ChayanonMon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ChayanonMon)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
