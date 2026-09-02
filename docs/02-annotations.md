# 02 - TestNG Annotations

## Common annotations
- `@BeforeSuite`
- `@AfterSuite`
- `@BeforeTest`
- `@AfterTest`
- `@BeforeClass`
- `@AfterClass`
- `@BeforeMethod`
- `@AfterMethod`
- `@Test`
- `@DataProvider`
- `@Parameters`

## Execution order
1. `@BeforeSuite`
2. `@BeforeTest`
3. `@BeforeClass`
4. `@BeforeMethod`
5. `@Test`
6. `@AfterMethod`
7. `@AfterClass`
8. `@AfterTest`
9. `@AfterSuite`

## Example
```java
import org.testng.annotations.*;

public class AnnotationExample {
    @BeforeMethod
    public void beforeMethod() {
        System.out.println("Before Method");
    }

    @Test
    public void testOne() {
        System.out.println("Test One");
    }

    @AfterMethod
    public void afterMethod() {
        System.out.println("After Method");
    }
}
```
