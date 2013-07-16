#Grails Testing: Cookbook

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

##Spock 

####Compare with previous value using old()

```groovy
when: 
  value * 30
then:
  value == old( value ) * 30
```

###Data Tables

####One column where clauses
```groovy
where:
  MyCollection | _
  'rabbit'     | _
  'cows'       | _
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

####Combinations 

Creates a data table with all 8 possible combinations: 

```groovy
where:
   [name, fingers, nombre] = [ [ 'one', 'two'], [1,2], ['uno', 'dos'] ].combinations()
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

##Remote Control

##Geb
