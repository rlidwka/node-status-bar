status-bar
==========

#### A status bar for file transfers ####

[![NPM version](https://badge.fury.io/js/status-bar.png)](http://badge.fury.io/js/status-bar "Fury Version Badge")
[![Dependency Status](https://david-dm.org/gagle/node-status-bar.png)](https://david-dm.org/gagle/node-status-bar "David Dependency Manager Badge")

[![NPM installation](https://nodei.co/npm/status-bar.png?mini=true)](https://nodei.co/npm/status-bar "NodeICO Badge")

#### Example ####

```javascript
var statusBar = require ("status-bar");

var bar = statusBar.create ({
  //Total file size
  total: size,
  //Render function
  render: function (stats){
    //Print the status bar as you like
    process.stdout.write (filename + " " + 
        this.format.storage (stats.currentSize) + " " +
        this.format.speed (stats.speed) + " " +
        this.format.time (stats.remainingTime) + " [" +
        this.format.progressBar (stats.percentage) + "] " +
        this.format.percentage (stats.percentage));
    process.stdout.cursorTo (0);
  }
});

//Update the status bar when you send or receive a chunk of a file
obj.on ("some-event", function (chunk){
  //You can pass any object that contains a length property or a simple number
  bar.update (chunk);
});

//Or simply pipe() things to it!
stream.pipe (bar);

//It will print something like this
//a-file                  17.8 MiB   23.6M/s 00:13 [#·······················]   6%
```

#### Why you should try this module ####

- It doesn't print anything, it just calculates and returns raw data and provides default formatting functions. Other modules similar to this force you to use their own formatting functions with the `readline` module, which is very unstable and may cause problems if you are already using a `readline` instance.
- The status bar can be displayed wherever you want, it is simply a string, so you can render it in the console, in HTML (probably with your own progress bar) sending it via websockets or with [node-webkit](https://github.com/rogerwang/node-webkit), etc.
- You decide how to format and arrange the elements. The default formatting functions have a fixed length, so you can format the status bar very easily.
- It is very easy to use. Just `pipe()` things to it!

#### Download example ####

```javascript
var http = require ("http");
var statusBar = require ("status-bar");

var url = "http://nodejs.org/dist/latest/node.exe";

http.get (url, function (res){
  var bar = statusBar.create ({
    total: res.headers["content-length"],
    render: function (stats){
      process.stdout.write (
          url + " " +
          this.format.storage (stats.currentSize) + " " +
          this.format.speed (stats.speed) + " " +
          this.format.time (stats.remainingTime) + " [" +
          this.format.progressBar (stats.percentage) + "] " +
          this.format.percentage (stats.percentage));
      process.stdout.cursorTo (0);
    }
  });
  
  res.pipe (bar);
}).on ("error", function (error){
  console.error (error);
});
```

Prints something similar to this:

```
http://nodejs.org/dist/latest/node.exe    2.7 MiB  502.4K/s 00:07 [############············]  49%
```

#### Render function examples ####

- `pacman` from Arch Linux:
  
  ```
  a-file                  17.8 MiB   23.6M/s 00:13 [#·······················]   6%
  ```

  ```javascript
  var statusBar = require ("status-bar");
  
  var formatFilename = function (filename){
    //80 - 59
    var filenameMaxLength = 21;
    if (filename.length > filenameMaxLength){
      filename = filename.slice (0, filenameMaxLength - 3) + "...";
    }else{
      var remaining = filenameMaxLength - filename.length;
      while (remaining--){
        filename += " ";
      }
    }
    return filename;
  };
  
  filename = formatFilename (filename);
  
  var render = function (stats){
    process.stdout.write (filename + " " + 
        this.format.storage (stats.currentSize) + " " +
        this.format.speed (stats.speed) + " " +
        this.format.time (stats.remainingTime) + " [" +
        this.format.progressBar (stats.percentage) + "] " +
        this.format.percentage (stats.percentage));
    process.stdout.cursorTo (0);
  };
  
  var bar = statusBar.create ({
    total: ...,
    render: render
  });
  ```

- `git clone`:
  
  ```
  Receiving objects: 18% (56655992/311833402), 54.0 MiB | 26.7M/s
  ```

  ```javascript
  var statusBar = require ("status-bar");
  
  var render = function (stats){
    process.stdout.write ("Receiving objects: " +
        this.format.percentage (stats.percentage).trim () +
        " (" + stats.currentSize + "/" + stats.totalSize + "), " +
        this.format.storage (stats.currentSize).trim () + " | " +
        this.format.speed (stats.speed).trim ());
    process.stdout.cursorTo (0);
  };
  
  
  var bar = statusBar.create ({
    total: ...,
    render: render
  });
  ```

#### Functions ####

- [_module_.create(options) : StatusBar](#create)

#### Objects ####

- [StatusBar](#statusbar_object)

---

<a name="create"></a>
___module_.create(options) : StatusBar__

Returns a new [StatusBar](#statusbar_object) instance.

Options:

- __finish__ - _Function_  
	Function that is called when the file transfer has finished.
- __frequency__ - _Number_  
  The rendering frequency in milliseconds. It must be a positive value. Default is 200.
- __progressBarComplete__ - _String_  
  The character that shows completion progress. Default is `#`.
- __progressBarIncomplete__ - _String_  
  The character that shows the remaining progress. Default is `·`.
- __progressBarLength__ - _Number_  
  The length of the progress bar. Default is 24.
- __render__ - _Function_  
	Function that is called when the status bar needs to be printed. It is required. It receives the stats object as an argument. All of its properties contain raw data, so you need to format them. You can use the default formatting functions.

  Stats:
  
  - __currentSize__ - _Number_  
  The current size in bytes.
  - __remainingSize__ - _Number_  
  The remaining size in bytes.
  - __totalSize__ - _Number_  
  The total size in bytes.
  - __percentage__ - _Number_  
  The complete percentage. A number between 0 and 1.
  - __speed__ - _Number_  
  The estimated current speed in bytes per second.
  - __elapsedTime__ - _Number_  
  The elapsed time in seconds.
  - __remainingTime__ - _Number_  
  The estimated remaining time in seconds. If the remaining time cannot be estimated because the status bar needs at least 2 chunks or because the transfer it's hung up, it returns `undefined`.
  
- __total__ - _Number_  
  The total size of the file. This option is required.

---

<a name="statusbar_object"></a>
__StatusBar__

__Methods__

- [StatusBar#cancel() : undefined](#statusbar_cancel)
- [StatusBar#update(chunk) : undefined](#statusbar_update)

__Properties__

- [StatusBar#format : Formatter](#statusbar_format)

---

<a name="statusbar_cancel"></a>
__StatusBar#cancel() : undefined__

When you need to cancel the rendering of the status bar because the transfer has been aborted due to an error or any other reason, call to this function to clear the timer.

---

<a name="statusbar_format"></a>
__StatusBar#format : Formatter__

Returns a [Formatter](#formatter) instance.

---

<a name="statusbar_update"></a>
__StatusBar#update(chunk) : undefined__

Updates the status bar. The `chunk` can be any object with a length property or a simple number.

---

<a name="formatter"></a>
__Formatter__

__Methods__

- [Formatter#percentage(percentage) : String](#formatter-percentage)
- [Formatter#progressBar(percentage) : String](#formatter-progressBar)
- [Formatter#speed(bytesPerSecond) : String](#formatter-speed)
- [Formatter#storage(bytes) : String](#formatter-storage)
- [Formatter#time(seconds) : String](#formatter-time)

---

<a name="formatter-percentage"></a>
__Formatter#percentage(percentage) : String__

The percentage must be a number between 0 and 1. Result string length: 4.

```javascript
console.log (this.format.percentage (0.5));
// 50%
```

---

<a name="formatter-progressbar"></a>
__Formatter#progressBar(percentage) : String__

The percentage must be a number between 0 and 1. Result string length: the length configured with the option `progressBarLength`.

```javascript
console.log (this.format.progressBar (0.06));
//#·······················
```

---

<a name="formatter-speed"></a>
__Formatter#speed(bytesPerSecond) : String__

Speed in bytes per second. Result string length: 9.

```javascript
console.log (this.format.speed (30098226));
//  30.1M/s
```

---

<a name="formatter-storage"></a>
__Formatter#storage(bytes) : String__

Result string length: 10.

```javascript
console.log (this.format.storage (38546744));
//  36.8 MiB
```

---

<a name="formatter-time"></a>
__Formatter#time(seconds) : String__

Result string length: 5 (_min_:_sec_). If `seconds` is undefined it prints `--:--`.

```javascript
console.log (this.format.time (63));
//01:03
```