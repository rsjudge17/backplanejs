Deploying as few files as possible to your web server dramatically speeds up performance, and so improves the experience for your users. This section shows how to create roll-up files for one or more of the modules in backplanejs.

## Tools ##

The tool used to create modules is [Sprockets](http://getsprockets.org/), a Ruby library that preprocesses and concatenates JS source files. The site contains full instructions on how to obtain and install the library.

The main reason for using Sprockets is that each file can contain a reference to the files it depends upon, rather than having to have a central list of dependencies, as with the [YUI loader](http://developer.yahoo.com/yui/yuitest/).

The Sprockets tool is invoked from Ant, details of which are in UsingTools.

## Source and build layout ##

Modules are defined in the `build` directory, and each one has a Sprockets file that indicates the source files to be loaded. This determines the files that will be concatenated.

Each source file can in turn contain further Sprockets `require` instructions, listing any dependencies that it has. This will determine the order in which the files are concatenated.

When the concatenation process is run, two files are created -- one JS and one CSS. These files are copied to the `deploy` directory, along with any assets that will be needed at run-time.

## A module loader file ##

Module files are located in the `build` directory. An example is [build\rdfa.js](http://code.google.com/p/backplanejs/source/browse/build/rdfa.js):

```
/*
 * Copyright (C) 2009 Backplane Ltd.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *  http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

// Backplane
//
//= require "core"
//

// RDFa
//
//= require <rdfa/RDFParser>
//= require <rdfa/RDFGraph>
//= require <rdfa/RDFStore>
//= require <rdfa/RDFQuery>
//= require <rdfa/fresnel>
//= require <kb/kb>
//= require <rdfa/metascan>
```

In Sprockets, any comments of the form `/* ... */` will be passed through to the output file, but any comments that begin `//` will not. This allows us to control the comments that will appear in the final scripts.

Note also that the paths are relative to the file itself, when using quotes, and relative to a predefined list of 'load paths', when using angle brackets. Since both the [core module](http://code.google.com/p/backplanejs/source/browse/build/core.js) and the `rdfa` module are in the `build` directory, quotes are used for the first reference. However, all of the references that follow in the module file, use the angle bracket syntax, because the location of source files may change.

### Specifying paths in require directives ###

To illustrate, the file [file.js](http://code.google.com/p/backplanejs/source/browse/third-party/uxf/src/lib/backplane/io/file.js), which is in the `io` directory, relies on the [uri.js](http://code.google.com/p/backplanejs/source/browse/third-party/uxf/src/lib/backplane/uri/uri.js) script, which is in the `uri` directory. This dependency is indicated in `file.js` as follows:

```
//= require <uri/uri>
```

The alternative would be to simply make the path relative to the current file:

```
//= require "../uri/uri"
```

but this would hard-code the relationships between the files in terms of the directories.

## Source files ##

_Note: Source files are currently located in various directories, but this will change._

The source files referred to in module files can optionally contain further `require` statements, which are then used to create the dependencies.

For example, in the backplane module, the [tokmap.js](http://code.google.com/p/backplanejs/source/browse/third-party/uxf/src/lib/backplane/core/tokmap.js) script makes use of the objects in the [array.js](http://code.google.com/p/backplanejs/source/browse/third-party/uxf/src/lib/backplane/core/array.js) script, so a `require` statement is be added:

```
//= require "array"
//
function mappings() {
	this._list = new ubArray();
}
.
.
.
```

This ensures that Sprockets will place `array.js` before `tokmap.js` in the resulting file. It also automatically causes `array.js` to be loaded, even if it's not referenced in the top-level loader list. This allows us to reduce the number of top-level references, if the functionality in a script is only ever used by some other script.

## Referring to assets ##

The Sprockets tool is also capable of copying assets to a directory, during the concatenation process. This means that images and CSS files can be kept with their associated source code during the development cycle, but then copied to a single directory when deploying.

To make these assets available, Sprocketize supports the `provide` directive. For example:

```
//= provide "../third-party/uxf/src/assets"
```

The results in the entire contents of the UXF assets directory being copied to the `assets` directory, within the `deploy` directory.

## Creating and deploying the roll-up files ##

To create the JS and CSS files for testing, the following Ant command is run:
```
ant compile
```

The result is a directory called `output` in the `target` directory, which contains the rolled-up JS file, and then a sub-directory called `assets` that contains the rolled-up CSS file, and all other assets. These files can be used directly, during the development process and also as the raw material for compressing, and ultimately packaging up for distribution.

## Testing ##

### Unit-tests ###

Using a similar approach to keeping assets 'local', each module also has its own unit-tests. These test files are also concatenated, and then copied to the corresponding sub-directory, in the `target/unit-tests` directory.

For example, all of the XForms unit tests will be concatenated together, and then copied to `target/unit-tests/xforms`.

In addition, an HTML driver file is copied to this test directory, and a JS 'test runner' file is copied to the parent directory. The HTML and JS files are located in [tools/test](http://code.google.com/p/backplanejs/source/browse/#hg/tools/test).

The tests are created and copied using the following Ant command:
```
ant test-compile
```

The unit-tests can be run as a group, by using the following Ant command:
```
ant test-ut
```

This will set a Selenium RC server running, and then use it to run each of the sets of tests. The actual tests to run are defined in the [test-list.xml](http://code.google.com/p/backplanejs/source/browse/build/test/test-list.xml) file.

To run the entire sequence of combining the source, generating the tests and running them, simply use this command:
```
ant test
```

The results of the tests will be in the `target/reports` directory.

_Note: Currently not all tests pass._

To run the unit-tests individually, you'll need a local web server. Assuming that you have followed the steps for InstallingTools then you can launch a web-server with the following Ant command:
```
ant start-local-gae
```

This launches a Python server (the Google App Engine), and sets a number of paths to various parts of your development environment (see [app.yaml](http://code.google.com/p/backplanejs/source/browse/tools/GoogleAppEngine/app.yaml) for more information on the paths). You can then run a particular module's unit-tests by navigating to the appropriate link:
```
http://localhost:8080/unit-tests/backplane/main.html
http://localhost:8080/unit-tests/rdfa/main.html
http://localhost:8080/unit-tests/xforms/main.html
```

### TDD ###

The `app.yaml` file also contains a TDD reference which can be used to point to a directory containing 'work in progress'. Set the path in `my.ant.properties`, and then restart the web-server for the change to be applied to the `app.yaml` file.

Once this change is in place you can access your development area like this:
```
http://localhost:8080/tdd/my-work.html
```

To change the host name, IP address and/or port number that the server launches on -- for example, so that you can run the server on one machine and the tests on another -- set the Ant variables `gae.host` and `gae.port` in your `my.ant.properties` file, and restart.

<a href='Hidden comment: 
== Compressing ==

The compile task will run the YUI Compressor on both the rolled-up JS file, and the rolled-up CSS file. Note that the YUI Compressor is referenced directly from the Ant tasks, so there is no need to install it.

The resulting files are placed in the same directories as the originals, and can be used as described in DeployingBackplanejs.
'></a>

## Packaging ##

To create a zip file of the compressed files and assets -- either to pass to someone for deployment, or for uploading to the Google Code downloads area -- run the following Ant command:
```
ant package
```

A zip file will be created in the `target\deploy` directory, containing all of the files from the deployment directory. The filename will contain the backplanejs version number, the platform, and the platform version number. For example:
```
backplanejs-0-6-4-yui-2.8.0.zip
```

## Deploying the package ##

To deploy the zip file to the Google Code downloads area, run the following Ant command:
```
ant deploy
```

Note that this Ant task requires a file called `my.ant.properties` to be in the same directory as the main `build.xml` file, and for it to contain your Google Code account details:

```
gc.username = john.doe@gmail.com
gc.password = A12BCdeF3
```

This file is ignored in [.hgignore](http://code.google.com/p/backplanejs/source/browse/.hgignore), so your password is safe.