{
  title: "Jenkins",
  description: "Run automated tests on Sauce Labs using Jenkins",
  category: "CI Integrations",
  index: 0,
  image: "/images/ci-integrations/jenkins.png",
}

## Using the Sauce Jenkins Plugin

[Jenkins](https://jenkins-ci.org/) is the most popular CI tool used by our customers. To make integrating with Jenkins even easier, we developed the Sauce Jenkins Plugin. It is __not necessary__ to use the Sauce Jenkins Plugin to integrate Sauce and Jenkins, however, it does do several handy things for you:

1. Provides a user interface that lets you set environment variables on the Jenkins server that can be used in your tests (e.g., platform configurations or your Sauce username and access key)
2. Automatically launches Sauce Connect
3. Handles reporting between Jenkins and Sauce

We'll explain each of these in more detail below as well as how to intsall the plugin. Much of what the plugin does relates to the setting of environment variables. 

__Note:__ *We recommend making sure your tests run on Sauce Labs without Jenkins first before attempting to use the Jenkins Plugin.*

## Installing the Jenkins Plugin
The Sauce OnDemand plugin for Jenkins can be installed through the Jenkins Administration page.

To access this page, click __Manage Jenkins__ from the left-hand navigation pane.

![](https://docs.saucelabs.com/images/ci-integrations/jenkins/manage-jenkins.022e1a4a.png)

Then click on the __Manage Plugins__ link:

![](https://docs.saucelabs.com/images/ci-integrations/jenkins/manage-plugins.b280cf1a.png)

Select the __Available__ tab:

![](https://docs.saucelabs.com/images/ci-integrations/jenkins/available-tab.9c7e9b06.png)

Scroll down the list of plugins to find the __Sauce OnDemand plugin__, select the check box and click the __Download and install after restart__ button:

![](https://docs.saucelabs.com/images/ci-integrations/jenkins/install-plugin.88ce8d73.png)

This will download the Sauce OnDemand plugin for Jenkins. The plugin file is quite large, so the download may take a few minutes to complete. Select __Restart Jenkins when installation is complete and no jobs are running__ and when the download has finished, the Jenkins instance will restart.

![](https://docs.saucelabs.com/images/ci-integrations/jenkins/restart-jenkins.73fd02e6.png)

## Configuring Your Sauce Credentials 
#### Username and Access Key
The first thing the plugin does is provide an interface for storing your Sauce authentication credentials (username and API access key) as environment variables on the Jenkins server. This allows you to reference these in your tests. 

To configure the authentication, first click __Manage Jenkins__ from the left-hand navigation pane. Click the __Configure System__ link.

Scroll down to the __Sauce Support__ section:

![](https://docs.saucelabs.com/images/ci-integrations/jenkins/sauce-admin.9927a717.png)

Here you can enter your Username and Sauce API Access Key. Both can be found on your User Settings page [here](https://saucelabs.com/beta/dashboard). The Jenkins Plugin manages authentication at a global level so you will be able to have multiple projects running with these credentials.

You can now refer to these in your tests by referencing ```SAUCE_USER_NAME``` and ```SAUCE_API_KEY``` in your tests like so: 

```
WebDriver driver = new RemoteWebDriver(
            new URL("http://System.getenv("SAUCE_USER_NAME"):System.getenv("SAUCE_API_KEY")@ondemand.saucelabs.com:80/wd/hub"),
                desiredCapabilities);

```

## Running Sauce Connect
If your test application is not publicly available over the Internet you will need to run [Sauce Connect](https://docs.saucelabs.com/reference/sauce-connect/) so that Sauce can reach it. The Plugin automatically installs on your Jenkins server.

#### Running Sauce Connect on Your Projects
Installing the Sauce Jenkins Plugin automatically installs Sauce Connect on your build server. You just need to configure your specific project to run Sauce Connect. 

Go to your Project in Jenkins. Select __Configure__ on the left hand nav bar. Scroll down to the "Build Environment" section and select __Sauce OnDemand Support__. 

This will open up a section called "Sauce Labs Options". At the top you will see a checkbox titled "Enable Sauce Connect?". Simply select the "Enable Sauce Connect" checkbox. Doing so will ensure that a Sauce Connect tunnel is started whenever Jenkins starts a build for that particular project.

![enable Sauce Connect](https://docs.saucelabs.com/images/ci-integrations/jenkins/sauce-configure.37b02158.png)

#### Advanced Options for Sauce Connect
Immediately below the __Sauce Labs Options__ section you will see a section called __Sauce Connect Advanced Options__. Click ```Advanced...``` to reveal extra fields in which you can input advanced options. 

Here you can input the [Sauce Connect command line options](https://docs.saucelabs.com/reference/sauce-connect/#command-line-options) that you would like applied to all Jenkins projects. 

Alternatively, you can configure globally available advanced options for Sauce Connect that will be available on all projects. 

Immediately below the fields where you input your Sauce Username and API Access Key is the ```Sauce Connect Options``` field. Here you can input the [Sauce Connect command line options](https://docs.saucelabs.com/reference/sauce-connect/#command-line-options) that you would like applied to all Jenkins projects. 

![Jenkins Config](https://docs.saucelabs.com/images/ci-integrations/jenkins/sauce-admin.9927a717.png)

Another advanced configuration is the Launch Sauce Connect on Slave - when selected, the Sauce Connect process will be launched on the Jenkins slave node which is executing the build. If not selected, then Sauce Connect will be launched on the Jenkins master node

#### Changing the Default Location of Sauce Connect
The Sauce Jenkins plugin comes bundled with the latest version of Sauce Connect, and when a Jenkins build is run with Sauce Connect enabled, the default behaviour of the plugin is to extract the Sauce Connect binary which is applicable for your operating system to your home directory.

You can change the location where the plugin extracts Sauce Connect by specifying a directory location in the __Sauce Connect Working Directory__ field. This can be done on a per-project basis under __Sauce Connect Advanced Options__ or for all projects under __Sauce Support__ in your Jenkins configuration.

## Setting Environment Variables
The next feature that the Sauce Jenkins Plugin provides is a way to set [environment variables]() on the Jenkins server which contain details that you want to reference in your tests.  This makes it easy for you to change the browsers and operating systems that your tests run against without requiring you to change your test code each time.

__Note:__ Good testing practice suggests you reference environment variables to access desired capabilities rather than hardcoding them into your tests. For more on that best practice, see this article on [Using Environment Variables in Your Tests](). 

#### Platform Environment Variables 
Just below the check box where we enabled Sauce Connect you will see some options to select browser/OS combinations. Selecting platforms here creates environment variables on the Jenkins server that your tests can reference. 

If a single platform is selected, then the ```SELENIUM_PLATFORM```, ```SELENIUM_VERSION```, and ```SELENIUM_BROWSER``` environment variables will be populated to contain the details of the selected browser. 

If you select multiple (by holding command or ctrl while selecting) platforms, the __SAUCE_ONDEMAND_BROWSERS__ environment variable will be populated with a JSON-formatted string containing the attributes of the selected browsers. An example of the JSON string is:

```
[
    {
    "platform":"LINUX",
    "os":"Linux",
    "browser":"firefox",
    "url":"sauce-ondemand:?os=Linux&browser=firefox&browser-version=16",
    "browserVersion":"16"
    },
    {
    "platform":"VISTA",
    "os":"Windows 2008",
    "browser":"iexploreproxy",
    "url":"sauce-ondemand:?os=Windows 2008&browser=iexploreproxy&browser-version=9",
    "browserVersion":"9"
    }
]

```
Your tests can then reference these environment variables. For example it might look like this in psuedo-code:

```
desiredCapabilities.setBrowserName(System.getenv("SELENIUM_BROWSER"));
desiredCapabilities.setVersion(System.getenv("SELENIUM_VERSION"));
desiredCapabilities.setCapability(CapabilityType.PLATFORM, System.getenv("SELENIUM_PLATFORM"));
```

When the __Use latest versions of browser__ checkbox is selected, the Sauce plugin will populate the environment variables with the version information for the latest available version that corresponds to the selected browser and operating system.

The following environment variables are set by this browser picker: 

- ```SELENIUM_PLATFORM``` - The operating system of the selected browser
- ```SELENIUM_VERSION``` - The version number of the selected browser
- ```SELENIUM_BROWSER``` - The browser name of the selected browser.
- ```SELENIUM_DEVICE``` - The device name of the selected browser (only available for mobile browsers)
- ```SELENIUM_DEVICE_TYPE``` - The device type of the selected browser (only available for Appium browsers)
- ```SAUCE_ONDEMAND_BROWSERS``` - A JSON-formatted string representing the selected browsers


#### Other Environment Variables
The plugin also sets a series of other environment variables that you can reference in your tests. 

##### Sauce Connect Options

- ```SELENIUM_HOST``` - The hostname of the Selenium server
- ```SELENIUM_PORT``` - The port of the Selenium server

If the ```Enable Sauce Connect?``` checkbox is selected, then the SELENIUM_HOST and SELENIUM_PORT variables will default to localhost:4445. If the checkbox is not set, then the SELENIUM_HOST and SELENIUM_PORT variables will be set to ondemand.saucelabs.com:4444.

Typically you do not need to set these variables and can simply use the defaults provided by the plugin.

The values for the SELENIUM_HOST and SELENIUM_PORT environment variables can be overridden by explicitly specifying the host and port in the Sauce Host and Sauce Port fields, which can be displayed by clicking on the Sauce Connect Advanced Options button.

##### Other Variables
- ```SELENIUM_DRIVER``` - Contains the operating system, version and browser name of the selected browser, in a format designed for use by the Selenium Client Factory
- ```SAUCE_USER_NAME``` - The user name used to invoke Sauce OnDemand
- ```SAUCE_API_KEY``` - The access key for the user used to invoke Sauce OnDemand
- ```SELENIUM_ENVIRONMENT_VARIABLE_PREFIX``` - If a value is supplied here, then all the environment variables set by the Sauce plugin will be prefixed with the value

By default, the plugin will use the authentication credentials that we specified in the Sauce Credentials section at the beginning of this tutorial. However, this can be overriden in a test by enabling the __Override default authentication__ checkbox and specifying values in the Username and Access key fields. 

## Making use of multiple browsers    
 
One of the great features of running your Selenium tests with Sauce Labs is that you can easily update your tests to run against multiple browsers in parallel.  The Jenkins plugin provides a few different ways you can achieve this.

### Parallel builds   

Typically running tests in parallel requires you to change your build and test scripts accordingly.  The steps involved in achieving this largely depend on the testing framework that you're using.

### Matrix Projects

Another way to run builds in parallel is to utilize the integration between the Sauce plugin and Jenkins matrix projects.  The Sauce plugin allows you to setup a matrix project using the browser/version/operating system as an axis, so a Jenkins build is invoked for each selected browser combination.

You will first need to create a Jenkins project that is setup for matrix support.  When creating a new project, select the `Multi-configuration project` radio button.

![multi config new project](https://raw.githubusercontent.com/rossrowe/docs/master/images/ci-integrations/jenkins/multi-config-new-project.png)

Then, select the `Add Axis` drop down and select `Sauce WebDriver Tests`.

![multi config config 1](https://raw.githubusercontent.com/rossrowe/docs/master/images/ci-integrations/jenkins/multi-config-config-1.png)

You can then select the browser combinations that you want to be used for your build.

![multi config config 2](https://raw.githubusercontent.com/rossrowe/docs/master/images/ci-integrations/jenkins/multi-config-config-2.png)

When Jenkins runs a build, it will launch a separate build process for each selected browser.  Each build that is invoked will be populated with the Sauce environment variables that relate to the specific browser combination selected.

### Parameterized builds

The Sauce plugin also allows you to configure your build so that you select the browsers at the build run time, rather than in the build configuration.  

This is useful, say, if you have a Jenkins project that is configured to run regression tests, and you want to quickly determine if your website works against a new browser combination.  

To configure your Jenkins build to use parameterized builds, you will first need to select the `This build is parameterized` checkbox within the configuration of your Jenkins job.

![parameterized content 1](https://raw.githubusercontent.com/rossrowe/docs/master/images/ci-integrations/jenkins/parameterized-config-1.png)

Then select the `Add Parameter` drop down and select `Sauce Labs Browsers`.

![parameterized content 2](https://raw.githubusercontent.com/rossrowe/docs/master/images/ci-integrations/jenkins/parameterized-config-2.png)

Now you will see a `Build with parameters` option instead of a `Build` option. 

![parameterized build](https://raw.githubusercontent.com/rossrowe/docs/master/images/ci-integrations/jenkins/parameterized-build.png)

When you click `Build with parameters`, you will be presented with a list of the supported Sauce browsers.  Select the browsers you want the test to use, and the Sauce plugin will populate the environment variables using the selected browsers.

![parameterized browsers](https://raw.githubusercontent.com/rossrowe/docs/master/images/ci-integrations/jenkins/parameterized-browsers.png)


## Reporting Between Sauce and Jenkins
The Jenkins plugin automatically handles reporting between Sauce and Jenkins. All you have to do is set the 'build' capability to the value of the `JENKINS_BUILD_NUMBER` environment variable. For example in psuedo-code:

```
desiredCapabilities.setBrowserName(System.getenv("JENKINS_BUILD_NUMBER"));
```

This will ensure that the Jenkins build number is stored when the job is first run.

Setting this information in your test will:

1. Provide you an easy way to filter and identify tests run for specific builds within your [Sauce dashboard](https://saucelabs.com/beta/dashboard).
2. Allows the plugin to present links to the Sauce jobs that were run in the context of the Jenkins project, allowing you to easily view the job information and navigate to their details.

For instance, the plugin will present links to the Sauce jobs executed as part of the build on the Jenkins build details page.

![build results](https://docs.saucelabs.com/images/ci-integrations/jenkins/sauce-summary-links.53abba2c.png)

Clicking on these links will display the job details, where you can view the commands executed, play the screencast or view the screenshots of the test.

![build result](https://docs.saucelabs.com/images/ci-integrations/jenkins/sauce-report.d8ff5f97.png)

##Marking Tests Pass/Fail
The Sauce Jenkins Plugin also supports marking the Sauce jobs as passed or failed.  

This feature requires the Jenkins build to be configured to parse test result files. To do this first got to __Post-build Actions__ at the bottom of the configuration page and click __Add a post-build action__. Select ```Publish JUnit test result report```.

Next, add a second __post-build action__ and select ```Run Sauce Labs Test Publisher```. 

Once this is done, the plugin performs the following logic when attempting to match Sauce jobs with the test results:

`if the Sauce job name equals the full name of the test`
`or if the Sauce job name contains the test name`
`or if the full name of the test contains the Sauce job name (matching whole words only)`
`then we have a match`                  

If the plugin finds a test result which matches the Sauce job, then it will update the pass/fail status of the Sauce job based on the test result which will show up in your Sauce [dashboard](https://saucelabs.com/beta/dashboard).
