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

###Mocking

###Data Tables



##Geb
