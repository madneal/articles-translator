https://github.com/webpack/docs/wiki/how-to-write-a-plugin

Plugins expose the full potential of the Webpack engine to third-party developers. Using staged build callbacks, developers can introduce their own behaviors into the Webpack build process. Building plugins is a bit more advanced than building loaders, because you'll need to understand some of the Webpack low-level internals to hook into them. Be prepared to read some source code!

插件能够将webpack引擎的全部潜力暴露给第三方的开发者。

## Compiler and Compilation

Among the two most important resources while developing plugins are the `compiler` and `compilation` objects. Understanding their roles is an important first step in extending the Webpack engine.

- The `compiler` object represents the fully configured Webpack environment. This object is built once upon starting Webpack, and is configured with all operational settings including options, loaders, and plugins. When applying a plugin to the Webpack environment, the plugin will receive a reference to this compiler. Use the compiler to access the main Webpack environment.

- A `compilation` object represents a single build of versioned assets. While running Webpack development middleware, a new compilation will be created each time a file change is detected, thus generating a new set of compiled assets. A compilation surfaces information about the present state of module resources, compiled assets, changed files, and watched dependencies. The compilation also provides many callback points at which a plugin may choose to perform custom actions.

These two components are an integral part of any Webpack plugin (especially a `compilation`), so developers will benefit by familiarizing themselves with these source files:

- [Compiler Source](https://github.com/webpack/webpack/blob/master/lib/Compiler.js)
- [Compilation Source](https://github.com/webpack/webpack/blob/master/lib/Compilation.js)

## Basic plugin architecture

Plugins are instanceable objects with an `apply` method on their prototype. This `apply` method is called once by the Webpack compiler while installing the plugin. The `apply` method is given a reference to the underlying Webpack compiler, which grants access to compiler callbacks. A simple plugin is structured as follows:

```javascript
function HelloWorldPlugin(options) {
  // Setup the plugin instance with options...
}

HelloWorldPlugin.prototype.apply = function(compiler) {
  compiler.plugin('done', function() {
    console.log('Hello World!'); 
  });
};

module.exports = HelloWorldPlugin;
```

Then to install the plugin, just include an instance in your Webpack config `plugins` array:

```javascript
var HelloWorldPlugin = require('hello-world');

var webpackConfig = {
  // ... config settings here ...
  plugins: [
    new HelloWorldPlugin({options: true})
  ]
};
```

## Accessing the compilation

Using the compiler object, you may bind callbacks that provide a reference to each new compilation. These compilations provide callbacks for hooking into numerous steps within the build process.

```javascript
function HelloCompilationPlugin(options) {}

HelloCompilationPlugin.prototype.apply = function(compiler) {

  // Setup callback for accessing a compilation:
  compiler.plugin("compilation", function(compilation) {
    
    // Now setup callbacks for accessing compilation steps:
    compilation.plugin("optimize", function() {
      console.log("Assets are being optimized.");
    });
  });
};

module.exports = HelloCompilationPlugin;
```

For more information on what callbacks are available on the `compiler`, `compilation`, and other important objects, see the [[plugins API|plugins]] doc.

## Async compilation plugins

Some compilation plugin steps are asynchronous, and pass a callback function that _must_ be invoked when your plugin is finished running.

```javascript
function HelloAsyncPlugin(options) {}

HelloAsyncPlugin.prototype.apply = function(compiler) {
  compiler.plugin("emit", function(compilation, callback) {

    // Do something async...
    setTimeout(function() {
      console.log("Done with async work...");
      callback();
    }, 1000);

  });
};

module.exports = HelloAsyncPlugin;
```

## A simple example

Once we can latch onto the Webpack compiler and each individual compilation, the possibilities become endless for what we can do with the engine itself. We can reformat existing files, create derivative files, or fabricate entirely new assets.

Let's write a simple example plugin that generates a new build file called `filelist.md`; the contents of which will list all of the asset files in our build. This plugin might look something like this:

```javascript
function FileListPlugin(options) {}

FileListPlugin.prototype.apply = function(compiler) {
  compiler.plugin('emit', function(compilation, callback) {
    // Create a header string for the generated file:
    var filelist = 'In this build:\n\n';

    // Loop through all compiled assets,
    // adding a new line item for each filename.
    for (var filename in compilation.assets) {
      filelist += ('- '+ filename +'\n');
    }
    
    // Insert this list into the Webpack build as a new file asset:
    compilation.assets['filelist.md'] = {
      source: function() {
        return filelist;
      },
      size: function() {
        return filelist.length;
      }
    };

    callback();
  });
};

module.exports = FileListPlugin;
```

## Useful Plugin Patterns

Plugins grant unlimited opportunity to perform customizations within the Webpack build system. This allows you to create custom asset types, perform unique build modifications, or even enhance the Webpack runtime while using middleware. The following are some features of Webpack that become very useful while writing plugins.

### Exploring assets, chunks, modules, and dependencies

After a compilation is sealed, all structures within the compilation may be traversed.

```javascript
function MyPlugin() {}

MyPlugin.prototype.apply = function(compiler) {
  compiler.plugin('emit', function(compilation, callback) {
    
    // Explore each chunk (build output):
    compilation.chunks.forEach(function(chunk) {
      // Explore each module within the chunk (built inputs):
      chunk.modules.forEach(function(module) {
        // Explore each source file path that was included into the module:
        module.fileDependencies.forEach(function(filepath) {
          // we've learned a lot about the source structure now...
        });
      });

      // Explore each asset filename generated by the chunk:
      chunk.files.forEach(function(filename) {
        // Get the asset source for each file generated by the chunk:
        var source = compilation.assets[filename].source();
      });
    });

    callback();
  });
};

module.exports = MyPlugin;
```

- `compilation.modules`: An array of modules (built inputs) in the compilation. Each module manages the build of a raw file from your source library.

- `module.fileDependencies`: An array of source file paths included into a module. This includes the source JavaScript file itself (ex: `index.js`), and all dependency asset files (stylesheets, images, etc) that it has required. Reviewing dependencies is useful for seeing what source files belong to a module.

- `compilation.chunks`: An array of chunks (build outputs) in the compilation. Each chunk manages the composition of a final rendered assets.

- `chunk.modules`: An array of modules that are included into a chunk. By extension, you may look through each module's dependencies to see what raw source files fed into a chunk.

- `chunk.files`: An array of output filenames generated by the chunk. You may access these asset sources from the `compilation.assets` table.

### Monitoring the watch graph

While running Webpack middleware, each compilation includes a `fileDependencies` array (what files are being watched) and a `fileTimestamps` hash that maps watched file paths to a timestamp. These are extremely useful for detecting what files have changed within the compilation:

```javascript
function MyPlugin() {
  this.startTime = Date.now();
  this.prevTimestamps = {};
}

MyPlugin.prototype.apply = function(compiler) {
  compiler.plugin('emit', function(compilation, callback) {
    
    var changedFiles = Object.keys(compilation.fileTimestamps).filter(function(watchfile) {
      return (this.prevTimestamps[watchfile] || this.startTime) < (compilation.fileTimestamps[watchfile] || Infinity);
    }.bind(this));
    
    this.prevTimestamps = compilation.fileTimestamps;
    callback();
  }.bind(this));
};

module.exports = MyPlugin;
```

You may also feed new file paths into the watch graph to receive compilation triggers when those files change. Simply push valid filepaths into the `compilation.fileDependencies` array to add them to the watch. Note: the `fileDependencies` array is rebuilt in each compilation, so your plugin must push its own watched dependencies into each compilation to keep them under watch.

### Changed chunks

Similar to the watch graph, it's fairly simple to monitor changed chunks (or modules, for that matter) within a compilation by tracking their hashes.

```javascript
function MyPlugin() {
  this.chunkVersions = {};
}

MyPlugin.prototype.apply = function(compiler) {
  compiler.plugin('emit', function(compilation, callback) {
    
    var changedChunks = compilation.chunks.filter(function(chunk) {
      var oldVersion = this.chunkVersions[chunk.name];
      this.chunkVersions[chunk.name] = chunk.hash;
      return chunk.hash !== oldVersion;
    }.bind(this));
    
    callback();
  }.bind(this));
};

module.exports = MyPlugin;
```