# 03 - Assertions in TestNG

## Hard Assertion
Stops execution immediately on failure.

```java
import org.testng.Assert;
import org.testng.annotations.Test;

public class HardAssertExample {
    @Test
    public void verifyTitle() {
        Assert.assertEquals("Google", "Google");
    }
}
```

## Soft Assertion
Collects failures and reports them at the end.

```java
import org.testng.asserts.SoftAssert;
import org.testng.annotations.Test;

public class SoftAssertExample {
    @Test
    public void verifyMultipleThings() {
        SoftAssert softAssert = new SoftAssert();
        softAssert.assertTrue(false, "First check failed");
        softAssert.assertEquals("A", "B", "Second check failed");
        softAssert.assertAll();
    }
}
```

## Common assertion methods
- `assertEquals`
- `assertTrue`
- `assertFalse`
- `assertNull`
- `assertNotNull`
