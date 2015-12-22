jsmake is a basic javascript preprocessor/build utility that you can easily integrate in your continuous integration environment.


  * [What's new](#What's_new.md)
  * [What's inside](#What's_inside.md)
  * [Directives](#Directives.md)
    * [!require](#!require.md)
    * [@define](#@define.md)
    * [@ifdef @end](#@ifdef_@end.md)
    * [@ifndef @end](#@ifndef_@end.md)
  * [Command line options](#Command_line_options.md)
    * [input](#input.md)
    * [output](#output.md)
    * [compression](#compression.md)
    * [packages](#packages.md)
    * [vars](#vars.md)
      * [Reserved variable](#Reserved_variable.md)
        * [TRUE\_ANON](#TRUE_ANON.md)
        * [NAME\_ANON](#NAME_ANON.md)
    * [define](#define.md)
    * [config](#config.md)
    * [help](#help.md)

# What's new #

**1.8**

- bug fix

> define directive in config.jsm was not parsed properly

> fix windows cygwin environment crlf handling

- enhancement

> output has been enhanced (file size is human readable,define variable is displayed)

> jsmake resolves input/output path settings relative to config.jsm folder


**1.7**

- bug fix regarding the -config option and little enhancement of the output.

**1.6**

- jsmake now supports [configuration file](#config.md) and -config option


# What's inside #

jsmake can:

- merge several javascript files into one

- load javascript files by dependency (dependencies of dependencies are loaded in order)

- compress the final output with different flavor of compression algorithm

- handle some preprocessor directives (ifdef,ifndef,variables,clean up).

# Directives #

all the directives must be written within javascript ` /* */ ` comments, on one line.
they can be prefixed by # or @, except for the require Directive which uses !.

## !require ##
```
/*!require: [javascript package names separated by a comma] */
```
example:
```
/*!require: Base,DOM,Plugins.Effects */
```
The !require: command is like a shebang command and should be written at the very first line of each javascript files that you intend to merge.

### About packages ###

the require directives assume that your javascript files are following the same packaging convention as actionscript or perl, which means that:
```
MyProject.DOM.Util
```
will be understood as the following folder structure:

MyProject/DOM/Util.js


Let's imagine a virtual project with the following packages and dependencies:

Core

Core.DOM             depends on Core,Core.Util.Events

Core.Util.Array      depends on Core

Core.Util.String     depends on Core

Core.Util.Queue      depends on Core,Core.Util.Array

Core.Util.Events     depends on Core,Core.Util.Queue

this represents the following folder tree structure:

Core.js

Core/DOM.js

Core/Util/Array.js

Core/Util/String.js

Core/Util/Queue.js

Core/Util/Events.js

If you want all of them to be merged, you need to add the shebang command !require:

Core.js                `/*!require: Core */`

Core/DOM.js            `/*!require: Core.Util.Events */`

Core/Util/Array.js     `/*!require: Core */`

Core/Util/String.js    `/*!require: Core */`

Core/Util/Queue.js     `/*!require: Core.Util.Array */`

Core/Util/Events.js    `/*!require: Core.Util.Queue,Core */`

you could write Core,Core.Util.Events. It's ok.

The order of dependency does not need to be respected in your definition
and writing interdependent packages several times should just be fine (ie, like Core/Util/Events.js).

If you need a package to be merged that do not depends on any other files,
just write the file name with no ".js" like Core.js.

So now, you want to create a built that uses the functionality in Core/DOM.js.

jsmake will therefore merge into one file all files below in this exact order:

Core.js

Core/Util/Array.js

Core/Util/Queue.js

Core/Util/Events.js

Core/DOM.js

As you can see Core/Util/String.js  being unnecessary is not in the list.

Easy as pie!

## @define ##
## #define ##
```
/*@define VARIABLE_NAME VALUE */
```
one space between each elements. no more, no less.

example:
```
/*@define VERSION 4.06 */

/*@define AUTHOR  hithere */
```
How to use these variables then?
```
var version = /*@=VERSION */
```
or
```
var version = /*#=VERSION */
```
once processed it will be output as:
```
var version = 4.06
```
notice, no space between = and the variable name.

Version is not VERSION.


## @ifdef @end ##

## #ifdef #end ##
```
/*@ifdef VARIABLE_NAME */

//code goes here

/*@end */
```
This directive works only with variables define on the command line utility and does not work with variables defined with the define variable.
Will add this later I think...

Example:
```
/*@ifdef DEBUG_MODE */

console.log("development server is in debug mode!");

/*@end */
```

if you defined the DEBUG\_MODE variable on the command line, this code will be output.

If it is not defined, the code will be stripped down.

## @ifndef @end ##

## #ifndef #end ##
```
/*@ifndef VARIABLE_NAME */

//code goes here

/*@end */
```
The exact opposite of ifdef.

# Command line options #

so now, how to use it??

just call from the command line the perl script:
```
$ jsmake
```
It should be working with as few things as the above.

jsmake will look for the current directory and merged/preprocess everything it cans into a file called output.js.

There are some options of course. Here we go:

## input ##
```
$ jsmake -input ../lib
```
This tells where to look for the javascript to be merged/preprocessed/smashed.

## output ##
```
$ jsmake -output ../src/merged.js
```
This tells where to output the final merged output.

## compression ##
```
$ jsmake -compression [compression name]
```
Example:
```
$ jsmake -compression mini
```
This tells what kind of compression it should use.

There are 3 different compressors for now:

  * packer
  * mini
  * closure

Closure use google online closure service with the least aggressive algorithm.

So... you need to be connected to the internet to use this one.

packer,mini,closure requires perl modules to be installed, respectively JavaScript::Packer
,JavaScript::Minifier and Javascript::Closure.

to install scripts on linux/mac, on the command line write:

$ perl -MCPAN -e shell

(if this is the first time you run cpan command line utility,

follow the setup and choose the defaults and your nearest mirror)

$ cpan > install Javascript::Closure

... fetch the package and install


$ cpan > install JavaScript::Mini

... fetch the package and install


$ cpan > install JavaScript::Packer

... fetch the package and install


Using the compression will output 2 files:

the one uncompressed and the one compressed prefixed with the name of the compression type, so it might be output.js and closure-output.js

## packages ##
```
$ jsmake -packages [packages name separated by a space]
```
Example:
```
$ jsmake -packages Core.DOM Core.Util.String
```
If you do not specify a package jsmake will merge all the files that contain the !require: directive.

By specifying the package name (which will be understood as Core/DOM.js),jsmake will only merge the file necessary for this package to work as defined in your dependencies.

As in the above example, Core.DOM does not need Core.Util.String so we add it here.

Well, in that case, it will merge all the files as they are all interdependent.

## vars ##
```
$ jsmake -vars [variable name separated by a space]
```
Example:
```
$ jsmake -vars DEBUG STRICT_MODE
```
By specifying the name of some variables, you will make them come true.

Therefore all the ifdef or ifndef will be stripped down accordingly.

if you want a STRICT\_MODE but no DEBUG messages, you will write:
```
$ jsmake -vars STRICT_MODE
```

### Reserved variable ###

#### TRUE\_ANON ####

Example:
```
$ jsmake -vars TRUE_ANON STRICT_MODE
```

If you set the variable TRUE\_ANON,the preprocessor will look for all the functions

starting with "` $_ `".

This was especially thought in order to erase the "name" of anonymous functions

that you might set in order to get a readable stack trace for debugging.

Example:
```
var myObject = {
    doSomething:function () {
       console.loggg();
   }
};

document.body.onclick=function(){
    myObject.doSomething();
}
```

you will get the following stack trace when you click:
```
onclick()
(?)()
```

not helpful...
If you name your anonymous function:

```
var myObject = {
    doSomething:function $_doSomething() {
       console.loggg();
   }
};

document.body.onclick=function $_bodyClick(){
    myObject.doSomething();
}
```

you will get the following stack trace when you click:
```
$_doSomething()
$_bodyClick()
```

which is way much helpful!

But this code should be removed when you are done with debugging your application.

By setting TRUE\_ANON, the preprocessor will clean up all the functions starting by ` $_ `.

#### NAME\_ANON ####

Example:
```
$ jsmake -vars NAME_ANON STRICT_MODE
```

If you set the variable NAME\_ANON,the preprocessor will look for all the anonymous functions and name them by prefixing them with "` $_ `".

This was especially thought for large existing codes or lazy persons (who said me?) who do not want to name all their anonymous functions for sole debugging purpose for then seeing them erased...

Example:
```
var testing = function () {
    return true;
}
var myObject = {
    doSomething:function () {
       console.log();
   }
};
document.body.onclick=function(){
    myObject.doSomething();
}
document.body.addEventListener('click',function(e){
    var object= {};
    object.testing = function() {
        return true;
    }
});

```

the above code has 5 anonymous functions...
This will not be easy to debug if there is an error somewhere as you will get error at anonymous()... (see TRUE\_ANON for a debug example).

By setting NAME\_ANON, the preprocessor will give a name to these functions and you will get the following output:

```
var testing = function $_testing1 () {
    return true;
}
var myObject = {
    doSomething : function $_doSomething1 () {
       console.log();
   }
};
document.body.onclick = function $_onclick1 (){
    myObject.doSomething();
}
document.body.addEventListener('click',function $_anon1(e){
    var object= {};
    object.testing = function $_testing2 () {
        return true;
    }
});

```

as you can see, when the functions are assigned to a variable or a property,the preprocessor use the variable/property name prefixed by ` "$_" ` and suffixed by an incremental number.

The incremental number is here to help a little when you have functions that are in different scopes but will end with the same name.
If you use a framework that requires some recurring code that can help.

When the anonymous function is not assigned to a variable/property, the preprocessor follow the same convention but use the default name "anon". which gives you, `"$_anon"` + an incremental number.


## define ##
```
$ jsmake -define [variable name=value variable name=value]
```
Example:
```
$ jsmake -define VERSION=6.01 AUTHOR=myself
```
vars allows you to overwrite the one inline in your javascript file.

So if you define in your javascript /**#define VERSION 3.01**/ but at the command line

set the variable to 6.01, the final output will be 6.01

Will look into merging vars and define functionality into one as it is confusing...

## config ##

If you're like me, remembering all these options can be quite cumbersome.
That's why from version 1.6, jsmake supports parameters setting via configuration file.
jsmake will look automaticly for a file called 'config.jsm' and load the parameters for you.

you can also specify your own configuration file:
```
$ jsmake -config ../myconfig.jsm
```

Beaware, that if you already have a config.jsm file and that you setup a -config file via the command line, you will get a cascading effect. More about this, afterwards.

### format ###

The format is rather straightforward:
```
parameter1:value
parameter2:value
```
On the command line, you need to prefix the parameters with '-'. In the config file you must not use the '-' prefix.
We could setup the following config.jsm:

config.jsm file:
```
input:../lib
output:../src/lib.js
vars:STRICT_MODE TRUE_ANON
compression:closure
packages: Core
```

Then we could also add an other configuration file on the command line:

```
$ jsmake -config myconfig.jsm
```

myconfig.jsm
```
vars:TRIDENT
packages:FX
```

It will basically do as follow:
```
input:../lib
output:../src/lib.js
vars:STRICT_MODE TRUE_ANON TRIDENT
compression:closure
packages: Core FX
```

you can even overwrite/add some settings on the command line:

```
$ jsmake -config myconfig.jsm -packages Easing -compression mini -output ../src/test.js
```

and jsmake will base its preprocessing on the following settings:
```
input:../lib
output:../src/test.js
vars:STRICT_MODE TRUE_ANON TRIDENT
compression:mini
packages: Core FX Easing
```


## help ##
```
$ jsmake -help
```
TO BE DONE!^^

That's it!!


# ENVIRONMENT #

Well, only tested this on mac and windows xp...

Should work under linux. If you try and give me some feedbacks!

# REQUIREMENTS #

- perl >= 5.8.8


## Warning ##

jsmake is a perl hack made in few hours.

It is a all-in-one ugly command line tool.

You shall not look into the source unless you want to loose faith in humanity.

If you find it useful or want some features, tell me so.

It may force me to do an entire rewrite with

clean packages,

clean separation of concerns,

clean oop interface,

a real parser with extensions

a suit of spec tests

and some real code inside...

Anyhow...

you've been warned.