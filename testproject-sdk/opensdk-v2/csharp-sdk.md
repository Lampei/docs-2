---
description: 'C# OpenSDK'
---

# C\#

## Getting Started

To get started, you need to complete the following prerequisites checklist:

* Login to your account at [https://app.testproject.io/](https://app.testproject.io/) or [register](https://app.testproject.io/signup/) a new one.
* [Download](https://app.testproject.io/#/download) and install an Agent for your operating system or pull a container from [Docker Hub](https://hub.docker.com/r/testproject/agent).
* Run the Agent and [register](https://docs.testproject.io/getting-started/installation-and-setup#register-the-agent) it with your Account.
* Get a development token from [Integrations / SDK](https://app.testproject.io/#/integrations/sdk) page.

### Installation

 The TestProject C\# OpenSDK is [available via NuGet](https://www.nuget.org/packages/TestProject.OpenSDK/).

* Right-click the project and select `Manage NuGet Packages...`
* Search for `TestProject SDK` and add it to your project.

## Test Development

Using a TestProject driver is exactly identical to using a Selenium driver. Changing the import statement is enough in most cases.

> The following examples use the `ChromeDriver`, but they are applicable to all other supported drivers.

Here's an example of how to create a TestProject version of `ChromeDriver`:

```text
// using OpenQA.Selenium.Chrome;; <-- Replaced
using TestProject.OpenSDK.Drivers.Web;

...

public class MyTest {
  ChromeDriver driver = new ChromeDriver(chromeOptions: ChromeOptions());
}
```

## Drivers

The TestProject SDK overrides standard Selenium/Appium drivers with extended functionality. Below is the packages structure containing all supported drivers:

```text
TestProject.OpenSDK.Drivers
├── Web
│   ├── ChromeDriver
│   ├── EdgeDriver
│   ├── FirefoxDriver
│   ├── InternetExplorerDriver
│   ├── SafariDriver
│   └── RemoteWebDriver
├── Generic
│   └── GenericDriver
```

> The GenericDriver can be used to run non-UI tests and still report the results to TestProject.

### Development Token

The SDK uses a development token for communication with the Agent and the TestProject platform. Drivers search the developer token in an environment variable `TP_DEV_TOKEN`. This token can be also provided explicitly using the constructor:

```text
ChromeDriver driver = new ChromeDriver(token: "your_token_goes_here");

```

### Remote Agent

By default, drivers communicate with the local Agent listening on [http://localhost:8585](http://localhost:8585/).

The Agent URL \(host and port\) can be also provided explicitly using this constructor:

```text
ChromeDriver driver = new ChromeDriver(remoteAddress: "your_address_and_port_go_here");
```

It can also be set using the `TP_AGENT_URL` environment variable.

## Reports

TestProject SDK reports all driver commands and their results to the TestProject Cloud. Doing so allows us to present beautifully designed reports and statistics in its dashboards.

Reports can be completely disabled using this constructor:

```text
ChromeDriver driver = new ChromeDriver(disableReports: true);
```

### Implicit Project and Job Names

By default, the SDK will attempt to infer Project and Job names when you're using NUnit, MSTest or XUnit as a testing framework.

If any of these unit testing frameworks is detected, the following reporting settings will be inferred:

* The project name will be equal to the final segment of the namespace that your test class is part of. For example, if your test class is in the `TestProject.OpenSDK.Example` namespace, the project name will be equal to `Example`.
* The job name will be equal to the name of your test class.
* The test name will be equal to the name of your test method.

Examples of implicit project and job names inferred from annotations:

* [MSTest example](https://github.com/testproject-io/csharp-sdk/blob/main/TestProject.OpenSDK.Tests/Examples/Frameworks/MSTest/InferredReportTest.cs)
* [NUnit example](https://github.com/testproject-io/csharp-sdk/blob/main/TestProject.OpenSDK.Tests/Examples/Frameworks/NUnit/InferredReportTest.cs)
* [XUnit example](https://github.com/testproject-io/csharp-sdk/blob/main/TestProject.OpenSDK.Tests/Examples/Frameworks/XUnit/InferredReportTest.cs)

### Explicit Names

Project and job names can also be specified explicitly using this constructor:

```text
ChromeDriver driver = new ChromeDriver(projectName: "your_project_name", jobName: "your_job_name");
```

Examples of explicit project and job name configuration:

* [MSTest example](https://github.com/testproject-io/csharp-sdk/blob/main/TestProject.OpenSDK.Tests/Examples/Frameworks/MSTest/ExplicitReportTest.cs)
* [NUnit example](https://github.com/testproject-io/csharp-sdk/blob/main/TestProject.OpenSDK.Tests/Examples/Frameworks/NUnit/ExplicitReportTest.cs)
* [XUnit example](https://github.com/testproject-io/csharp-sdk/blob/main/TestProject.OpenSDK.Tests/Examples/Frameworks/XUnit/ExplicitReportTest.cs)

### Tests Reports

#### Automatic Tests Reporting

Tests are reported automatically when a test **ends** or when driver _quits_. This behavior can be overridden or disabled \(see the [Disabling Reports](https://github.com/testproject-io/csharp-sdk#disabling-reports) section below\).

In order to determine whether a test has ended, the call stack is inspected, searching for the current test method. When the test name is different from the latest known test name, it is concluded that the execution of the previous test has ended. This is supported for MSTest, NUnit and XUnit.

#### Manual Tests Reporting

To report tests manually, you can use the `driver.Report().Test()` method, for example:

```text
ChromeDriver driver = new ChromeDriver(new ChromeOptions());
driver.Report().Test("My First Test");
```

> It is important to disable automatic tests reporting when using the manual option to avoid any collision.

#### Steps

Steps are reported automatically when driver commands are executed. If this feature is disabled, or in addition, manual reports can be performed, for example:

```text
ChromeDriver driver = new ChromeDriver(new ChromeOptions());
driver.Report().Step("User logged in successfully");
```

### Disabling Reports

If reports were **not** disabled when the driver was created, they can be disabled or enabled later. However, if reporting was explicitly disabled when the driver was created, it can **not** be enabled later.

#### Disable all reports

This will disable all types of reports:

```text
ChromeDriver driver = new ChromeDriver(new ChromeOptions());
driver.Report().DisableReports(true);
```

#### Disable automatic test reports

This will disable automatic test reporting. All steps will end up in a single test report, unless tests are reported manually using `driver.Report().Test()`:

```text
ChromeDriver driver = new ChromeDriver(new ChromeOptions());
driver.Report().DisableAutoTestReports(true);
```

#### Disable driver command reports

This will disable driver _command_ reporting. The resulting report will have no steps, unless reported manually using `driver.Report().Step()`:

```text
ChromeDriver driver = new ChromeDriver(new ChromeOptions());
driver.Report().DisableCommandReports(true);
```

#### Disable command redaction

When reporting driver commands, the SDK performs redaction of sensitive data \(values\) sent to secured elements. If the element is one of the following:

* Any element with `type` attribute set to `password`
* With XCUITest, on iOS an element type of `XCUIElementTypeSecureTextField`

the values sent to these elements will be converted to three asterisks - `***`. This behavior can be disabled as follows:

```text
ChromeDriver driver = new ChromeDriver(new ChromeOptions());
driver.Report().DisableRedaction(true);
```

## SpecFlow support

The SDK also supports automatic reporting of SpecFlow features, scenarios and steps through the [TestProject OpenSDK SpecFlow plugin](https://www.nuget.org/packages/TestProject.OpenSDK.SpecFlowPlugin/).

After installing the plugin package using NuGet, SpecFlow-based scenarios that use an SDK driver will be automatically reported to TestProject Cloud.

When the plugin detects that SpecFlow is used, it will disable the reporting of driver command and automatic reporting of tests.

Instead, it will report:

* A separate job for every feature file
* A test for every scenario in a feature file
* All steps in a scenario as steps in the corresponding test

Steps are automatically marked as passed or failed, and Scenario Outlines are supported to create comprehensive living documentation from your specifications on TestProject Cloud.

A working example project can be found [here](https://github.com/testproject-io/csharp-sdk/tree/main/TestProject.OpenSDK.SpecFlowExamples).

## Examples

More usage examples for the SDK can be found [here](https://github.com/testproject-io/csharp-sdk/tree/main/TestProject.OpenSDK.Tests/Examples):

* Drivers
  * [Chrome Test](https://github.com/testproject-io/csharp-sdk/blob/main/TestProject.OpenSDK.Tests/Examples/Drivers/ChromeDriverTest.cs)
  * [Edge Test](https://github.com/testproject-io/csharp-sdk/blob/main/TestProject.OpenSDK.Tests/Examples/Drivers/EdgeDriverTest.cs)
  * [Firefox Test](https://github.com/testproject-io/csharp-sdk/blob/main/TestProject.OpenSDK.Tests/Examples/Drivers/FirefoxDriverTest.cs)
  * [Internet Explorer Test](https://github.com/testproject-io/csharp-sdk/blob/main/TestProject.OpenSDK.Tests/Examples/Drivers/InternetExplorerDriverTest.cs)
  * [Safari Test](https://github.com/testproject-io/csharp-sdk/blob/main/TestProject.OpenSDK.Tests/Examples/Drivers/SafariDriverTest.cs)
  * [Remote Web Driver Test](https://github.com/testproject-io/csharp-sdk/blob/main/TestProject.OpenSDK.Tests/Examples/Drivers/RemoteWebDriverTest.cs)
  * [Generic Driver Test](https://github.com/testproject-io/csharp-sdk/blob/main/TestProject.OpenSDK.Tests/Examples/Drivers/GenericDriverTest.cs)
* Frameworks
  * MSTest
    * [Inferred Report](https://github.com/testproject-io/csharp-sdk/blob/main/TestProject.OpenSDK.Tests/Examples/Frameworks/MSTest/InferredReportTest.cs)
    * [Explicit Report](https://github.com/testproject-io/csharp-sdk/blob/main/TestProject.OpenSDK.Tests/Examples/Frameworks/MSTest/ExplicitReportTest.cs)
  * NUnit
    * [Inferred Report](https://github.com/testproject-io/csharp-sdk/blob/main/TestProject.OpenSDK.Tests/Examples/Frameworks/NUnit/InferredReportTest.cs)
    * [Explicit Report](https://github.com/testproject-io/csharp-sdk/blob/main/TestProject.OpenSDK.Tests/Examples/Frameworks/NUnit/ExplicitReportTest.cs)
  * XUnit
    * [Inferred Report](https://github.com/testproject-io/csharp-sdk/blob/main/TestProject.OpenSDK.Tests/Examples/Frameworks/XUnit/InferredReportTest.cs)
    * [Explicit Report](https://github.com/testproject-io/csharp-sdk/blob/main/TestProject.OpenSDK.Tests/Examples/Frameworks/XUnit/ExplicitReportTest.cs)

## License

The TestProject SDK For C\# is licensed under the LICENSE file in the root directory of this source tree.

