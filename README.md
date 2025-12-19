# Selenium Test Automation Framework - Advanced Topics

A comprehensive guide to advanced Selenium WebDriver concepts including TestNG Listeners, Log4j2 Logging, and Parallel Test Execution using ThreadLocal.

---

## Overview

This framework demonstrates advanced Selenium automation techniques with:
- **Page Object Model (POM)** design pattern
- **TestNG Listeners** for test lifecycle management
- **Log4j2** for comprehensive logging
- **Parallel test execution** with ThreadLocal for thread safety

---

## Project Structure

```
selenium-pom-framework/
│
├── src/
│   ├── main/java/
│   │   └── org/example/
│   │       └── LogUtils.java              # Logging utility
│   │
│   └── test/
│       ├── java/
│       │   ├── base/
│       │   │   └── BaseTest.java          # ThreadLocal WebDriver setup
│       │   │
│       │   ├── CustomListners/
│       │   │   └── TestNGListeners.java   # Custom TestNG listeners
│       │   │
│       │   ├── Pages/
│       │   │   ├── HomePage.java
│       │   │   ├── LoginPage.java
│       │   │   └── AccountCreationPage.java
│       │   │
│       │   └── tests/
│       │       ├── LoginTest.java
│       │       └── RegisterTest.java
│       │
│       └── resources/
│           ├── log4j2.properties          # Log4j2 configuration
│           └── META-INF/services/
│               └── org.testng.ITestNGListener
│
├── test-outputs/
│   └── Logs/                              # Generated log files
│
├── screenshots/                           # Failure screenshots
│
├── testng.xml                             # TestNG suite configuration
└── pom.xml                                # Maven dependencies
```

---

## Prerequisites

Add the following dependencies to your `pom.xml`:
- Selenium WebDriver (4.21.0)
- TestNG (7.9.0)
- Log4j2 Core & API (3.0.0-alpha1)
- Log4j2 SLF4J Implementation
- Commons IO (2.14.0) for screenshot handling
- WebDriverManager (5.7.0)

---

## TestNG Listeners

TestNG Listeners allow you to hook into the test execution lifecycle and perform custom actions like logging, screenshots, and reporting.

### Step 1: Understanding Listener Interfaces

TestNG provides several listener interfaces:

- **ITestListener**: Listens to test events (success, failure, skip)
- **IInvokedMethodListener**: Listens to method invocations (before/after each test method)
- **IExecutionListener**: Listens to suite execution start/finish
- **IRetryAnalyzer**: Automatically retries failed tests (optional)

### Step 2: Create Your Custom Listener Class

Create `TestNGListeners.java` in the `CustomListners` package.

**Key methods to implement:**

```java
// From ITestListener
onTestSuccess(ITestResult result)      // Called when test passes
onTestFailure(ITestResult result)      // Called when test fails
onTestSkipped(ITestResult result)      // Called when test is skipped

// From IInvokedMethodListener  
beforeInvocation(IInvokedMethod method, ITestResult testResult)
afterInvocation(IInvokedMethod method, ITestResult testResult)

// From IExecutionListener
onExecutionStart()                     // Called before suite starts
onExecutionFinish()                    // Called after suite ends
```

**Important features in the listener:**
- Log test results using `LogUtils.logger()`
- Capture screenshots on test failure
- Get WebDriver from test instance: `((BaseTest) result.getInstance()).getDriver()`
- Save screenshots to `screenshots/` folder using Apache Commons IO

### Step 3: Register Listeners

**Method 1: Using testng.xml (Recommended)**

Add listener to your `testng.xml`:

```xml
<suite name="AutomationPracticeSuite">
    <listeners>
        <listener class-name="CustomListners.TestNGListeners"/>
    </listeners>
</suite>
```

**Method 2: Using @Listeners Annotation**

```java
@Listeners(CustomListners.TestNGListeners.class)
public class LoginTest extends BaseTest {
    // Test methods
}
```

**Method 3: Using Service Provider Configuration**

Create file: `src/test/resources/META-INF/services/org.testng.ITestNGListener`

Content:
```
CustomListners.TestNGListeners
```

### Step 4: Key Listener Methods Explained

| Method | When It's Called | Use Case |
|--------|-----------------|----------|
| `onTestSuccess()` | After a test passes | Log success messages |
| `onTestFailure()` | After a test fails | Take screenshots, log errors |
| `onTestSkipped()` | When a test is skipped | Log skip reasons |
| `beforeInvocation()` | Before any @Test method | Setup, logging |
| `afterInvocation()` | After any @Test method | Cleanup, reporting |
| `onExecutionStart()` | Before entire suite starts | Initialize reports |
| `onExecutionFinish()` | After entire suite ends | Finalize reports |

### Step 5: Screenshot Capture Logic

In `onTestFailure()` method:
1. Get the test instance from `ITestResult`
2. Cast to `BaseTest` and get WebDriver
3. Use `TakesScreenshot` interface to capture screenshot
4. Save to file using `FileUtils.copyFile()`

---

## Log4j2 Logging Configuration

Proper logging is crucial for debugging and monitoring test execution.

### Step 1: Create log4j2.properties

Create `log4j2.properties` in `src/test/resources/`

**Configuration structure:**
- **Property for log location**: `property.basePath = test-outputs/Logs`
- **Console Appender**: Colored output for terminal
- **File Appender**: Detailed logs with timestamp
- **Root Logger**: Connects both appenders

### Step 2: Console Appender Configuration

**Key settings:**
- Type: Console
- Layout: PatternLayout
- Pattern with colors and styles using `%highlight` and `%style`

**Pattern components:**
- `%d{HH:mm:ss}`: Timestamp
- `%highlight{%-5level}`: Colored log level (INFO=Green, ERROR=Red)
- `%t`: Thread name
- `%c{1}`: Class name (abbreviated)
- `%msg`: The log message
- `%n`: New line

### Step 3: File Appender Configuration

**Key settings:**
- Type: File
- FileName: `${basePath}/log_${date:yyyy-MM-dd_hh-mm-ss a}.log`
- Layout: PatternLayout with aligned columns

**Pattern components:**
- `%d{yyyy-MM-dd HH:mm:ss}`: Full timestamp
- `%-5level`: Fixed width (5 chars) for level
- `%-20.20t`: Fixed width (20 chars) for thread name
- `%-30.30c{1}`: Fixed width (30 chars) for class name
- Creates aligned columns for easy reading

### Step 4: Create LogUtils Helper Class

Create `LogUtils.java` in `org.example` package.

**Key method:**
```java
public static Logger logger() {
    // [3] gets the calling class from stack trace
    return LogManager.getLogger(Thread.currentThread().getStackTrace()[3].getClassName());
}
```

**Utility methods provided:**
- `info(String message)`
- `error(String message)`
- `warn(String message)`
- `fatal(String message)`
- `debug(String message)`

### Step 5: Using Logs in Tests

Import and use in your test classes:

```java
import static org.example.LogUtils.logger;

logger().info("Starting login test");
logger().debug("Navigated to homepage");
logger().error("Login failed");
```

### Step 6: Log Output Examples

**Console Output (Colored):**
```
14:05:42 INFO  [main] TestNG - Execution Started
14:05:44 INFO  [RegisterAndLoginTest-1] WebDriverManager - Using chromedriver
14:05:53 ERROR [RegisterAndLoginTest-1] TestListenerHelper - Test Failed
```

**File Output (Aligned Columns):**
```
2025-12-09 14:05:42 | INFO  | main                 | TestNG          | Execution Started 
2025-12-09 14:05:53 | ERROR | LoginTest-1          | TestListener    | Test Failed
```

---

## Parallel Execution with ThreadLocal

Parallel execution speeds up test suite execution but requires thread-safe WebDriver management.

### Step 1: Understanding the Problem

**Without ThreadLocal:**
- Multiple threads share the same WebDriver instance
- Causes race conditions and test interference
- Tests fail unpredictably
- Browser actions overlap

**Solution:** Use `ThreadLocal` to give each thread its own WebDriver instance.

### Step 2: Implement ThreadLocal in BaseTest

Create `BaseTest.java` in `base` package.

**Key components:**

```java
protected WebDriver driver;
protected static ThreadLocal<WebDriver> threadDriver = new ThreadLocal<>();
```

**In @BeforeClass:**
1. Setup WebDriver using WebDriverManager
2. Create ChromeDriver instance
3. Store in ThreadLocal: `threadDriver.set(driver)`
4. Configure browser (maximize, implicit wait)

**Thread-safe getter:**
```java
public WebDriver getDriver() {
    return threadDriver.get();  // Returns current thread's driver
}
```

**In @AfterClass:**
1. Quit the driver
2. **Important**: Clean up ThreadLocal with `threadDriver.remove()`

### Step 3: Understanding ThreadLocal

**What is ThreadLocal?**
- Java class that provides thread-local variables
- Each thread has its own isolated copy
- No synchronization needed

**How it works:**
- Thread 1: `threadDriver.set(driver1)` → `threadDriver.get()` returns driver1
- Thread 2: `threadDriver.set(driver2)` → `threadDriver.get()` returns driver2
- Both run simultaneously without interference

### Step 4: Configure Parallel Execution in testng.xml

```xml
<suite name="AutomationPracticeSuite" parallel="classes" thread-count="2">
```

**Parallel Execution Options:**

| Option | Description | Use When |
|--------|-------------|----------|
| `parallel="tests"` | Runs `<test>` tags in parallel | Multiple test suites |
| `parallel="classes"` | Runs test classes in parallel | Independent test classes |
| `parallel="methods"` | Runs test methods in parallel | Methods within same class |
| `thread-count="2"` | Number of parallel threads | Based on CPU cores |

### Step 5: Test Classes Using BaseTest

All test classes extend `BaseTest`:
- `LoginTest.java` extends BaseTest
- `RegisterTest.java` extends BaseTest

**In @BeforeClass:**
- Initialize page objects using `getDriver()` instead of `driver`
- Example: `homePage = new HomePage(getDriver())`

### Step 6: ThreadLocal Best Practices

** DO:**
- Always use `getDriver()` instead of `driver` directly
- Call `threadDriver.remove()` in `@AfterClass` to prevent memory leaks
- Use `protected static ThreadLocal` for inheritance
- Pass `getDriver()` to page object constructors

** DON'T:**
- Use instance variable `driver` directly in parallel tests
- Forget to clean up ThreadLocal with `remove()`
- Share WebDriver instances between threads
- Access ThreadLocal from different thread contexts

### Step 7: Parallel Execution Flow

```
Suite Start
    ↓
Thread 1: RegisterTest              Thread 2: LoginTest
    ↓                                       ↓
@BeforeClass                           @BeforeClass
    threadDriver.set(driver1)              threadDriver.set(driver2)
    ↓                                       ↓
@Test (testRegister)                   @Test (testLogin)
    Uses driver1                           Uses driver2
    ↓                                       ↓
@AfterClass                            @AfterClass
    driver1.quit()                         driver2.quit()
    threadDriver.remove()                  threadDriver.remove()
    ↓                                       ↓
Suite End
```

---
