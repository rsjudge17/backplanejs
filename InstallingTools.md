## Introduction ##

This page explains how to install the applications that are needed by the build tools that are described in UsingTools.

## Ant ##

All tasks are run via Ant.

### Verify Ant ###

You can see if you have Ant installed by running the following command:
```
ant -version
```
If Ant is installed correctly, then you'll see a version number. Ensure that you have the latest version, currently 1.8.0RC1.  Also, the version 1.7.1 that ships with Eclipse Galileo (v3.5.1) appears to work for this project.

### Installing Ant ###

The latest version of Apache Ant is available from the [Apache Ant](http://ant.apache.org/) site. Download the appropriate archive for the latest version, and unpack it to your machine.

Then follow the instructions in the [Apache Ant manual](http://ant.apache.org/manual/index.html).

Alternately, if you have Eclipse Galileo installed, then you can add the following to your PATH:

```
%ECLIPSE_DIR%\plugins\org.apache.ant_1.7.1.v20090120-1145\bin
```

Open a new command prompt (an existing one won't have the new entries in the environment variables) and follow the steps in the previous section -- _Verify Ant_.

## Python ##

### Verify Python ###

You can see if you have Python installed by running the following command:
```
python --version
```
If Python is installed correctly, then you'll see a version number. Ensure that you have the latest 2.x version, currently 2.6.4.

## Install Python ##

The latest versions of Python are on the [Python download page](http://www.python.org/download/). The Windows installers for version 2.x are recommended over version 3 installers. At the time of writing the latest was version 2.6.4.

Add the Python and Python scripts directories to your path. Also, on Windows add `.PY` to `PATHEXT` so that Python scripts can be executed from the command-line.

Open a new command prompt (an existing one won't have the new entries in the environment variables) and  and follow the steps in the previous section -- _Verify Python_.

## Mercurial ##

The backplanejs repository uses Mercurial.

### Verify Mercurial ###

You can see if you have Mercurial installed by running the following command:
```
hg --version
```
If Mercurial is installed correctly, then you'll see a version number. If you intend to use Mercurial for serious development then ensure that you have the latest version.

### Installing Mercurial ###

The latest version of Mercurial is available from the [Mercurial](http://mercurial.selenic.com/) site. Note that Mercurial is written in Python, so you will need to have completed the previous section, and installed Python.

Alternately, for Windows-based development with Eclipse Galileo, select 'Help|Install New Software...' and use the update site http://cbes.javaforge.com/update to obtain the Mercurial plugin for eclipse, including the windows executable.  Then, add the following to your PATH:

```
%ECLIPSE_DIR%\plugins\com.intland.hgbinary.win32_1.4.3.v201005111545\os\win32
```

Open a new command prompt (an existing one won't have the new entries in the environment variables) and follow the steps in the previous section -- _Verify Mercurial_.

## backplanejs tools ##

The backplanejs tools are available by obtaining a copy of the repository. See the [Checkout](http://code.google.com/p/backplanejs/source/checkout) page for details.

To check the backplanejs tools, change into the backplanejs directory and run the following command:
```
ant help
```
You should see a reference to the UsingTools wiki page.

## Ruby ##

To compress javascript and CSS files, backplanejs uses Sprocketize, which in turn uses Ruby.

### Verify Ruby ###

You can see if you have Ruby installed by running the following command:
```
ruby --version
```
If Ruby is installed correctly, then you'll see a version number.

### Installing Ruby ###

The Easiest way to get Ruby is from the [Ruby on Rails download page](http://rubyonrails.org/download).

Open a new command prompt (an existing one won't have the new entries in the environment variables) and follow the steps in the previous section -- _Verify Ruby_.

## Sprocketize ##

Once Ruby is installed then Sprocketize can be installed by typing:
```
gem install --remote sprockets
```

Note that there is a problem on Windows that leaves the Sprocketize batch file unusable. To fix this locate `sprocketize.bat` and remove the duplicated quote characters that appear after the reference to `ruby.exe`.

## Google App Engine ##

The Google App Engine (GAE) is used for local testing and to create a project site, which includes samples, documentation and tests.

To install the GAE, go to the [download page](http://code.google.com/appengine/downloads.html). Ensure you obtain the Python version.

During development GAE will be run from the command-line via Ant. The easiest way to set the required shortcuts and environment variables is to run the GUI version of the GAE once.

Unfortunately, the first time GAE is run from the command-line it will ask whether it should check for updates each time it is run, and then pause for user input. This interaction causes a problem when running GAE from within Ant, so you need to ensure that the first time GAE is run after it has been installed, is from the command-line, giving you the chance to acknowledge the prompt.

However, GAE also needs to have a config file in place in order to load, so before it can be run from the command-line the config file needs to be created. To do this perform the following from the command-line:
```
ant start-test-server
```
This will fail, but the required files will have been copied.

To run GAE and answer the update question run the following from the command-line:
```
dev_appserver.py target/tests
```
If you have changed the location of the tests directory then you'll need to set this path to whatever the value of `tests.dir` is. When asked, answer 'n' so that GAE doesn't check for updates, and then hit `Ctrl-C` or `Ctrl-Break` to stop the GAE server.

## Configuring `my.ant.properties` ##

There are a number of private properties that the tools will need, such as passwords or local IP addresses. These values can be set in the file `my.ant.properties` which is set to be ignored in the `.hgignore` file.

Create your own copy of `my.ant.properties` by copying `copy-to-my.ant.properties` in the root of the project. The values that you can set are:

| Property | Purpose |
|:---------|:--------|
| **gae.host** | The host name or IP address to use when running GAE. The default value of `localhost` is fine in most situations, but you may want to set this if you have other local servers running, or you want to use Parallels on a Mac. |
| **gae.port** | The port number to use when running GAE. The default is 8080. |