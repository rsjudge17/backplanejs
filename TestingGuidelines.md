## Introduction ##

There are three types of testing that are used in backplanejs. The first is unit-testing, which is used to verify that each component within the system works as expected. Since we use _test-driven development_ (TDD) then unit-tests will be the predominant form of test.

The second type of test is integration testing, where we check that one or more components work together.

And finally we have end-to-end testing, which tests that the entire framework hangs together in the correct manner. There will generally only be a few of these types of test.

### Unit-testing ###
The main environment for unit-testing is currently [YUITest](http://developer.yahoo.com/yui/yuitest/).

To create unit-tests simply add a file that begins with the prefix `ut-` and place it one of the code directories. Generally we place the files alongside the code being tested, but there are some older tests that reside in their own directory.

To illustrate, in the directory [third-party/uxf/src/lib/functions/backplane](http://code.google.com/p/backplanejs/source/browse/third-party/uxf/src/lib/functions/backplane/) we have the file [serialize.js](http://code.google.com/p/backplanejs/source/browse/third-party/uxf/src/lib/functions/backplane/serialize.js) which contains an XPath function to serialise an XML structure as text. In the same directory is [ut-serialize.js](http://code.google.com/p/backplanejs/source/browse/third-party/uxf/src/lib/functions/backplane/ut-serialize.js) which contains the associated unit-tests.

The general pattern of a unit-test file is for the test to register itself with the YUITest framework. All tests are combined into a single roll-up file using the following command from the backplanejs tools (see UsingTools):
```
ant test-compile
```
To copy the rolled-up test file to a location where the test server can see it, and to also launch the test server, use the following command:
```
ant start-test-server
```
Any sub-directory in the directory defined by `${unit-tests.dir}` is now available to the server. (This is usually `target/tests/unit-tests`.) To illustrate, if the test server had been launched on port `192.168.1.10`, then the tests would be available via the browser at the following locations:
```
http://192.168.1.10:8080/unit-tests/backplane/main.html
http://192.168.1.10:8080/unit-tests/rdfa/main.html
http://192.168.1.10:8080/unit-tests/xforms/main.html
```
This default configuration is sufficient for most scenarios, but if more directories need to be referred to from the test server then the `yaml` file will need to be updated. This can be found in the `build\test` directory, and the file is called `unit-test-app.yaml`. This file is copied to the GAE area, and in the process tokens are substituted for the directories, and the file is renamed to `app.yaml`.

All of the unit-tests can also be run in one go, in an automated way, via the command:
```
ant unit-test
```
Ant will exit if any collection of test fails.

### Integration tests ###
The main environment for integration testing is currently [Selenium](http://selenium.openqa.org/).

### End-to-end tests ###
The main environment for end-to-end testing is currently [Selenium](http://selenium.openqa.org/).