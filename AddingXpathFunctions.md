## Introduction ##

XForms relies heavily on XPath for providing different processing functionality. Adding new functions to the library is achieved by 'registering' the function with the XForms processor.

## Details ##

The registration function to use is `extend()`, and the parameter to pass is an object:
```
FormsProcessor.extend({
  ...
});
```

To add an extension function that just adds two numbers together, we only need to provide an object with one property; the name of the property will be used as the name of the XPath function, and the value of the property will be the function itself:
```
FormsProcessor.extend({
  "add": function(a, b) {
    return a + b;
  };
});
```
Once registered, the function can be used within any XPath expression. For example:
```
<xf:bind nodeset="sum" calculate="add(../a, ../b)" />
```
or:
```
<xf:setvalue ref="x" calculate="add(., 5)" />
```