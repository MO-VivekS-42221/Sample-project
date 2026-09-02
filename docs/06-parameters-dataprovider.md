# 06 - Parameters and DataProvider

## `@Parameters`
Pass values from `testng.xml`.

```java
import org.testng.annotations.Parameters;
import org.testng.annotations.Test;

public class ParameterExample {
    @Test
    @Parameters({"browser", "url"})
    public void launchApp(String browser, String url) {
        System.out.println(browser + " -> " + url);
    }
}
```

## `@DataProvider`
Used for data-driven testing.

```java
import org.testng.annotations.DataProvider;
import org.testng.annotations.Test;

public class DataProviderExample {
    @DataProvider(name = "loginData")
    public Object[][] loginData() {
        return new Object[][] {
            {"user1", "pass1"},
            {"user2", "pass2"}
        };
    }

    @Test(dataProvider = "loginData")
    public void loginTest(String username, String password) {
        System.out.println(username + " / " + password);
    }
}
```
