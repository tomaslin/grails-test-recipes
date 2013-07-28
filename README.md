#Recipes for Testing Grails

##The Command Line

To enter the command line, type 'grails' in your terminal. 

####Run all tests
```
test-app
```

####Re-run failing tests
```
test-app -rerun
```

####Run only a test based on their phase:
```
test-app unit:
test-app integration:
test-app functional:
test-app other:
```

####Run tests that match filenames:
```
test-app A*
test-app A* B* C*
```

####Run tests that match package names:
```
test-app com.mypackage.*
test-app com.mypackage.**.*
```

####Run tests based on type:
```
test-app :spock
```

####Restarting test daemon
```
restart-daemon
```

####View results of test
```
open test-report
```

####Configuring the test daemon
The test deamon in the grails shell exists in 2.3.x branches to make running of the tests faster. This is similar to the gradle daemon. 

To disable this functionality, modify the BuildConfig.groovy file and change the last attribute of this map to false

```
    test: [maxMemory: 768, minMemory: 64, debug: false, maxPerm: 256, daemon:true],
```

####Printing out which tests are running

Add the following script to scripts/_Events.groovy

```groovy
eventTestCaseStart = { name ->
    println '-' * 60
    println "|$name : started"
}

eventTestCaseEnd = { name, err, out ->
    println "\n|$name : finished"
}
```

####Sharding tests for CI servers

Add the following to scripts/TestParallel.groovy

```groovy
import org.codehaus.groovy.grails.test.*
import org.codehaus.groovy.grails.test.support.*
import org.codehaus.groovy.grails.test.event.*

includeTargets << grailsScript("_GrailsClean")
includeTargets << grailsScript("_GrailsTest")

target(main: "Runs functional tests in parallel in sets of bucketSize") {

    depends(checkVersion, configureProxy, parseArguments, cleanTestReports)

    def tests = new SpecFinder(binding).getTestClassNames()
    def id = (argsMap.set ?: 0) as int
    def sets = (argsMap.total ?: 4) as int

    List<String> shard = []

    tests.eachWithIndex { test, index ->
        if (index % sets == id) {
             shard << "${ tests.get(index) }"
        }
    }

    testNames = shard

    phasesToRun = ['functional']

    allTests()

}

setDefaultTarget(main)

class SpecFinder extends GrailsTestTypeSupport {

    SpecFinder(binding) {
        super('name', 'functional')
        buildBinding = binding
    }

    int doPrepare() {
        0
    }

    GrailsTestTypeResult doRun(GrailsTestEventPublisher eventPublisher) {
        null
    }

    def getTestClassNames() {
        findSourceFiles(new GrailsTestTargetPattern('**.*Spec')).sort { -it.length() }.collect { sourceFileToClassName(it) }
    }
}
```

To run, call grails test test-Parallel --set=2 --total=6 

where set is the shard number and total is the total number of shards.

Be aware that you need to set the environment or it will default to Development. 

##Spock 

( Mostly from: http://docs.spockframework.org/en/latest )

####Compare with previous value using old()

```groovy
when: 
  value * 30
then:
  value == old( value ) * 30
```

####Changing label formatting in IntelliJ Idea

####Generate a HTML report based on labels

Project on github - https://github.com/chanwit/spock-report-html

###Extensions

####Useful Extensions

* @Unroll - splits out features into separate tests
* @Ignore - do not run this test
* @Ignore(reason = "I am too old for this shit") - can have a reason
* @IgnoreRest - only run this test
* @IgnoreIf({ os == 'windows' }) 
* @Require({ payscale == 'jedi' }) 
* @Stepwise - causes each feature method to be executed in declaration order and any after the first failure to be ignored (i.e. story mode)
* @Shared - makes a field available to all tests / where clauses
* @Timeout - causes a method to fail if it takes to long (and will interrupt it if it does take too long)
* @AutoCleanup - allows a named cleaning up/closing method to be automatically called by Spock on a field
* @FailsWith - makes a feature pass only if it raises a certain exception and it’s uncaught

####Make your own extensions

Follow Luke Daley's guide at http://ldaley.com/post/971946675/annotation-driven-extensions-with-spock

###Data Tables

####One column where clauses
```groovy
where:
  animal   | _
  'rabbit' | _
  'cow'    | _
```

####Double Pipe to separate input from output
```groovy
where:
  english  | plural || spanish 
  'rabbit' | true   || 'conejos'
  'cow'    | false  || 'vaca'
```

####Assign variables
```groovy
where:
   a = 'cars'
```

####Data Pipes
```groovy
where:
   a << ['one', 'two', 'three' ]
```

This will run each value of the pipe at each iteration - resulting in three tests.

####Multiple assignments
```groovy
where:
   [a, b, c] << [ [ 1, 2, 3 ],
                  [ 1, 3, 5 ] ]
```

####Skip values
```groovy
where:
   [a, _, c] << [ [ 1, 2, 3 ], [ 1, 3, 5 ] ]
```

####Combinations 

Creates a data table with all 8 possible combinations: 

```groovy
where:
   [name, fingers, nombre] << [ [ 'one', 'two'], [1,2], ['uno', 'dos'] ].combinations()
```

This is equivalent to:
```groovy
where:
   name  | fingers | nombre
   'one' |    1    | 'uno'
   'two' |    1    | 'uno'
   'one' |    2    | 'uno'
   'two' |    2    | 'uno'
   'one' |    1    | 'dos'
   'two' |    1    | 'dos'
   'one' |    2    | 'dos'
   'two' |    2    | 'dos'
```

####Can combine different types of data table format
```groovy
where:
   name  | nombre
   'one' | 'uno'

   properName = name.capitalize()
   postCode << ['90210', '90222']
```

IntelliJ won't recognize mixed format so you have to do the data table, format it and then the other values.

####Presentations on Spock

* Peter Niederwieser : Spock - the smarter testing and specification framework (Devoxx)
 http://parleys.com/play/514892260364bc17fc56bdb5/about
* Rob Fletcher - Groovier Testing with Spock - http://skillsmatter.com/podcast/groovy-grails/groovier-testing-with-spock
* Luke Daley - Smarter Testing with Spock - http://skillsmatter.com/podcast/java-jee/spock

##Betamax

####Presentations on Betamax
* Rob Fletcher - Testing HTTP dependencies with Betamax  http://skillsmatter.com/podcast/groovy-grails/testing-http-dependencies-with-betamax

##Geb

####Adding a button that pauses Geb via javascript (Luke Daley)

```groovy
   private void pause() {
       js.exec """
           (function() {
               window.__gebPaused = true;
               var div = document.createElement("div");
               div.setAttribute('style', "\\
                   position: absolute; top: 0px;\\
                   z-index: 3000;\\
                   padding: 10px;\\
                   background-color: red;\\
                   border: 2px solid black;\\
                   border-radius: 2px;\\
                   text-align: center;\\
               ");

               var button = document.createElement("button");
               button.innerHTML = "Unpause Geb";
               button.onclick = function() {
                   window.__gebPaused = false;
               }
               button.setAttribute("style", "\\
                   font-weight: bold;\\
               ");

               div.appendChild(button);
               document.getElementsByTagName("body")[0].appendChild(div);
           })();
       """

       waitFor(300) { !js.__gebPaused }
   }
```

####Extend your own elements by providing your own navigators (Sky)

Provide your own class for empty and nonempty navigators
```
class NonEmptyNavigator extends geb.navigator.NonEmptyNavigator {
    NonEmptyNavigator(Browser browser, Collection<? extends WebElement> contextElements) {
        super(browser, contextElements)
    }

    boolean isDirty() {
        hasClass 'dirty'
    }

    boolean isHidden() {
        hasClass 'hidden'
    }
}
```

Add this to GebConfig

```
innerNavigatorFactory = { Browser browser, List<WebElement> elements ->
    elements ? new NonEmptyNavigator(browser, elements) : new EmptyNavigator(browser)
}
```

####Handling css transitions (Sky)

Add this to the nonempty navigator / webkit only

```
void waitForCssTransition(Closure trigger) {
        def element = firstElement()

        browser.js.exec(element, '''
            var o = jQuery(arguments[0]);
            window.setTransitionFinishedClass = function() {
                $(this).addClass('transitionFinished');
            }
            o.bind('webkitTransitionEnd', window.setTransitionFinishedClass);
        ''')

        try {
            trigger.call()
            browser.waitFor {
                hasClass('transitionFinished')
            }
        } finally {
            browser.js.exec(element, '''
                var o = jQuery(arguments[0]);
                o.removeClass('transitionFinished')
                o.unbind('webkitTransitionEnd', window.setTransitionFinishedClass);
                window.setTransitionFinishedClass = undefined;
            ''')
        }
}
```

####Automatically download chromedriver (Sky)

Into your gebConfig
```
driver = { new FirefoxDriver() }

private void downloadDriver(File file, String path) {
    if (!file.exists()) {
        def ant = new AntBuilder()
        ant.get(src: path, dest: 'driver.zip')
        ant.unzip(src: 'driver.zip', dest: file.parent)
        ant.delete(file: 'driver.zip')
        ant.chmod(file: file, perm: '700')
    }
}

environments {

    // run as “grails -Dgeb.env=chrome test-app”
    // See: http://code.google.com/p/selenium/wiki/ChromeDriver
    chrome {
        def chromeDriver = new File('test/drivers/chrome/chromedriver')
        downloadDriver(chromeDriver, "http://chromedriver.googlecode.com/files/chromedriver_mac_23.0.1240.0.zip")
        System.setProperty('webdriver.chrome.driver', chromeDriver.absolutePath)
        driver = { new ChromeDriver() }
    }

```

####Page Builder Pattern (Craig Atkinson)

Helps make tests more explicit by having helper methods always return the actual page. 

Instead of

``` 
to HomePage 
loginButton.click()
```

make it

```
Homepage homepage = to(HomePage)
homepage.loginButton.click()
```

all helper methods should return the page as the last argument

```
SignedInPage login( String username, String password ){
      // code here
      at SignedInPage
      browser.page
}
```

In your tests, you call


```
when:
  Homepage homepage = to(HomePage)
  SignedInPage signedInPage = login( 'bob', 'password' )
then:
  signedInPage.displayedUsername == 'Bob Marley'
```

####Inject a js library to the page

Add helper method to Geb base page

```
def injectLibrary( library ){
     js.exec("document.body.appendChild(document.createElement('script')).src='$library'"); 
}
```

Call in page that needs injection

```
injectLibrary( 'http://sinonjs.org/releases/sinon-1.4.2.js' )

```

####Setting up headless browsing on Jenkins with xvfb

http://www.labelmedia.co.uk/blog/setting-up-selenium-server-on-a-headless-jenkins-ci-build-machine.html

####Use Sauce Labs

add dependency to buildconfig

```
test "org.seleniumhq.selenium:selenium-remote-driver:2.31.0"
```

in GebConfig

```
import org.openqa.selenium.remote.DesiredCapabilities
import org.openqa.selenium.remote.RemoteWebDriver
driver = {
   DesiredCapabilities capabilities = DesiredCapabilities.firefox()
   capabilities.setCapability("version", "17")
   capabilities.setCapability("platform", "Windows 2012")
   new RemoteWebDriver(
     new URL("http://:<access_key>@ondemand.saucelabs.com:80/wd/hub"), capabilities
   )
}
```

####Use PhantomJS via Ghostdriver

In BuildConfig

```
test( "com.github.detro.ghostdriver:phantomjsdriver:1.0.1" ) {
   transitive = false
}
```

in GebConfig

```
import org.openqa.selenium.phantomjs.PhantomJSDriver
import org.openqa.selenium.remote.DesiredCapabilities
import org.openqa.selenium.Dimension
driver = {
    new PhantomJSDriver(new DesiredCapabilities())
}
```

####Switching to a different driver for one Spec

```
import geb.spock.GebSpec
import spock.lang.Shared
import org.openqa.selenium.firefox.FirefoxProfile
import org.openqa.selenium.firefox.FirefoxDriver
 
class MobileSpeck extends GebSpec {
 
@Shared def cachedDriver
 
def setupSpec(){
  cachedDriver = new FirefoxDriver()
}
 
def setup(){
  // assign this as the default driver on the browser for each test
  browser.driver = cachedDriver
}
 
def cleanupSpec(){
  // after running the spec, kill the driver
  cachedDriver.quit()
}
 
}
```

####Useful WebDriver settings

Fixed Screen Size

```
import org.openqa.selenium.phantomjs.PhantomJSDriver
import org.openqa.selenium.remote.DesiredCapabilities
import org.openqa.selenium.Dimension
driver = {
    def d = new PhantomJSDriver(new DesiredCapabilities())
    d.manage().window().setSize(new Dimension(1028, 768))
    d
}
```

Spoof User Agent in PhantomJS

```
def capabilities = new DesiredCapabilities()
capabilities.setCapability("phantomjs.page.settings.userAgent",
"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_8_3) AppleWebKit/537.36 (KHTML,
like Gecko) Chrome/27.0.1453.93 Safari/537.36")
new PhantomJSDriver(capabilities)
```

Spoof User Agent Firefox

```
FirefoxProfile profile = new FirefoxProfile();
profile.setPreference("general.useragent.override", 'Mozilla/5.0 (iPhone; U; CPU iPhone OS 3_0 like Mac OS X; en-us) AppleWebKit/528.18 (KHTML, like Gecko) Version/4.0 Mobile/7A341 Safari/528.1');
cachedDriver = new FirefoxDriver(profile)
```

####Presentations on Geb

* Luke Daley - The evolution of Geb - http://skillsmatter.com/podcast/home/the-evolution-of-an-open-source-projects-build-automation
* Luke Daley - Productive Grails functional testing - http://skillsmatter.com/podcast/home/productive-grails-functional-testing
* Luke Daley - Very Groovy Browser Automation - http://www.youtube.com/watch?v=T2qXCBT_QBs 
* Tomas Lin - Testing Dynamic Pages with Geb - http://skillsmatter.com/podcast/groovy-grails/talk-by-tomas-lin
* Richard Paul - Acceptance Testing via Geb - http://skillsmatter.com/podcast/agile-testing/acceptance-testing-with-geb
* Craig Atkinson - Geb Groovy Grails Functional Testing - https://github.com/sjurgemeyer/GR8ConfUS2013/blob/master/CraigAtkinson-Geb/Geb_Groovy_Grails_Functional_Testing_Gr8Conf_US_2013_slides.pdf
* Mike Ensor - Writing an Extensible Web Testing Framework ready for the Cloud - http://gr8conf.eu/Presentations/Writing-an-Extensible-Web-Test
