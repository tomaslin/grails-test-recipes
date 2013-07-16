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

##Geb

###Configuring WebDriver

See blog post: http://fbflex.wordpress.com/2013/03/18/how-to-configure-webdriver-in-grails-for-your-geb-tests/
