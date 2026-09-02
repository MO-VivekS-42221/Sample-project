# TestNG + Selenium in Java

This repository contains a practical learning guide for **TestNG with Java and Selenium**.

## What is TestNG?
TestNG is a Java testing framework used for unit, integration, and UI automation testing. It is especially useful with Selenium because it provides test execution control, annotations, assertions, grouping, parallel execution, and reporting.

## Why use TestNG with Selenium?
- Better test organization
- Setup and teardown support
- Grouping for smoke/regression suites
- Data-driven testing with `@DataProvider`
- Parameter support via `testng.xml`
- Parallel execution
- Retry mechanism for flaky tests
- Listener support for logging and reporting

## Core TestNG topics
- Annotations
- Assertions
- `testng.xml`
- Priorities and dependencies
- Groups
- Parameters and DataProvider
- Listeners
- Retry Analyzer
- Parallel execution
- Page Object Model integration

## Example Selenium test
```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;

public class SampleLoginTest {
    WebDriver driver;

    @BeforeMethod
    public void setUp() {
        driver = new ChromeDriver();
        driver.manage().window().maximize();
    }

    @Test
    public void openGoogle() {
        driver.get("https://www.google.com");
        Assert.assertTrue(driver.getTitle().contains("Google"));
    }

    @AfterMethod
    public void tearDown() {
        driver.quit();
    }
}
```

## Suggested learning path
1. Java basics
2. Selenium basics
3. TestNG annotations
4. Assertions
5. XML suite execution
6. Data-driven testing
7. Framework design with POM
8. Parallel and advanced execution

## Chapter files
- `docs/01-introduction.md`
- `docs/02-annotations.md`
- `docs/03-assertions.md`
- `docs/04-testng-xml.md`
- `docs/05-priority-dependencies-groups.md`
- `docs/06-parameters-dataprovider.md`
- `docs/07-listeners-retry-parallel.md`
- `docs/08-pom-selenium-examples.md`
- `docs/09-interview-notes.md`

## Interview notes
For a concise interview-focused version, see `docs/09-interview-notes.md`.

## How to run
- Add TestNG and Selenium dependencies to `pom.xml`
- Create a `testng.xml`
- Run tests using your IDE or Maven

## Recommended practice
Build a mini framework with:
- `BaseTest`
- `LoginPage`
- `LoginTest`
- `CustomListener`
- `RetryAnalyzer`
