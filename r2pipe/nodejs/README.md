Pipe bindings for radare2 (r2pipe)
==================================
![r2pipe logo](http://lolcathost.org/b/r2pipe.png)

The r2pipe APIs are based on a single r2 primitive found behind r_core_cmd_str() which is a function that accepts a string parameter describing the r2 command to run and returns a string with the result.

The decision behind this design comes from a series of benchmarks with different libffi implementations and resulted that using the native API is more complex and slower than just using raw command strings and parsing the output.

As long as the output can be tricky to parse, it's recommended to use the JSON output and deserializing them into native language objects which results much more handy than handling and maintaining internal data structures and pointers.

Also, memory management results into a much simpler thing because you only have to care about freeing the resulting string.


Getting Started
===============

This plugin requires radare >= `0.9.8` (some features such as rlangpipe require git version)

Once radare2 is installed, you may install this plugin using this command:

```shell
npm install r2pipe
```

Once the plugin has been installed, you can load it with this line of JavaScript:

```js
var r2pipe = require('r2pipe');
```

Access methods
==============

There are multiple ways to interact with a radare2 session

### pipe

Spawns a new process and comunicate with it through standard stdin, stdout, stderr file descriptors

```js
var r2pipe = require('r2pipe');

function doSomeStuff(r2) {
   ...
}

r2pipe.pipe ("/bin/ls", doSomeStuff);
```

### rlangpipe

This method is intended to be used while running scripts from the r2 console

```js
var r2pipe = require('r2pipe');

function doSomeStuff(r2) {
   ...
}

r2pipe.rlangpipe (doSomeStuff);
```

Execute the script from r2 command prompt
```
$ r2 binfile.elf
[0x080480a0]> #!pipe node /tmp/yourscript.js
Analizing file
Analysis finished
Searching for syscalls
 - found Syscall: write
 - found Syscall: close
 - found Syscall: read
 - found Syscall: write
 - found Syscall: exit
 - found Syscall: write
 - found Syscall: munmap
 - found Syscall: mmap
[0x080480a0]> 
```

### launch

Launch radare2 and listen for cmds through a tcp port

```js
var r2pipe = require('r2pipe');

function doSomeStuff(r2) {
   ...
}

r2pipe.launch ("/bin/ls", doSomeStuff);
```


### connect

Connect to an already running radare2 instance running an http listener

```bash
~$ r2 -
 -- Trust no one, nor a zero. Both lie.
[0x00000000]> =h 8182
Starting http server...
open http://localhost:8182/
r2 -C http://localhost:8182/cmd/
```

```js
var r2pipe = require('r2pipe');

function doSomeStuff(r2) {
   ...
}

r2pipe.connect ("http://localhost:8182/cmd/", doSomeStuff);
```

API
===

r2pipes provides five basic commands


### cmd

Runs a radare2 command

```js
var r2pipe = require('r2pipe');

function doSomeStuff(r2) {
   r2.cmd ("iS", function(output) {
    console.log (output);
  });
}

r2pipe.launch ("/bin/ls", doSomeStuff);
```


### cmdj

Runs a radare2 command and tries to convert the output into an object.

Note that this will only work with commands producing JSON output.

```js
var r2pipe = require('r2pipe');

function doSomeStuff(r2) {
   r2.cmdj ("iSj", function(output) {
    if (output !== null)
       console.log (output);
    else
       console.log("An error has occurred");
  });
}

r2pipe.launch ("/bin/ls", doSomeStuff);
```

In case of error "null" will be passed as argument to the callback instead of an valid object

### syscmd

Runs a system command, used mainly to access radare companion tools such as rabin2, raddif2, etc...

```js
var r2pipe = require('r2pipe');

function doSomeStuff(r2) {
   r2.syscmd ("rabin2 -S /bin/true", function(output) {
    console.log (output);
  });
}

r2pipe.launch ("/bin/ls", doSomeStuff);
```


### syscmdj

Runs a system command and tries to convert the output into an object.

Note that this will only work with commands producing JSON output.

```js
var r2pipe = require('r2pipe');

function doSomeStuff(r2) {
   r2.syscmdj ("rabin2 -j -S /bin/true", function(output) {
    console.log (output);
  });
}
t
r2pipe.launch ("/bin/ls", doSomeStuff);
```

In case of error "null" will be passed as argument to the callback instead of an valid object


### quit

Close the connection or kill the radare spawned process

```js
var r2pipe = require('r2pipe');

function doSomeStuff(r2) {
   r2.quit();
}

r2pipe.launch ("/bin/ls", doSomeStuff);
```


Example
=======

This is a small example using the pipe connection method for standalone scripts.

```js
var r2pipe = require ("r2pipe");


function doSomeStuff(r2) {

  r2.cmdj ("aij entry0+2", function(o) {
    console.log (o);
  });

  r2.cmd ('af @ entry0', function(o) {
    r2.cmd ("pdf @ entry0", function(o) {
      console.log (o);
      r2.quit ()
    });
  });

}


r2pipe.pipe ("/bin/ls", doSomeStuff);
r2pipe.launch ("/bin/ls", doSomeStuff);
r2pipe.connect ("http://cloud.rada.re/cmd/", doSomeStuff);
```

