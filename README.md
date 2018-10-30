# Autoprefixer [![Cult Of Martians][cult-img]][cult]

<img align="right" width="94" height="71"
     src="http://postcss.github.io/autoprefixer/logo.svg"
     title="Autoprefixer logo by Anton Lovchikov">

[PostCSS] plugin to parse CSS and add vendor prefixes to CSS rules using values
from [Can I Use]. It is [recommended] by Google and used in Twitter and Taobao.

Write your CSS rules without vendor prefixes (in fact, forget about them
entirely):

```css
::placeholder {
  color: gray;
}
```

Autoprefixer will use the data based on current browser popularity and property
support to apply prefixes for you. You can try the [interactive demo]
of Autoprefixer.

```css
::-webkit-input-placeholder {
  color: gray;
}
:-ms-input-placeholder {
  color: gray;
}
::-ms-input-placeholder {
  color: gray;
}
::placeholder {
  color: gray;
}
```

Twitter account for news and releases: [@autoprefixer].

<a href="https://evilmartians.com/?utm_source=autoprefixer">
<img src="https://evilmartians.com/badges/sponsored-by-evil-martians.svg" alt="Sponsored by Evil Martians" width="236" height="54">
</a>

[interactive demo]: http://autoprefixer.github.io/
[@autoprefixer]:    https://twitter.com/autoprefixer
[recommended]:      https://developers.google.com/web/tools/setup/setup-buildtools#dont_trip_up_with_vendor_prefixes
[Can I Use]:        http://caniuse.com/
[cult-img]:         http://cultofmartians.com/assets/badges/badge.svg
[PostCSS]:          https://github.com/postcss/postcss
[cult]:             http://cultofmartians.com/tasks/autoprefixer-grid.html



## Contents  <!-- omit in toc -->

<details>
  <summary>View table of contents</summary>

- [Browsers](#browsers)
- [FAQ](#faq)
    - [Does Autoprefixer polyfill Grid Layout for IE?](#does-autoprefixer-polyfill-grid-layout-for-ie)
    - [No prefixes in production](#no-prefixes-in-production)
    - [What is the unprefixed version of `-webkit-min-device-pixel-ratio`?](#what-is-the-unprefixed-version-of--webkit-min-device-pixel-ratio)
    - [Does it add polyfills?](#does-it-add-polyfills)
    - [Why doesn’t Autoprefixer add prefixes to `border-radius`?](#why-doesnt-autoprefixer-add-prefixes-to-border-radius)
    - [Why does Autoprefixer use unprefixed properties in `@-webkit-keyframes`?](#why-does-autoprefixer-use-unprefixed-properties-in--webkit-keyframes)
    - [How to work with legacy `-webkit-` only code?](#how-to-work-with-legacy--webkit--only-code)
    - [Does Autoprefixer add `-epub-` prefix?](#does-autoprefixer-add--epub--prefix)
    - [Why doesn’t Autoprefixer transform generic font-family `system-ui`?](#why-doesnt-autoprefixer-transform-generic-font-family-system-ui)
- [Usage](#usage)
    - [Gulp](#gulp)
    - [Webpack](#webpack)
    - [Grunt](#grunt)
    - [Other Build Tools:](#other-build-tools)
    - [Preprocessors](#preprocessors)
    - [CSS-in-JS](#css-in-js)
    - [GUI Tools](#gui-tools)
    - [CLI](#cli)
    - [JavaScript](#javascript)
    - [Text Editors and IDE](#text-editors-and-ide)
- [Warnings](#warnings)
- [Disabling](#disabling)
    - [Prefixes](#prefixes)
    - [Features](#features)
    - [Control Comments](#control-comments)
- [Options](#options)
- [Grid Autoplacement support in IE](#grid-autoplacement-support-in-ie)
    - [Beware of enabling autoplacment in already existing projects](#beware-of-enabling-autoplacment-in-already-existing-projects)
    - [Autoplacement limitations](#autoplacement-limitations)
        - [Both columns and rows must be defined](#both-columns-and-rows-must-be-defined)
        - [No manual cell placement or column/row spans allowed inside an autoplacement grid](#no-manual-cell-placement-or-columnrow-spans-allowed-inside-an-autoplacement-grid)
- [Debug](#debug)

</details>

## Browsers

Autoprefixer uses [Browserslist], so you can specify the browsers
you want to target in your project with queries like `> 5%`
(see [Best Practices]).

The best way to provide browsers is a `.browserslistrc` file in your project
root, or by adding a `browserslist` key to your `package.json`.

We recommend the use of these options over passing options to Autoprefixer so
that the config can be shared with other tools such as [babel-preset-env] and
[Stylelint].

See [Browserslist docs] for queries, browser names, config format, and defaults.

[Browserslist docs]: https://github.com/ai/browserslist#queries
[babel-preset-env]:  https://github.com/babel/babel/tree/master/packages/babel-preset-env
[Best Practices]:    https://github.com/browserslist/browserslist#best-practices
[Browserslist]:      https://github.com/ai/browserslist
[Stylelint]:         http://stylelint.io/


## FAQ

### Does Autoprefixer polyfill Grid Layout for IE?

Autoprefixer can be used to translate modern CSS Grid syntax into IE 10 and IE 11 syntax,
but this polyfill will not work in 100% of cases. This is why it is disabled by default.

First, you need to enable Grid prefixes by using either the `grid: "autoplace"` option or the `/* autoprefixer grid: autoplace */` control comment.

Second, you need to test every fix with Grid in IE. It is not an enable and
forget feature, but it is still very useful.
Financial Times and Yandex use it in production.

Third, there is only very limited auto placement support. Read the [Grid Autoplacement support in IE](#grid-autoplacement-support-in-ie) section for more details.

Fourth, if you are not using the autoplacment feature, the best way to use Autoprefixer is by using  `grid-template` or `grid-template-areas`.

```css
.page {
    display: grid;
    grid-gap: 33px;
    grid-template:
        "head head  head" 1fr
        "nav  main  main" minmax(100px, 1fr)
        "nav  foot  foot" 2fr /
        1fr   100px 1fr;
}
.page__head {
    grid-area: head;
}
.page__nav {
    grid-area: nav;
}
.page__main {
    grid-area: main;
}
.page__footer {
    grid-area: foot;
}
```

See also:

* [The guide about Grids in IE and Autoprefixer].
* [`postcss-gap-properties`] to use new `gap` property
  instead of old `grid-gap`.
* [`postcss-grid-kiss`] has alternate “everything in one property” syntax,
  which makes using Autoprefixer’s Grid translations safer.

[The guide about Grids in IE and Autoprefixer]: https://css-tricks.com/css-grid-in-ie-css-grid-and-the-new-autoprefixer/
[`postcss-gap-properties`]:                     https://github.com/jonathantneal/postcss-gap-properties
[`postcss-grid-kiss`]:                          https://github.com/sylvainpolletvillard/postcss-grid-kiss

### No prefixes in production

Many other tools contain Autoprefixer. For example, webpack uses Autoprefixer
to minify CSS by cleaning unnecessary prefixes.

If you pass your browsers to Autoprefixer using its `browsers` option, the other
tools will use their own config, leading webpack to remove the prefixes that
the first Autoprefixer added.

To avoid this, ensure you use either the [browserslist config file] or
`browsers` key in your `package.json`, so that all tools (Autoprefixer,
cssnano, doiuse, cssnext, etc) use the same browsers list.

[browserslist config file]: https://github.com/ai/browserslist#config-file


### What is the unprefixed version of `-webkit-min-device-pixel-ratio`?

```css
@media (min-resolution: 2dppx) {
    .image {
        background-image: url(image@2x.png);
    }
}
```

Will be compiled to:

```css
@media (-webkit-min-device-pixel-ratio: 2),
       (-o-min-device-pixel-ratio: 2/1),
       (min-resolution: 2dppx) {
    .image {
        background-image: url(image@2x.png);
    }
}
```


### Does it add polyfills?

No. Autoprefixer only adds prefixes.

Most new CSS features will require client side JavaScript to handle a new
behavior correctly.

Depending on what you consider to be a “polyfill”, you can take a look at some
other tools and libraries. If you are just looking for syntax sugar,
you might take a look at:

- [postcss-preset-env] is a plugins preset with polyfills and Autoprefixer
  to write future CSS today.
- [Oldie], a PostCSS plugin that handles some IE hacks (opacity, rgba, etc).
- [postcss-flexbugs-fixes], a PostCSS plugin to fix flexbox issues.

[postcss-flexbugs-fixes]: https://github.com/luisrudge/postcss-flexbugs-fixes
[postcss-preset-env]:     https://github.com/jonathantneal/postcss-preset-env
[Oldie]:                  https://github.com/jonathantneal/oldie


### Why doesn’t Autoprefixer add prefixes to `border-radius`?

Developers are often surprised by how few prefixes are required today.
If Autoprefixer doesn’t add prefixes to your CSS, check if they’re still
required on [Can I Use].

[Can I Use]: http://caniuse.com/


### Why does Autoprefixer use unprefixed properties in `@-webkit-keyframes`?

Browser teams can remove some prefixes before others, so we try to use all
combinations of prefixed/unprefixed values.


### How to work with legacy `-webkit-` only code?

Autoprefixer needs unprefixed property to add prefixes. So if you only
wrote `-webkit-gradient` without W3C’s `gradient`,
Autoprefixer will not add other prefixes.

But [PostCSS] has plugins to convert CSS to unprefixed state.
Use [postcss-unprefix] before Autoprefixer.

[postcss-unprefix]: https://github.com/gucong3000/postcss-unprefix


### Does Autoprefixer add `-epub-` prefix?

No, Autoprefixer works only with browsers prefixes from Can I Use.
But you can use [postcss-epub]
for prefixing ePub3 properties.

[postcss-epub]: https://github.com/Rycochet/postcss-epub


### Why doesn’t Autoprefixer transform generic font-family `system-ui`?

`system-ui` is technically not a prefix and the transformation is not
future-proof. You can use [postcss-font-family-system-ui] to transform
`system-ui` to a practical font-family list.

[postcss-font-family-system-ui]: https://github.com/JLHwung/postcss-font-family-system-ui


## Usage

### Gulp

In Gulp you can use [gulp-postcss] with `autoprefixer` npm package.

```js
gulp.task('autoprefixer', function () {
    var postcss      = require('gulp-postcss');
    var sourcemaps   = require('gulp-sourcemaps');
    var autoprefixer = require('autoprefixer');

    return gulp.src('./src/*.css')
        .pipe(sourcemaps.init())
        .pipe(postcss([ autoprefixer() ]))
        .pipe(sourcemaps.write('.'))
        .pipe(gulp.dest('./dest'));
});
```

With `gulp-postcss` you also can combine Autoprefixer
with [other PostCSS plugins].

[gulp-postcss]:          https://github.com/postcss/gulp-postcss
[other PostCSS plugins]: https://github.com/postcss/postcss#plugins


### Webpack

In [webpack] you can use [postcss-loader] with `autoprefixer`
and [other PostCSS plugins].

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.css$/,
                use: ["style-loader", "css-loader", "postcss-loader"]
            }
        ]
    }
}
```

And create a `postcss.config.js` with:

```js
module.exports = {
  plugins: [
    require('autoprefixer')
  ]
}
```

[other PostCSS plugins]: https://github.com/postcss/postcss#plugins
[postcss-loader]:        https://github.com/postcss/postcss-loader
[webpack]:               http://webpack.github.io/


### Grunt

In Grunt you can use [grunt-postcss] with `autoprefixer` npm package.

```js
module.exports = function(grunt) {
    grunt.loadNpmTasks('grunt-postcss');

    grunt.initConfig({
        postcss: {
            options: {
                map: true,
                processors: [
                    require('autoprefixer')
                ]
            },
            dist: {
                src: 'css/*.css'
            }
        }
    });

    grunt.registerTask('default', ['postcss:dist']);
};
```

With `grunt-postcss` you also can combine Autoprefixer
with [other PostCSS plugins].

[other PostCSS plugins]: https://github.com/postcss/postcss#plugins
[grunt-postcss]:         https://github.com/nDmitry/grunt-postcss


### Other Build Tools:

* **Ruby on Rails**: [autoprefixer-rails]
* **Neutrino**: [neutrino-middleware-postcss]
* **Jekyll**: add `autoprefixer-rails` and `jekyll-assets` to `Gemfile`
* **Brunch**: [postcss-brunch]
* **Broccoli**: [broccoli-postcss]
* **Middleman**: [middleman-autoprefixer]
* **Mincer**: add `autoprefixer` npm package and enable it:
  `environment.enable('autoprefixer')`

[neutrino-middleware-postcss]: https://www.npmjs.com/package/neutrino-middleware-postcss
[middleman-autoprefixer]:      https://github.com/middleman/middleman-autoprefixer
[autoprefixer-rails]:          https://github.com/ai/autoprefixer-rails
[broccoli-postcss]:            https://github.com/jeffjewiss/broccoli-postcss
[postcss-brunch]:              https://github.com/iamvdo/postcss-brunch


### Preprocessors

* **Less**: [less-plugin-autoprefix]
* **Stylus**: [autoprefixer-stylus]
* **Compass**: [autoprefixer-rails#compass]

[less-plugin-autoprefix]: https://github.com/less/less-plugin-autoprefix
[autoprefixer-stylus]:    https://github.com/jenius/autoprefixer-stylus
[autoprefixer-rails#compass]:     https://github.com/ai/autoprefixer-rails#compass


### CSS-in-JS

There is [postcss-js] to use Autoprefixer in React Inline Styles, [Free Style],
Radium and other CSS-in-JS solutions.

```js
let prefixer = postcssJs.sync([ autoprefixer ]);
let style = prefixer({
    display: 'flex'
});
```

[postcss-js]: https://github.com/postcss/postcss-js
[Free Style]: https://github.com/blakeembrey/free-style


### GUI Tools

* [CodeKit](https://codekitapp.com/help/autoprefixer/)
* [Prepros](https://prepros.io)


### CLI

You can use the [postcss-cli] to run Autoprefixer from CLI:

```sh
npm install postcss-cli autoprefixer
npx postcss *.css --use autoprefixer -d build/
```

See `postcss -h` for help.

[postcss-cli]: https://github.com/postcss/postcss-cli


### JavaScript

You can use Autoprefixer with [PostCSS] in your Node.js application
or if you want to develop an Autoprefixer plugin for a new environment.

```js
var autoprefixer = require('autoprefixer');
var postcss      = require('postcss');

postcss([ autoprefixer ]).process(css).then(function (result) {
    result.warnings().forEach(function (warn) {
        console.warn(warn.toString());
    });
    console.log(result.css);
});
```

There is also a [standalone build] for the browser or for a non-Node.js runtime.

You can use [html-autoprefixer] to process HTML with inlined CSS.

[html-autoprefixer]: https://github.com/RebelMail/html-autoprefixer
[standalone build]:  https://raw.github.com/ai/autoprefixer-rails/master/vendor/autoprefixer.js
[PostCSS]:           https://github.com/postcss/postcss


### Text Editors and IDE

Autoprefixer should be used in assets build tools. Text editor plugins are not
a good solution, because prefixes decrease code readability and you will need
to change values in all prefixed properties.

I recommend you to learn how to use build tools like [Gulp].
They work much better and will open you a whole new world of useful plugins
and automation.

If you can’t move to a build tool, you can use text editor plugins:

* [Sublime Text](https://github.com/sindresorhus/sublime-autoprefixer)
* [Brackets](https://github.com/mikaeljorhult/brackets-autoprefixer)
* [Atom Editor](https://github.com/sindresorhus/atom-autoprefixer)
* [Visual Studio](https://github.com/madskristensen/WebCompiler)

[Gulp]:  http://gulpjs.com/


## Warnings

Autoprefixer uses the [PostCSS warning API] to warn about really important
problems in your CSS:

* Old direction syntax in gradients.
* Old unprefixed `display: box` instead of `display: flex`
  by latest specification version.

You can get warnings from `result.warnings()`:

```js
result.warnings().forEach(function (warn) {
    console.warn(warn.toString());
});
```

Every Autoprefixer runner should display these warnings.

[PostCSS warning API]: https://github.com/postcss/postcss/blob/master/docs/api.md#warning-class


## Disabling

### Prefixes

Autoprefixer was designed to have no interface – it just works.
If you need some browser specific hack just write a prefixed property
after the unprefixed one.

```css
a {
    transform: scale(0.5);
    -moz-transform: scale(0.6);
}
```

If some prefixes were generated incorrectly, please create an [issue on GitHub].

[issue on GitHub]: https://github.com/postcss/autoprefixer/issues


### Features

You can use these plugin options to disable some of Autoprefixer’s features.

* `grid: "autoplace"` will enable `-ms-` prefixes for Grid Layout including some
  [limited autoplacment support](#grid-autoplacement-support-in-ie).
* `supports: false` will disable `@supports` parameters prefixing.
* `flexbox: false` will disable flexbox properties prefixing.
  Or `flexbox: "no-2009"` will add prefixes only for final and IE
  versions of specification.
* `remove: false` will disable cleaning outdated prefixes.

You should set them inside the plugin like so:

```js
autoprefixer({ grid: "autoplace" });
```


### Control Comments

If you do not need Autoprefixer in some part of your CSS,
you can use control comments to disable Autoprefixer.

```css
.a {
    transition: 1s; /* will be prefixed */
}

.b {
    /* autoprefixer: off */
    transition: 1s; /* will not be prefixed */
}

.c {
    /* autoprefixer: ignore next */
    transition: 1s; /* will not be prefixed */
    mask: url(image.png); /* will be prefixed */
}
```

There are three types of control comments:

* `/* autoprefixer: (on|off) */`: enable/disable all Autoprefixer translations for the
  whole block both *before* and *after* the comment.
* `/* autoprefixer: ignore next */`: disable Autoprefixer only for the next property
  or next rule selector or at-rule parameters (but not rule/at‑rule body).
* `/* autoprefixer grid: (autoplace|no-autoplace|off) */`: control how Autoprefixer handles
  grid translations for the whole block:
    * `autoplace`: enable grid translations with autoplacement support.
    * `no-autoplace`: enable grid translations with autoplacement support *disabled*.
      (alias for deprecated value `on`)
    * `off`: disable all grid translations.

You can also use comments recursively:

```css
/* autoprefixer: off */
@supports (transition: all) {
    /* autoprefixer: on */
    a {
        /* autoprefixer: off */
    }
}
```

Note that comments that disable the whole block should not be featured in the same
block twice:

```css
/* How not to use block level control comments */

.do-not-do-this {
    /* autoprefixer: off */
    transition: 1s;
    /* autoprefixer: on */
    transform: rotate(20deg);
}
```


## Options

Function `autoprefixer(options)` returns a new PostCSS plugin.
See [PostCSS API] for plugin usage documentation.

```js
autoprefixer({ cascade: false })
```

Available options are:

* `env` (string): environment for Browserslist.
* `cascade` (boolean): should Autoprefixer use Visual Cascade,
  if CSS is uncompressed. Default: `true`
* `add` (boolean): should Autoprefixer add prefixes. Default is `true`.
* `remove` (boolean): should Autoprefixer [remove outdated] prefixes.
  Default is `true`.
* `supports` (boolean): should Autoprefixer add prefixes for `@supports`
  parameters. Default is `true`.
* `flexbox` (boolean|string): should Autoprefixer add prefixes for flexbox
  properties. With `"no-2009"` value Autoprefixer will add prefixes only
  for final and IE versions of specification. Default is `true`.
* `grid` (boolean|"autoplace"): should Autoprefixer add IE prefixes for Grid Layout
  properties?
  * `false` (default): prevent Autoprefixer from outputting CSS Grid translations.
  * `"autoplace"`: enable Autoprefixer grid translations and *include* autoplacement
    support. You can also use `/* autoprefixer grid: autoplace */` in your CSS.
  * `"no-autoplace"`: enable Autoprefixer grid translations but *exclude* autoplacement
    support. You can also use `/* autoprefixer grid: no-autoplace */` in your CSS.
    (alias for the deprecated `true` value)
* `stats` (object): custom [usage statistics] for `> 10% in my stats`
  browsers query.
* `browsers` (array): list of queries for target browsers. Try to not use it.
  The best practice is to use `.browserslistrc` config
  or `browserslist` key in `package.json` to share target browsers
  with Babel, ESLint and Stylelint. See [Browserslist docs]
  for available queries and default value.
* `ignoreUnknownVersions` (boolean): do not raise error on unknown browser
  version in Browserslist config or `browsers` option. Default is `false`.

Plugin object has `info()` method for debugging purpose.

You can use PostCSS processor to process several CSS files
to increase performance.

[usage statistics]: https://github.com/ai/browserslist#custom-usage-data
[PostCSS API]:      http://api.postcss.org

## Grid Autoplacement support in IE

If the `grid` option is set to `"autoplace"`, limited autoplacement support is added to Autoprefixers grid translations. You can also use the `/* autoprefixer grid: autoplace */` control comment to enable autoplacement

Autoprefixer will only autoplace grid cells if both `grid-template-rows` and `grid-template-columns` has been set. If `grid-template` or `grid-template-areas` has been set, Autoprefixer will use area based cell placement instead.

Autoprefixer supports autoplacement by using `nth-child` CSS selectors. It creates [number of columns] x [number of rows] `nth-child` selectors. For this reason Autoplacement is only supported within the explicit grid.

```css
/* Input CSS */

/* autoprefixer grid: autoplace */

.autoplacement-example {
    display: grid;
    grid-template-columns: 1fr 1fr;
    grid-template-rows: auto auto;
    grid-gap: 20px;
}
```

```css
/* Output CSS */

/* autoprefixer grid: autoplace */

.autoplacement-example {
  display: -ms-grid;
  display: grid;
  -ms-grid-columns: 1fr 20px 1fr;
  grid-template-columns: 1fr 1fr;
  -ms-grid-rows: auto 20px auto;
  grid-template-rows: auto auto;
  grid-gap: 20px;
}

.autoplacement-example > *:nth-child(1) {
  -ms-grid-row: 1;
  -ms-grid-column: 1;
}

.autoplacement-example > *:nth-child(2) {
  -ms-grid-row: 1;
  -ms-grid-column: 3;
}

.autoplacement-example > *:nth-child(3) {
  -ms-grid-row: 3;
  -ms-grid-column: 1;
}

.autoplacement-example > *:nth-child(4) {
  -ms-grid-row: 3;
  -ms-grid-column: 3;
}
```

### Beware of enabling autoplacment in already existing projects

Be careful about enabling autoplacement in any already established projects that have
previously not used Autoprefixer's grid autoplacement feature before.

The following CSS will not work as expected with the autoplacement feature enabled:

```css
/* Unsafe CSS when Autoplacement is enabled */

.grid-cell {
    grid-column: 2;
    grid-row: 2;
}

.grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(3, 1fr);
}
```

Swapping the rules around so that the grid template styles are declared first will fix
the issue:

```css
/* Place grid template styles before the grid cell styles to be safe */

.grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(3, 1fr);
}

.grid-cell {
    grid-column: 2;
    grid-row: 2;
}
```

So as long as the grid cell styles are always declared after the grid-template styles,
it should be safe to enable autoplacment in old projects.


### Autoplacement limitations

#### Both columns and rows must be defined

Autoplacement only works inside the explicit grid. The columns and rows need to be defined
so that Autoprefixer knows how many `nth-child` selectors to generate.

```css
.not-allowed {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
}

.is-allowed {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(10, auto);
}
```

#### No manual cell placement or column/row spans allowed inside an autoplacement grid

Elements must not be manually placed or given column/row spans inside an autoplacement
grid. Only the most basic of autoplacement grids are supported. Grid cells can still be
placed manually outside the the explicit grid though. Support for manually placing
individual grid cells inside an explicit autoplacement grid is planned for a
future release.

```css
.autoplacement-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(3, auto);
}

/*
    grid cells placed inside the explicit grid
    will break the layout in IE
*/
.not-permitted-grid-cell {
    grid-column: 1;
    grid-row: 1;
}

/*
    grid cells placed outside the
    explicit grid will work in IE
*/
.permitted-grid-cell {
    grid-column: 1 / span 2;
    grid-row: 4;
}
```

If manual cell placement is required, we recommend using `grid-template` or
`grid-template-areas` instead:

```css
.page {
    display: grid;
    grid-gap: 30px;
    grid-template:
        "head head"
        "nav  main" minmax(100px, 1fr)
        "foot foot" /
        200px 1fr;
}
.page__head {
    grid-area: head;
}
.page__nav {
    grid-area: nav;
}
.page__main {
    grid-area: main;
}
.page__footer {
    grid-area: foot;
}
```


## Debug

Run `npx autoprefixer --info` in your project directory to check
which browsers are selected and which properties will be prefixed:

```
$ npx autoprefixer --info
Browsers:
  Edge: 16

These browsers account for 0.26% of all users globally

At-Rules:
  @viewport: ms

Selectors:
  ::placeholder: ms

Properties:
  appearance: webkit
  flow-from: ms
  flow-into: ms
  hyphens: ms
  overscroll-behavior: ms
  region-fragment: ms
  scroll-snap-coordinate: ms
  scroll-snap-destination: ms
  scroll-snap-points-x: ms
  scroll-snap-points-y: ms
  scroll-snap-type: ms
  text-size-adjust: ms
  text-spacing: ms
  user-select: ms
```

JS API is also available:

```js
var info = autoprefixer().info();
console.log(info);
```
