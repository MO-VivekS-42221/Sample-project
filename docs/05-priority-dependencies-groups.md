# 05 - Priorities, Dependencies, and Groups

## Priority
Use when execution order matters loosely.

```java
import org.testng.annotations.Test;

public class PriorityExample {
    @Test(priority = 1)
    public void login() {}

    @Test(priority = 2)
    public void search() {}
}
```

## dependsOnMethods
Use when one test must finish successfully before another.

```java
import org.testng.annotations.Test;

public class DependencyExample {
    @Test
    public void launchBrowser() {}

    @Test(dependsOnMethods = "launchBrowser")
    public void openApplication() {}
}
```

## Groups
Useful for smoke, regression, sanity.

```java
import org.testng.annotations.Test;

public class GroupExample {
    @Test(groups = "smoke")
    public void smokeTest() {}

    @Test(groups = "regression")
    public void regressionTest() {}
}
```
