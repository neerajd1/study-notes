Resources

https://stackoverflow.com/questions/35062852/npm-vs-bower-vs-browserify-vs-gulp-vs-grunt-vs-webpack


Theory

npm & bower are package managers. They just download the dependencies and don't know how to build projects on their own. What they know is to call webpack/gulp/grunt after fetching all the dependencies.
bower is like npm, but builds a flattened dependency trees (unlike npm which does it recursively). Meaning npm fetches the dependencies for each dependency (may fetch the same a few times), while bower expects you to manually include sub-dependencies. Sometimes bower and npm are used together for front-end and back-end respectively (since each megabyte might matter in front-end).
grunt and gulp are task runners to automate everything that can be automated (i.e. compile CSS/Sass, optimize images, make a bundle and minify/transpile it).
grunt vs. gulp (is like maven vs. gradle or configuration vs. code). Grunt is based on configuring separate independent tasks, each task opens/handles/closes file. Gulp requires less amount of code and is based on Node streams, which allows it to build pipe chains (w/o reopening the same file) and makes it faster.
webpack (webpack-dev-server) - for me it's a task runner with hot reloading of changes which allows you to forget about all JS/CSS watchers.
npm/bower + plugins may replace task runners. Their abilities often intersect so there are different implications if you need to use gulp/grunt over npm + plugins. But task runners are definitely better for complex tasks (e.g. "on each build create bundle, transpile from ES6 to ES5, run it at all browsers emulators, make screenshots and deploy to dropbox through ftp").
browserify allows packaging node modules for browsers. browserify vs node's require is actually https://addyosmani.com/writing-modular-js/

Webpack and Browserify
Webpack and Browserify do pretty much the same job, which is processing your code to be used in a target environment (mainly browser, though you can target other environments like Node). Result of such processing is one or more bundles - assembled scripts suitable for targeted environment.

For example, let's say you wrote ES6 code divided into modules and want to be able to run it in a browser. If those modules are Node modules, the browser won't understand them since they exist only in the Node environment. ES6 modules also won't work in older browsers like IE11. Moreover, you might have used experimental language features (ES next proposals) that browsers don't implement yet so running such script would just throw errors. Tools like Webpack and Browserify solve these problems by translating such code to a form a browser is able to execute. On top of that, they make it possible to apply a huge variety of optimisations on those bundles.

However, Webpack and Browserify differ in many ways, Webpack offers many tools by default (e.g. code splitting), while Browserify can do this only after downloading plugins but using both leads to very similar results. It comes down to personal preference (Webpack is trendier). Btw, Webpack is not a task runner, it is just processor of your files (it processes them by so called loaders and plugins) and it can be run (among other ways) by a task runner.

Webpack Dev Server
Webpack Dev Server provides a similar solution to Browsersync - a development server where you can deploy your app rapidly as you are working on it, and verify your development progress immediately, with the dev server automatically refreshing the browser on code changes or even propagating changed code to browser without reloading with so called hot module replacement.

Task runners vs NPM scripts
I've been using Gulp for its conciseness and easy task writing, but have later found out I need neither Gulp nor Grunt at all. Everything I have ever needed could have been done using NPM scripts to run 3rd-party tools through their API. Choosing between Gulp, Grunt or NPM scripts depends on taste and experience of your team.

While tasks in Gulp or Grunt are easy to read even for people not so familiar with JS, it is yet another tool to require and learn and I personally prefer to narrow my dependencies and make things simple. On the other hand, replacing these tasks with the combination of NPM scripts and (propably JS) scripts which run those 3rd party tools (eg. Node script configuring and running rimraf for cleaning purposes) might be more challenging. But in the majority of cases, those three are equal in terms of their results.

Examples
As for the examples, I suggest you have a look at this React starter project, which shows you a nice combination of NPM and JS scripts covering the whole build and deploy process. You can find those NPM scripts in package.json in the root folder, in a property named scripts. There you will mostly encounter commands like babel-node tools/run start. Babel-node is a CLI tool (not meant for production use), which at first compiles ES6 file tools/run (run.js file located in tools) - basically a runner utility. This runner takes a function as an argument and executes it, which in this case is start - another utility (start.js) responsible for bundling source files (both client and server) and starting the application and development server (the dev server will be probably either Webpack Dev Server or Browsersync).

Speaking more precisely, start.js creates both client and server side bundles, starts an express server and after a successful launch initializes Browser-sync, which at the time of writing looked like this (please refer to react starter project for the newest code).

const bs = Browsersync.create();  
bs.init({
      ...(DEBUG ? {} : { notify: false, ui: false }),

      proxy: {
        target: host,
        middleware: [wpMiddleware, ...hotMiddlewares],
      },

      // no need to watch '*.js' here, webpack will take care of it for us,
      // including full page reloads if HMR won't work
      files: ['build/content/**/*.*'],
}, resolve)
The important part is proxy.target, where they set server address they want to proxy, which could be http://localhost:3000, and Browsersync starts a server listening on http://localhost:3001, where the generated assets are served with automatic change detection and hot module replacement. As you can see, there is another configuration property files with individual files or patterns Browser-sync watches for changes and reloads the browser if some occur, but as the comment says, Webpack takes care of watching js sources by itself with HMR, so they cooperate there.

Now I don't have any equivalent example of such Grunt or Gulp configuration, but with Gulp (and somewhat similarly with Grunt) you would write individual tasks in gulpfile.js like

gulp.task('bundle', function() {
  // bundling source files with some gulp plugins like gulp-webpack maybe
});

gulp.task('start', function() {
  // starting server and stuff
});
where you would be doing essentially pretty much the same things as in the starter-kit, this time with task runner, which solves some problems for you, but presents its own issues and some difficulties during learning the usage, and as I say, the more dependencies you have, the more can go wrong. And that is the reason I like to get rid of such tools.




