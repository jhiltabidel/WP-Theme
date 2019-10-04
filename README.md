# gate39theme
A Starter Theme for Gate 39 Media

# Branching

## Quick Legend
<table>
  <thead>
    <tr>
      <th>Instance</th>
      <th>Branch</th>
      <th>Description, Instructions, Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Stable</td>
      <td>stable</td>
      <td>Latest code deployed to production ( Production )</td>
    </tr>
    <tr>
      <td>Working</td>
      <td>master</td>
      <td>Latest delivered development changes ( Development )</td>
    </tr>
  </tbody>
</table>

## Main Branches

The main repository will always hold two branches:

* `master`
* `stable`

The main branch should be considered `origin/master` and will be the main branch where the source code of `HEAD` always reflects a state with the latest delivered development changes.

Consider `origin/stable` to always represent the latest code deployed to production.

When the source code in the `master` branch is stable and has been deployed, all of the changes will be merged into `stable` and tagged with a release number.

# Installing gulp
```nodejs
npm install gulp-cli -g
npm install gulp -D
touch gulpfile.js
```

# Setting up the project.
```nodejs
npm init //This will run through creating the package.json file
npm install -g gulp //If you haven't installed gulp globally before
npm install --save-dev gulp
npm install --save-dev gulp-sass
npm install --save-dev gulp-sourcemaps
npm install --save-dev gulp-rename
npm install --save-dev gulp-uglify
npm install --save-dev gulp-util
npm install --save-dev vinyl-ftp
```

# Running gulp
gulp gulpfile.js

# gulpfile configuration

```nodejs
'use strict';

var gulp = require('gulp');
var sass = require('gulp-sass');
var sourcemaps = require("gulp-sourcemaps");
var rename = require('gulp-rename');
var uglify = require('gulp-uglify');

var gutil = require( 'gulp-util' );
var ftp = require( 'vinyl-ftp' );

/** Configuration **/
var user = 'gate39theme';
var password = '';
var host = '18.220.82.191';
var port = 21;
var localFilesGlob = ['wp-content/**','sample/**','docs/**'];
var remoteFolder = '/'

// helper function to build an FTP connection based on our configuration
function getFtpConnection() {
    return ftp.create({
        host: host,
        port: port,
        user: user,
        password: password,
        parallel: 4,
        reload : true,
        log: gutil.log
    });
}

gulp.task("styles-gate39media-app", function () {
    gulp.src("wp-content/themes/gate39media/sass/**/*.scss")
    .pipe(sourcemaps.init())
    .pipe(sass({outputStyle: "compressed"}).on("error", sass.logError))
    //.pipe(rename({basename: 'gate39media-app'}))
    .pipe(sourcemaps.write("./"))
    .pipe(gulp.dest("wp-content/themes/gate39media/"))
})

gulp.task('compress-js-gate39media-js', function () {
  gulp.src('wp-content/themes/gate39media/js/gate39media-site.js')
    .pipe(uglify())
    .pipe(rename({ suffix: '.min' }))
    .pipe(gulp.dest('wp-content/themes/gate39media/js/'))
})

gulp.task('compress-gate39media-js-remote-post', function () {
  gulp.src('wp-content/themes/gate39media/js/gate39media-remote-post.js')
    .pipe(uglify())
    .pipe(rename({ suffix: '.min' }))
    .pipe(gulp.dest('wp-content/themes/gate39media/js/'))
})

gulp.task('compress-gate39media-js-quotes', function () {
  gulp.src('wp-content/themes/gate39media/js/gate39media-quotes.js')
    .pipe(uglify())
    .pipe(rename({ suffix: '.min' }))
    .pipe(gulp.dest('wp-content/themes/gate39media/js/'))
})

//Watch task
gulp.task('default',function() {

    //CSS
    gulp.watch('wp-content/themes/gate39media/sass/**/*.scss',['styles-gate39media-app']);

    //JS
    gulp.watch("wp-content/themes/gate39media/js/gate39media-site.js", ["compress-js-gate39media-js"]);
    gulp.watch("wp-content/themes/gate39media/js/gate39media-remote-post.js", ["compress-gate39media-js-remote-post"]);
    gulp.watch("wp-content/themes/gate39media/js/gate39media-quotes.js", ["compress-gate39media-js-quotes"]);

    var conn = getFtpConnection();

    gulp.watch(localFilesGlob)
    .on('change', function(event) {
      console.log('Changes detected! Uploading file "' + event.path + '", ' + event.type);

      return gulp.src( [event.path], { base: '.', buffer: false } )
        .pipe( conn.newer( remoteFolder ) ) // only upload newer files
        .pipe( conn.dest( remoteFolder ) )
      ;
    });

});
```
