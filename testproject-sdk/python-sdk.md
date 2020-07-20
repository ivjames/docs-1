---
description: TestProject SDK For Python
---

# Python SDK

## Getting started

To get started, you need to complete the following prerequisites checklist:

* Login to your account at [https://app.testproject.io/](https://app.testproject.io/) or register a new one.
* [Download](https://app.testproject.io/#/download) and install an Agent for your operating system or pull a container from [Docker Hub](https://hub.docker.com/r/testproject/agent).
* Run the Agent and [register](https://docs.testproject.io/getting-started/installation-and-setup#register-the-agent) it with your Account.
* Get a development token from the [Integrations / SDK](https://app.testproject.io/#/integrations/sdk) page.

## Installation

The TestProject Python SDK is [available on PyPI](https://pypi.org/project/testproject-python-sdk/). All you need to do is add it as a Python module using:

```text
pip install testproject-python-sdk
```

and you're good to go.

## Drivers

The TestProject SDK overrides standard Selenium/Appium drivers with extended functionality.

The examples shown in this document are based on Chrome. The SDK works in the same way for all other supported browsers:

* Firefox
* Safari
* Edge
* Internet Explorer
* Android apps \(using Appium\)
* iOS apps \(using Appium\)

Using a TestProject driver is identical to using a Selenium driver. Once you have added the SDK as a dependency to your project, changing the import statement is enough in most cases.

You can create a TestProject-powered version of a test using Chrome by using the TestProject Chrome driver:

```text
# from selenium import webdriver  <-- replace this import
from src.testproject.sdk.drivers import webdriver

def test_create_a_chrome_driver_instance():
    driver = webdriver.Chrome()
    # Your test code goes here
    driver.quit()
```

Here's an example of a complete test that is using the Chrome driver from the TestProject SDK:

```text
from src.testproject.sdk.drivers import webdriver

if __name__ == "__main__":
    driver = webdriver.Chrome()

    driver.get("https://example.testproject.io/web/")

    driver.find_element_by_css_selector("#name").send_keys("John Smith")
    driver.find_element_by_css_selector("#password").send_keys("12345")
    driver.find_element_by_css_selector("#login").click()

    passed = driver.find_element_by_css_selector("#logout").is_displayed()

    print("Test passed") if passed else print("Test failed")

    driver.quit()
```

## Development token

The SDK uses a development token for communication with the Agent and the TestProject platform. To configure your development token for use with the SDK, you have to specify it in an environment variable `TP_DEV_TOKEN`.

Alternatively, you can pass in your developer token as an argument to the driver constructor:

```text
def test_create_a_chrome_driver_instance():
    driver = webdriver.Chrome(token='YOUR_TOKEN_GOES_HERE')
    # Your test code goes here
    driver.quit()
```

## TestProject Agent

By default, drivers communicate with the local Agent listening on [http://localhost:8585](http://localhost:8585/). This value can be overridden by setting the `TP_AGENT_URL` environment variable to the correct Agent address.

## Driver command reporting

By default, the TestProject SDK reports all executed driver commands and their results to the TestProject Cloud. This allows us to create and display detailed HTML reports and statistics in your project dashboards.

This functionality can be disabled if desired:

```text
def test_disable_automatic_reporting():
    driver = webdriver.Chrome()
    driver.report().disable_command_reports(True)
    # From here on, driver commands will not be reported automatically
    driver.quit()
```

## Driver command report redaction

When driver command are being reported, the SDK will, by default, replaces the values typed into sensitive elements by replacing the actual text with three asterisks \(`***`\) in the report. Elements are considered sensitive if they:

* have an attribute `type` with value `password` \(all browsers and platforms\)
* are of type `XCUIElementTypeSecureTextField` \(iOS / XCUITest only\)

This redaction of sensitive commands can be disabled, if desired:

```text
def test_disable_driver_command_report_redaction():
    driver = webdriver.Chrome()
    driver.report().disable_redaction(True)
    # From here on, driver commands will not be redacted
    driver.quit()
```

## Test reports

Tests are reported automatically when the driver quits. You can specify a custom name for your test using the `@report` decorator:

```text
from src.testproject.decorator import report

@report(test='Your custom test name here')
def test_specify_test_name_in_decorator():
    driver = webdriver.Chrome()
    # Your test code goes here
    driver.quit()
```

If no test name is specified using the decorator, the test method name will be used as the test name in the report.

You can disable the automatic reporting of tests as well:

```text
def test_disable_automatic_test_reporting():
    driver = webdriver.Chrome()
    driver.report().disable_auto_test_reports(True)
    # Tests will not be reported automatically from here on
    driver.quit()
```

In addition to this, you can also manually report a test:

```text
def test_report_a_custom_test():
    driver = webdriver.Chrome()
    driver.report().test(name='My custom test name', passed=True, message='A custom message')
    driver.quit()
```

## Switching reporting on or off

If you want to temporarily disable and later reenable all reporting for a section of a test, you can do that, too:

```text
def test_temporarily_disable_all_reporting_then_reenable_it_later():
    driver = webdriver.Chrome()
    driver.report().disable_reports(True)
    driver.find_element_by_id('your_element_id').click()  # This statement will not be reported
    driver.report().disable_reports(False)
    driver.quit()
```

## Disable all reporting for a test

Finally, you can also prevent the Agent from creating a test report on TestProject at by setting the `disable_reports` flag in the driver constructor:

```text
def test_do_not_create_a_report_at_all():
    driver = webdriver.Chrome(disable_reports=True)
    # No reporting will be done at all for this test
    driver.quit()
```

Please note that reporting **can not be reenabled** at a later point for this specific driver instance.

## Specifying project and job names

There are different ways to specify custom project and job names for use in your reports. In order of precedence, these are:

1. Similar to the test name, you can also use the `@report` decorator to specify a custom project and job name:

```text
from src.testproject.decorator import report

@report(project='My project', job='My job')
def test_specify_project_and_job_name_in_decorator():
    driver = webdriver.Chrome()
    # Your test code goes here
    driver.quit()
```

1. You can also specify custom project and job names by passing them as arguments to your driver constructor:

```text
def test_specify_project_and_job_names_in_driver_constructor():
    driver = webdriver.Chrome(projectname='My custom project', jobname='My custom job')
    # Your test code goes here
    driver.quit()
```

1. If neither of the above options is used, the SDK will attempt to automatically infer project and job names from your package and test module names. This is only supported for **pytest** and **unittest**.

   > * For **pytest**, tests in the `my_tests.py` module in the `e2e_tests/chrome` package will be reported with a project name `e2e_tests.chrome` and job name `my_tests`.
   > * For **unittest**, tests in the `my_tests.py` module in the `e2e_tests/chrome` package will be reported with a project name `chrome` and job name `my_tests`.

## Step reports

As mentioned earlier, by default, all driver commands that are executed will be reported to TestProject Cloud. In addition to this, you can also report custom steps, whether they should be marked as passed or failed, and include a screenshot of the current browser state:

```text
def test_report_a_custom_step():
    driver = webdriver.Chrome()
    driver.report().step(description='My step decription', message='A custom message', passed=True, screenshot=True)
    driver.quit()
```

## The importance of using `quit()`

Even more so than with regular Selenium- or Appium-based tests, it is important to make sure that you call the `quit()` method of your TestProject driver object at the end of every test that uses the TestProject SDK.

Upon calling `quit()`, the SDK will send all remaining report items to the Agent, ensuring that your report on the TestProject platform is complete.

**Tip for pytest users**: use a [pytest fixture](https://docs.pytest.org/en/stable/fixture.html#fixtures-as-function-arguments) to ensure that `quit()` is called at the end of the test, even when an error occurred during test execution:

```text
import pytest

@pytest.fixture
def driver():
    driver = webdriver.Chrome()
    yield driver
    driver.quit()

def test_using_pytest_fixture(driver):
    driver.get("https://example.testproject.io/web")
```

**Tip for unittest users**: use the `setUp()` and `tearDown()` [methods](https://docs.python.org/3/library/unittest.html#organizing-tests) for driver creation and destroying:

```text
import unittest

class ChromeTest(unittest.TestCase):

    def setUp(self):
        self.driver = webdriver.Chrome()

    def test_using_unittest_setup_and_teardown(self):
        driver.get("https://example.testproject.io/web")

    def tearDown(self):
        self.driver.quit()
```

## Logging

The TestProject Python SDK uses the `logging` framework built into Python. The default logging level is `INFO` and the default logging format is `%(asctime)s %(levelname)s %(message)s`, which results in log entries formatted like this:

`13:37:45 INFO Using http://localhost:8585 as the Agent URL`

If you wish, you can override the default log configuration:

* For **pytest** users, it is recommended to provide alternative values [in your pytest.ini](https://docs.pytest.org/en/latest/reference.html#ini-options-ref)
* Users of **unittest** can override the configuration by setting the `TP_LOG_LEVEL` and / or `TP_LOG_FORMAT` environment variables, respectively, to the desired values

See [this page](https://docs.python.org/3/library/logging.html#logging-levels) for a list of accepted logging levels and [look here](https://docs.python.org/3/howto/logging.html#changing-the-format-of-displayed-messages) for more information on how to define a custom logging format.

## License

The TestProject Python SDK is licensed under the LICENSE file in the root directory of the project source tree.
