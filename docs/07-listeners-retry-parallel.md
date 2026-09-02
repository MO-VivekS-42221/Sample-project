# 07 - Listeners, Retry Analyzer, and Parallel Execution

## Listener example
```java
import org.testng.ITestContext;
import org.testng.ITestListener;
import org.testng.ITestResult;

public class CustomListener implements ITestListener {
    public void onTestSuccess(ITestResult result) {
        System.out.println("PASSED: " + result.getName());
    }
    public void onTestFailure(ITestResult result) {
        System.out.println("FAILED: " + result.getName());
    }
    public void onStart(ITestContext context) {
        System.out.println("Started");
    }
    public void onFinish(ITestContext context) {
        System.out.println("Finished");
    }
}
```

## Retry Analyzer
```java
import org.testng.IRetryAnalyzer;
import org.testng.ITestResult;

public class RetryAnalyzer implements IRetryAnalyzer {
    private int count = 0;
    private static final int maxRetry = 2;

    public boolean retry(ITestResult result) {
        if (count < maxRetry) {
            count++;
            return true;
        }
        return false;
    }
}
```

## Parallel execution in XML
```xml
<suite name="ParallelSuite" parallel="tests" thread-count="2">
    <test name="Test1">
        <classes>
            <class name="tests.ChromeTest"/>
        </classes>
    </test>
    <test name="Test2">
        <classes>
            <class name="tests.FirefoxTest"/>
        </classes>
    </test>
</suite>
```
