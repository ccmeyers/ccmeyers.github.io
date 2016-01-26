---
layout: post
title: "Don't Fence Me In: How to use Gulp with Shopify and Timber to Gain Control of Your Front End"
date: 2015-12-03 11:59:01 -0500
comments: true
categories:
- Shopify
- Timber
- Gulp
- Bourbon
- Neat
---

For my first Shopify project, I decided to use the <a href="http://shopify.github.io/Timber/" target="_blank">Timber</a> framework to help with my theme development. I think it's a great way to get your theme started fast with the right setup. However, I hated the lack of organization and control of your front end code. Everything was dumped in the 'assets' folder. Images and JS files and SCSS files all thrown in together? I think not! Plus I wasn't a huge fan their grid system and wanted to use <a href="http://neat.bourbon.io/" target="_blank">Bourbon Neat</a>, which is my favorite grid system. But here's what Timber has to say about imports and mixins and Bourbon Neat:
> You cannot use @imports in a Sass liquid file. That means no dependence on Compass, Bourbon or similar Sass frameworks.

Don't tell me what to do Timber! I'm a grown woman, and I'll use Bourbon if I want to!

So, I set out to set up my shopify-theme to be able to precompile assets locally. There's a bunch of documentation and blog posts on how to use Grunt to do this, but I like Gulp better...so I decided to figure that out. 

First, create your project. In your terminal:
`mkdir my-shopify-shop`
`cd my-shopify-shop`

Then bring in Timber:
`git clone https://github.com/Shopify/Timber.git`

Now that we have all the Timber files, we need to upload those to Shopify. For this we're going to use the <a href="https://github.com/Shopify/shopify_theme" target="_blank">shopify_theme gem</a>.

`cd Timber`

Open the `Gemfile` inside the Timber folder and add `gem 'shopify_theme'`. Then run `bundle install` in your terminal. In the root of the Timber folder, you have a `config-sample.yml` file. Duplicate that file, rename it `config.yml` and add your Shopify API  credentials to it. To get your API credentials, go into your Shopify admin panel and create a <a href="http://docs.shopify.com/api/authentication/creating-a-private-app" target="_blank">private app</a>. 

Then add `config.yml` to your `.gitignore` (the one in the root of your project, not the Timber folder).

Using the shopify_theme gem, we are going to run `theme replace` in your terminal. Make sure you are in the `Timber` folder while doing this so it has access to your `Gemfile` and your `config.yml`. This will replace all of Shopify's default files, with Timber's default files. You only have to do this once. From here on out, any file we save will automatically be uploaded to Shopify through another library. 

Go back to the root of your project `cd ..`.

Then make yourself a folder where you'll put all your precompiled assets. I named mine 'lib':
`mkdir lib`
`cd lib`
`mkdir js scss images`

Here's what your file structure looks like so far:
```
my-shopify-shop
├── lib
│   └── images
│   └── js
│   └── scss
├── Timber
│   └── All the automatically installed Timber stuff (assets, config, layout, etc.)
```

Next let's start our gulp configuration. We need it to do the following:
1. Compile our scss and js files
2. Optimize images
3. Watch for changes
4. Upload any changes to Shopify

One disclaimer before we start. Say good-bye to livereload right now. Shed some tears and move on, cuz it ain't gonna work (at least that I've tried). This is because we're working from Shopify's servers and not our own local environment. Our changes have to be pushed up to Shopfiy, then you have to make a hard refresh to see the changes. I know. Life is hard.

Go back to the root `cd ..` and `touch gulpfile.js`

You're also going to have to create a 'package.json' for all your node_modules.
`npm init` and press enter through all the prompts until it's created.

Let's start with uploading changes to Shopify.
`npm install --global gulp`
`npm install --save-dev gulp gulp-watch gulp-shopify-upload`
That's going to install both <a href="https://github.com/gulpjs/gulp/blob/master/docs/getting-started.md" target="_blank">gulp</a>, <a href="https://www.npmjs.com/package/gulp-watch" target="_blank"></a>, and <a href="https://github.com/mikenorthorp/gulp-shopify-upload" target="_blank">gulp-shopify-upload</a>

Create a file called `config.json` in your root and put your Shopify API credentials in there. It should look something like this (but obviously with all your credentials):

config.json
```json
{
  "shopify_api_key": "whateveryourapikeyishere",
  "shopify_api_password": "whateveryourapipasswordishere",
  "shopify_url": "YOUR-STORE-NAME.myshopify.com"
}
```
You can also add your theme id if you're using one. Don't forget to add `config.json` to your `.gitignore`!!!! You don't want to be sharing your Shopify API credentials with the world.

Now let's create a task for gulp-shopify-upload.
```javascript
// Gulp plugin setup
var gulp = require('gulp');
// Watches single files
var watch = require('gulp-watch');
var gulpShopify = require('gulp-shopify-upload');
// Grabs your API credentials
var config = require('config.json');
 
gulp.task('shopifywatch', function() {
  var options = {
    "basePath": "./Timber/"
  };

  return watch('./Timber/+(assets|layout|config|snippets|templates|locales)/**')
  .pipe(gulpShopify(config.shopify_api_key, config.shopify_api_password, config.shopify_url, null, options));
});
 
// Default gulp action when gulp is run
gulp.task('default', [
  'shopifywatch'
]);
```

Notice a few things here. We changed the url for the watch command to go one level deeper since we're changing the file structure from the example given in gulp-shopify-upload's docs. And we have to add the option for basePath for the same reason. Also, we've moved all our API credentials into a secret config.json file that we'll require at the top.

Now if you run `gulp` in your terminal, it'll watch for any changes. Go ahead and make a change to one of the templates. You'll see it say 'Upload Complete' in the terminal. But remember, you'll have to do a hard refresh in order to see the change in your browser. Did a single tear just fall down your cheek? I know. Me too.

Next we'll want to precompile our scss and spit the compiled file into Timber's assets folder. Also, I'm going to include Bourbon Neat here along with some handy error handling.

`npm install --save-dev gulp-sass gulp-autoprefixer gulp-notify node-bourbon node-neat`

Add to your gulpfile:
```javascript
// Compiles SCSS files
var sass         = require('gulp-sass');
var autoprefixer = require('gulp-autoprefixer');
// Notifies of errors
var notify = require("gulp-notify");
// Includes the Bourbon Neat libraries
var neat         = require('node-neat').includePaths;

function handleErrors() {
  var args = Array.prototype.slice.call(arguments);

  // Send error to notification center with gulp-notify
  notify.onError({
    title: "Compile Error",
    message: "<%= error %>"
  }).apply(this, args);

  // Keep gulp from hanging on this task
  this.emit('end');
}

gulp.task('sass', function () {
  gulp.src('./lib/scss/*.{sass,scss}')
    .pipe(sass({includePaths: neat}))
    .on('error', handleErrors)
    .pipe(autoprefixer({ browsers: ['last 2 version'] }))
    .pipe(gulp.dest('./Timber/assets'));
});

gulp.task('watch', function () {
  gulp.watch('./lib/scss/**/*.{sass,scss}', ['sass']);
});
```

And add it to the default task:
```javascript
gulp.task('default', [
  'shopifywatch', 'watch'
]);
```

Change the stylesheet tag in Timber's template to point to your compiled file instead of the default 'timber.scss.css'. For me it's on line 31 of `layout/theme.liquid`.

Now to compile our JavaScript files. For this we're using <a href="http://browserify.org/", target="_blank">browserify</a>, <a href="https://github.com/substack/watchify" target="_blank">watchify</a>, and <a href="https://www.npmjs.com/package/vinyl-source-stream" target="_blank">vinyl-source-stream</a>. Keep in mind, this is a pretty simple gulpfile. You can really beef this up, especially when it comes to browserify.

`npm install --save-dev browserify watchify vinyl-source-stream`

Add to your gulpfile:
```javascript
// Concats your JS files
var browserify = require('browserify');
var watchify = require('watchify');
var source = require('vinyl-source-stream');

gulp.task('browserify', function() {
  return browserify('./lib/js/app.js')
      .bundle()
      .on('error', handleErrors)
      //Pass desired output filename to vinyl-source-stream
      .pipe(source('bundle.js'))
      // Start piping stream to tasks!
      .pipe(gulp.dest('./Timber/assets/'));
});
```

And add this to your watch task:
```javascript
gulp.task('watch', function () {
  gulp.watch('./lib/scss/**/*.{sass,scss}', ['sass']);
  gulp.watch('./lib/js/**/*.js', ['browserify']);

  var watcher = watchify(browserify({
    // Specify the entry point of your app
    entries: ['./lib/js/app.js'],
    debug: true,
    cache: {}, packageCache: {}, fullPaths: true
  }));

  return watcher.on('update', function() {
    watcher.bundle()
      .pipe(source('bundle.js'))
      .pipe(gulp.dest('./Timber/assets/'))
  })
});
```

Add another script tag below the one calling for `timber.js` (for me it's at line 379) to call for your bundled js `bundle.js` in `layout/theme.liquid`.

Now let's opmtimize our images. For that we use <a href="https://github.com/sindresorhus/gulp-imagemin" target="_blank">gulp-imagemin</a> and <a href="https://www.npmjs.com/package/gulp-changed" target="_blank">gulp-changed</a>.

`npm install --save-dev gulp-imagemin gulp-changed`

Add this to your gulpfile:
```javascript
var imagemin   = require('gulp-imagemin');
var changed    = require('gulp-changed');

gulp.task('images', function() {
  return gulp.src('./lib/images/**')
    .pipe(changed('./Timber/assets/')) // Ignore unchanged files
    .pipe(imagemin()) // Optimize
    .pipe(gulp.dest('./Timber/assets/'))
});
```

And add `gulp.watch('lib/images/*.{jpg,jpeg,png,gif,svg}', ['images']);` to your gulp watch task.

If you're using local font files, you'll also want to add a fonts taks that will throw your font files in Timber's asset folder as well.

You can check out a finished example of this code <a href="https://github.com/ccmeyers/shopify-timber-gulp-setup" target="_blank">here</a>.