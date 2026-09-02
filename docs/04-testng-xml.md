# 04 - TestNG XML Configuration

## Why use `testng.xml`?
It controls suite execution, classes, groups, parameters, and parallel behavior.

## Basic XML
```xml
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd" >
<suite name="SampleSuite">
    <test name="SampleTest">
        <classes>
            <class name="tests.IntroTest"/>
        </classes>
    </test>
</suite>
```

## Passing parameters
```xml
<suite name="ParamSuite">
    <test name="ParamTest">
        <parameter name="browser" value="chrome"/>
        <parameter name="url" value="https://example.com"/>
        <classes>
            <class name="tests.ParameterExample"/>
        </classes>
    </test>
</suite>
```
