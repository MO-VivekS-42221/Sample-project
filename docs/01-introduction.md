# 01 - Introduction to TestNG

## What is TestNG?
TestNG is a Java testing framework that makes test execution flexible and manageable.

## Why it is used with Selenium
- Controls test flow
- Supports annotations
- Supports assertions
- Handles data-driven testing
- Supports parallel execution
- Provides better reports

## Key features
- `@Test` annotation
- `testng.xml`
- Groups
- Parameters
- DataProvider
- Listeners
- Retry Analyzer

## Minimal example
```java
import org.testng.annotations.Test;

public class IntroTest {
    @Test
    public void sample() {
        System.out.println("TestNG is running");
    }
}
```
