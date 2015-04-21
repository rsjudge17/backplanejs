## Introduction ##

The backplanejs tools are used to manage the backplanejs library, such as creating and compressing rolled-up files, deploying archives, producing documentation, running tests and so on. They have also been structured to allow them to be used with other applications.

See InstallingTools for information on how to install all of the applications referred to here.

## Ant ##

All tools for building and testing backplanejs are accessed via Ant.

Ant has a notion of named tasks which carry out some action and can be invoked by name. For example, to deploy the source archive to the Google Code Project [Downloads](http://code.google.com/p/backplanejs/downloads/list) page we would run the following command:
```
ant deploy
```

Ant tasks can depend on other tasks, so it's possible to specify that before deployment can be carried out the tests must pass, and before the tests can pass the scripts must be combined.

Tasks are located in the [build.xml](http://code.google.com/p/backplanejs/source/browse/build.xml) file, and when adding new tasks the [The Elements of Ant Style](http://wiki.apache.org/ant/TheElementsOfAntStyle) guidelines should be followed.

## Phases and goals ##

Inspired by the [Maven lifecycle](http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html) we've differentiated between tasks that are part of the project lifecycle, and tasks that might be run at any time, independent of the lifecycle.

For example, testing requires that the files have already been combined, packaging requires that the tests have already been run successfully, and deployment requires that a package has been created. These different tasks are called _phases_, and each one will run the previous one in the life-cycle.

However, we might also want to delete all of the directories so as to start again with a fresh install, or generate the documentation; tasks such as these fall outside of the normal lifecycle and can be run at any time. These tasks are called _goals_.

The tasks available are listed below.

### Phases ###

The following phases are available:
| **Phase** | **Action** |
|:----------|:-----------|
|compile|Combine the JS files into a single file, and do the same for the CSS files. Results are placed in $output.dir |
|test|Test the compiled source code using the unit-test and functional test frameworks.|
|package|Take the compiled code and package it in its distributable (deployable) format. Resulting zip file is placed in $deploy.dir |
|deploy|Copy the final package to Google Code Project [Downloads](http://code.google.com/p/backplanejs/downloads/list) page for sharing with developers.|

Note that each phase will run the previous phase. So creating a package will involve running the tests, which will in turn combine the source files.

### Goals ###

Goals can be run standalone, independent of the phases:
| **Goal** | **Action** |
|:---------|:-----------|
|initialize|Initialise the build state by setting properties and creating directories.|
|test-compile|Combine the unit-tests and functional tests, ready for testing.|
|site|Generate the project's web-site, including documentation.|
|site-deploy|Deploy the web-site to Google App Engine.|

Note that goals are used to implement the phases. So the `test` phase will not only call the `compile` phase, but it will also use the `test-compile` goal.

## Helper tasks ##

We also have a collection of useful tasks:

| **Task** | **Action** |
|:---------|:-----------|
|start-test-server|Start a local server which will be used for testing. The root of the server points to the test directory (see the _Directories_ section below). The test server is Google App Engine, so ensure this has been installed using the instructions on the InstallingTools page.|
|start-selenium-rc|Start Selenium RC ready to accept test instructions.|
|unit-test|Run the unit-tests.|
|functional-test|Run the functional tests.|
|regression-test|Run the regression tests (functional tests that are linked to an issue in the issue-tracker).|

## Testing ##

Tests can be run in a number of ways, depending on the degree of control you want to retain, and how frequently you will be running the tests.

### Testing everything in one step ###

`ant test` will compile all of the source and tests, launch the necessary test servers, run the unit-tests, the functional tests and the regression tests, and then close the servers. This is mainly used by continuous integration systems, but it is also useful if you have applied a patch from someone else and want to test it before continuing, or are about to submit a patch.

### Unit-testing for TDD ###

When doing test-driven development (TDD) you will most likely be using unit-testing most of the time. There are a number of ways that this can be done, but the two main ones are:

  * use `ant start-test-server` to start the test server in its own command window. This can be left running for the duration of your development session, and the unit-tests can then be run by navigating to `http://localhost:8080/unit-tests/{module}/main.html` (where `{module}` is one of `xforms`, `rdfa`, `kb` or `backplane`);
  * alternatively, after starting the test server, use `ant start-selenium-rc` in another command window. This can also be left running for the duration of your development session. Now you can run _all_ unit-tests for all of the modules by typing `ant unit-test`.

Note that if any or both of the servers aren't running, then `ant unit-test` will start them for the duration of the tests, and then terminate them again. However, if they are already running it won't terminate them.

### Functional testing ###

Functional testing checks larger components of the library and how they work together. As with unit-testing you need to start the test server in its own command window with `ant start-test-server`. It can then be left running for the duration of your development session.

The functional tests can be run with the command `ant functional-test`.

Note that if the test server isn't running then `ant functional-test` will start it for the duration of the tests, and then terminate it again. However, if the server is already running it won't be terminated.

### Regression testing ###

There is a special class of functional tests called _regression tests_ which relate to issues that have been entered into the tracking system. These tests are created to prove that a bug has been fixed, but are re-run to ensure that the bug doesn't reappear.

As with the other types of testing you need to start the test server in its own command window with `ant start-test-server`. It can then be left running for the duration of your development session.

The regression tests can then be run with the command `ant regression-test`.

Note that if the test server isn't running then `ant regression-test` will start it for the duration of the tests, and then terminate it again. However, if the server is already running it won't be terminated.

## Directories ##

The various directories used by the tools are defined using Ant variables, which makes them very easy to override. The directories are also defined relative to each other, so that if any of the root directories are redefined, all sub-directories will automatically change.

For example, the target directory (the root of all output) is defined as _target_, whilst the temporary directory is defined as _${target.dir}/tmp_. This means that if _${target.dir}_ is redefined as being _my-target_, then the temp directory will automatically become _my-target/tmp_.

Note also that sub-directories can be moved independently of their parents; if the temp directory is redefined as _c:/tmp_ that will have no effect on the target directory.

The default paths are as follows:

| **Name** | **Default location** | **Description** |
|:---------|:---------------------|:----------------|
| build.dir | build | Root directory of all files needed for a build. |
| src.dir | ${build.dir}/src | Root directory of all source files, including configuration files that might be needed by other other tools. |
| target.dir | target | Root directory for all output, including files to be deployed, reports, temporary files, and so on. |
| output.dir | ${target.dir}/output | Output directory for rolled-up files and assets. |
| reports.dir | ${target.dir}/report | Output directory for any reports, such as those created by automated testing tools. |
| deploy.dir | ${target.dir}/deploy | Output directory for any files that are ready for deployment. |
| site.dir | ${target.dir}/site | Output directory for the site that accompanies the project. |
| docs.dir | ${site.dir}/docs | Output directory within this site, which will hold the documentation. |
| tests.dir | ${target.dir}/tests | Directory for all tests that will be accessed via the test server. |
| unit-tests.dir | ${tests.dir}/unit-tests | Directory for the unit-tests that will be accessed via the test server. |
| regression-tests.dir | ${tests.dir}/regression-tests | Directory for the regression tests that will be accessed via the test server. |
| tmp.dir | ${target.dir}/tmp | General purpose temporary directory. |