# Front end development & build process

Issues with current process:
- Front end and backend are coupled together which is considered bad very practice
- 25 minute builds - take 3 developers doing 3 builds per day (under-estimate) thats thats 2 hours per day wasted
- Karma tests launch a chrome window for each test - unnecessary and is either poor sequencing or using the wrong web driver (should be using phantom)
- No JS/CSS/HTML bundling - loading time is very poor due to number of requests firing off when landing on the page.
- No cache busting techniques being used so cache issues will be affecting user experience

## Repo
- https://github.com/dan05011991/demo-application-frontend - Front end repository

## Installation

Run `npm install` - this installs all the dependent npm packages

Either run your own backend or run `docker run -d -p=8090:8090 dan05011991/demo-backend:0.0.131-SNAPSHOT`

Example prices:
Run `./node_modules/gulp/bin/gulp serve` to serve the website
Run `./node_modules/gulp/bin/gulp test` to run tests

## Possible issues and resolutions
- SNAPSHOT version is displayed on the bottom of the page which is populated during the build phase.
- - Solution here is either to add an endpoint to allow the front end to query the backend for it's version or to pass the version of the backend into the front end container. There are probably other solutions but on the top of my head these 2 are viable (I prefer the first)

## New tooling

### Gulp or Webpack
- Gulp is a JS pre-processor which runs in node and executes various task. You can do anything you would be able to do 
in node so the options are endless. 
- Webpack is used primarily for bundling but does have the ability to add plugins to match the capabilities of gulp.

Gulp is more useful for us at the moment due to its flexibility but if we move towards newer frameworks, webpack may be a better route to go down.

### Gulp tasks
Gulp is designed with a concept of `tasks`. A `task` is some sort of pre-processing. This pre-processing can be chained with other processing in series (sequential order) or in parallel to speed up processing. We can also specify a build argument to indicate whether this is a `debug` or `release` build. This instructs tasks to perform additional operations e.g. minify css & js.

#### Terminology
When `vendor` is used, it refers to files/packages that have been developer elsewhere e.g. bootstrap, angular.
When `site` is used, it refers to custom files/packages which we have written specifically for this project.

These are the tasks which have been written (all tasks watch their dependent sources meaning any changes to the sources trigger the bundle to be recreated):

### Assets
Bundles site and vendor fonts and outputs the bundle in the output directory.

### Clean
Cleans output directory prior to bundling - this is executed prior to bundling tasks

### Config
Specifies an additional javascript file (switches based on build argument) to be bundled in the `site` JS bundle, this can be:

```
var siteConfig = angular.module('site.config', [])
  .constant('siteConfig', {
    root: 'http://localhost',
    backendBase: 'http://localhost:8090/',
    appName: 'Demo Application',
    appVersion: 'dev'
  });
```
or
```
var siteConfig = angular.module('site.config', [])
  .constant('siteConfig', {
    root: 'http://frontend',
    backendBase: 'http://backend:8090/',
    appName: 'Demo Application',
    appVersion: '1.0.3'
  });
```

### index
Copies the index.html file into the output directory and add cache bundling tag onto each bundle

### jslint
Runs linter on javascript files

### paths
Config task which contains directories to process and where to output

### serve
Starts a server to serve output directory. Any source changes will retrigger a reload.

### siteCSS
Bundles all css files included in defined directories.

### siteJS
Bundles all js files included in defined directories. Directive partial templates are embedded in the js using `template`.
e.g:
```js
template: template('../partials/snippet.html')
```
before
```html
template: '<p>hello this is the snippet</p>'
```

### test
Runs karma tests using phantom. Any source changes will retrigger tests.

### vendorCSS
Bundles all `vendor` css files defined in included `package.json`.

### vendorJS
Bundles all `vendor` js files defined in included `package.json`.

### watch
Watches all sources

### wiring (possible future enhancement - not required)
- automatically add `require` statements in `app.js` for each module
- automatically adds `controller`, `directive`, `config` and `factory` declarations for each module for all files listed in predefined folders. e.g:
```
- app.js
- modules
-- module1
--- module1.js
--- controllers
---- controller1.js
--- directives
---- directive1.js
--- services
---- service1.js
--- config
---- config1.js
```

`app.js`
```
var app = angular.module('app', [
  'app.module1'
]);
```
becomes:
```js
require('./modules/module1');

var app = angular.module('app', [
  'app.module1'
]);
```

module1.js
```js
var module1 = angular.module('app.module1', []);
```
becomes:
```js
var module1 = angular.module('app.module1', []);
                    .directive('directive1', require('./directives/directive1'))
                    .controller('controller1', require('./directives/controller1'))
                    .factory('directive1', require('./directives/service1'))
                    .config('directive1', require('./directives/config1'));
```

## Result
- JS and CSS files are bundled requiring one include statement for each on the main html page
- Each bundle include statement includes a cache busting version param to avoid the browser using its cache e.g. 
```html
<script type="text/javascript" src="/css/vendorCSS.css?v=120703488947589432" />
<script type="text/javascript" src="/css/siteCSS.css?v=120703488947589432" />

<script type="text/javascript" src="/js/vendorJS.js?v=120703488947589432" />
<script type="text/javascript" src="/js/siteJS.js?v=120703488947589432" />
```
- Build of front end takes around 5-10 seconds
- Tests run in <5 seconds
