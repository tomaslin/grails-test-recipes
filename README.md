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

##Remote Control

##Betamax

##Geb

####Adding a button that pauses Geb via javascript 

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

####Configuring WebDriver

See blog post: http://fbflex.wordpress.com/2013/03/18/how-to-configure-webdriver-in-grails-for-your-geb-tests/

####Presentations on Geb

* Luke Daley - The evolution of Geb - http://skillsmatter.com/podcast/home/the-evolution-of-an-open-source-projects-build-automation
* Luke Daley - Productive Grails functional testing - http://skillsmatter.com/podcast/home/productive-grails-functional-testing
* Luke Daley - Very Groovy Browser Automation - http://www.youtube.com/watch?v=T2qXCBT_QBs 
* Tomas Lin - Testing Dynamic Pages with Geb - http://skillsmatter.com/podcast/groovy-grails/talk-by-tomas-lin
* Richard Paul - Acceptance Testing via Geb - http://skillsmatter.com/podcast/agile-testing/acceptance-testing-with-geb
* Craig Atkinson - Geb Groovy Grails Functional Testing - https://github.com/sjurgemeyer/GR8ConfUS2013/blob/master/CraigAtkinson-Geb/Geb_Groovy_Grails_Functional_Testing_Gr8Conf_US_2013_slides.pdf
