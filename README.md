# gulp-browserify

[![Build Status](https://img.shields.io/travis/romeovs/gulp-browserify.svg?style=flat-square)][travis]
[![Coverage Status](https://img.shields.io/coveralls/romeovs/gulp-browserify.svg?style=flat-square)][coveralls]
[![Dependencies](https://img.shields.io/david/romeovs/gulp-browserify.svg?style=flat-square)][david]
[![devDependencies](https://img.shields.io/david/dev/romeovs/gulp-browserify.svg?style=flat-square)][david-dev]
[![license](https://img.shields.io/badge/license-ISC-373737.svg?style=flat-square)][license]
[![gitter](https://img.shields.io/badge/GITTER-join%20chat%20→-00d86e.svg?style=flat-square)][gitter]

`gulp-browserify` is a drop-in replacement for [`browserify`][browserify] that
can be configured easily and has support for [`watchify`][watchify]
to improve performance drastically.

It is based on [this gulp recipe][recipe], but with some adaptations to make
configuring it easier.

## Installation
To install `gulp-browserify` do:

```
npm install --save-dev romeovs/gulp-browserify
```

## Usage
At it's most basic use, you can use `gulp-browserify` like this:

```js
var gulp       = require('gulp')
  , Browserify = require('gulp-browserify')
  ;

var options = {
  entries: [ './src/main.js' ]
};

gulp.task('browserify', function() {
  Browserify(options)
    .done(function(strm) {
      // strm is a gulp strm containing
      // the bundled file, you can do anything
      // you would do with a normal gulp stream
      strm
        // .pipe(somegulpplugin)
        .pipe(gulp.dest('./dest'));
    });
});
```

## Options
`gulp-browserify` has the same signature as [`browserify`][browserify]:

```js
Browserify(entries, options)
// or
Browserify(options)
```

All the options are passed to `browserify` internally so you can use
`gulp-browserify` as a drop-in replacement fro `browserify`.

### `options.entries`
An array of filenames that serve as an entrypoint, as with `browserify`.  Can
also be passed as a normal argument to `gulp-browserify`.

### `options.watching`
If this is set to `true`, `gulp-browserify` will keep 
on watching the all the relevant files and rebuild everytime
one of them changes.  This is done using [`watchify`][watchify],
which caches intermediate files, making the builds super fast.

Defaults to `false`.

### `options.delay`
This is the default delay (in ms) `gulp-browserify` will wait before 
triggering a rebuild.

Defaults to `60`.

### `options.filename`
The filename the eventual build file will have in the gulp pipe.

Defaults to `main.js`.

### `options.logtime`
The function used to log build times.  Gets the time
the build took in ms as a parameter.

Defaults to [a nice gulp-like logging function](./src/defaults/log-time.js)

### `options.logfiles`
The function to log the changed files when watching. Gets an array of the changed
files as parameter.

Defaults to [a nice gulp-like logging function](./src/defaults/log-files.js)

### `options.logerror`
A function to log errors with.  Gets the error as parameter.

Defaults to [a nice gulp-like logging function](./src/defaults/log-error.js)

## Middleware
You can configure the bundler even more by using middleware functions.
To attach some middleware, use the `configure` method before calling `done`:

```js
Browserify(options)
  .configure(function() {
    // configure middleware
  })
  .done(/* ... */)
```

A middleware function has access to three things:

  - `this.options` this is the options object
     as was passed to `gulp-browserify`, possibly
     already edited by other middleware.
  - `this.bundler` the bundler that was created by `browserify`.  You can call
    almost all methods on it (`transform`, `add`, `require`, `external`,
    `exclude`, `ignore`, `plugin`, `on`) to configure the bundler.
  - `this.done` a function that will handle the final stream created
     when the files are bundled.  This handler gets a stream and should 
     return the transformed stream.

You can change these properties by setting them to their new value in
the middleware function.  Make sure you take into account previous options
that might have been set in other middleware (eg. concat your new extension
to the `this.options.extensions` array instead of overwiting it).

As an example, here is a transform that adds `jsx` support via `babelify`:
```js
var babelify = require('babelify');
// ...
Browserify(options)
  .configure(function() {
    this.options.extensions = ['.jsx'].concat(this.options.extensions || []);

    this.bundler.transform(babelify.configure({
      experimental: options.experimental
    }));
  })
  .done(function(strm) {
    // ...
  });
};
```


## Todo
- write a lot more tests
- add examples
- add links to middleware repos


### License
This code is licensed under the [ISC license][license]

[travis]:     https://travis-ci.org/romeovs/gulp-browserify
[coveralls]:  https://coveralls.io/r/romeovs/gulp-browserify?branch=master
[david]:      https://david-dm.org/romeovs/gulp-browserify#info=dependencies
[david-dev]:  https://david-dm.org/romeovs/gulp-browserify#info=devDependencies
[gitter]:     https://gitter.im/romeovs/gulp-browserify?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge
[license]:    ./LICENSE
[browserify]: http://browserify.org
[watchify]:   https://github.com/substack/watchify
[recipe]:     https://github.com/gulpjs/gulp/blob/master/docs/recipes/fast-browserify-builds-with-watchify.md "gulp recipe"
