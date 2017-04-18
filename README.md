# A PHP port of the YUI CSS compressor

[![Latest Stable Version](https://poser.pugx.org/tubalmartin/cssmin/v/stable)](https://packagist.org/packages/tubalmartin/cssmin) [![Total Downloads](https://poser.pugx.org/tubalmartin/cssmin/downloads)](https://packagist.org/packages/tubalmartin/cssmin) [![Daily Downloads](https://poser.pugx.org/tubalmartin/cssmin/d/daily)](https://packagist.org/packages/tubalmartin/cssmin) [![License](https://poser.pugx.org/tubalmartin/cssmin/license)](https://packagist.org/packages/tubalmartin/cssmin)

This port is based on version 2.4.8 (Jun 12, 2013) of the [YUI compressor](https://github.com/yui/yuicompressor).

**Table of Contents**

1.  [Installation, requirements & usage](#installuse)
2.  [YUI compressor fixed bugs](#bugsfixed)
3.  [Unit Tests](#unittests)
4.  [API Reference](#api)
5.  [Who uses this?](#whousesit)
6.  [Changelog](#changelog)

<a name="installuse"></a>

## 1. Installation, requirements & usage

### Installation

Use [Composer](http://getcomposer.org/) to include the library into your project:

    $ composer.phar require tubalmartin/cssmin

Require Composer's autoloader file:

```php
<?php

require './vendor/autoload.php';

use tubalmartin\CssMin\Minifier as CSSmin;

// Use it!
$compressor = new CSSmin();
```

### Requirements

* PHP 5.3.2 or newer with PCRE extension.

### Usage

Here's an example that covers all minifier options:

```php
<?php

// Autoload libraries
require './vendor/autoload.php';

use tubalmartin\CssMin\Minifier as CSSmin;

// Extract the CSS code you want to compress from your CSS files
$input_css1 = file_get_contents('test1.css');
$input_css2 = file_get_contents('test2.css');

// Create a new CSSmin object.
// By default CSSmin will try to raise PHP settings.
// If you don't want CSSmin to raise the PHP settings pass FALSE to
// the constructor i.e. $compressor = new CSSmin(false);
$compressor = new CSSmin();

// Override chunk length
// Default is 5000 chars. Lower it if you are having issues with PCRE.
// Minimum length is 100 chars.
$compressor->setChunkLength(1000);

// Override any PHP configuration options before calling run() (optional)
$compressor->setMemoryLimit('256M');
$compressor->setMaxExecutionTime(120);

// Compress the CSS code in 1 long line and store the result in a variable
$output_css1 = $compressor->run($input_css1);

// You can change any PHP configuration option between run() calls
// and those will be applied for that run
$compressor->setPcreBacktrackLimit(3000000);
$compressor->setPcreRecursionLimit(150000);

// Compress the CSS code splitting lines after a specific column (2000) and
// store the result in a variable
$output_css2 = $compressor->run($input_css2, 2000);

// Do whatever you need with the compressed CSS code
echo $output_css1 . $output_css2;
```

**Need a GUI?**

We've made a simple web based GUI to use the compressor, it's in the `gui` folder.

GUI features:

* Optional on-the-fly LESS compilation before compression with error reporting included.
* Absolute control of the library.

How to use the GUI:

* You need a server with PHP 5.3.2+ installed.
* Download the repository and upload it to a folder in your server.
* Run `php composer.phar install` in project's root to install dependencies.
* Open your favourite browser and enter the URL to the `/gui` folder.

<a name="bugsfixed"></a>

## 2. FIXED BUGS still present in the original YUI compressor

* Only one `@charset` at-rule per file and pushed at the beginning of the file. YUI compressor does not remove @charset at-rules if single quotes are used `@charset 'utf-8'`.
* Safer/improved comment removal. YUI compressor would ruin part of the output if the `*` selector is used right after a comment: `a{/* comment 1 */*width:auto;}/* comment 2 */* html .b{height:100px}`. See issues [#2528130](http://yuilibrary.com/projects/yuicompressor/ticket/2528130), [#2528118](http://yuilibrary.com/projects/yuicompressor/ticket/2528118) & [this topic](http://yuilibrary.com/forum/viewtopic.php?f=94&t=9606)
* `background: none;` is not compressed to `background:0;` anymore. See issue [#2528127](http://yuilibrary.com/projects/yuicompressor/ticket/2528127).
* `text-shadow: 0 0 0;` is not compressed to `text-shadow:0;` anymore. See issue [#2528142](http://yuilibrary.com/projects/yuicompressor/ticket/2528142)
* Trailing `;` is not removed anymore if the last property is prefixed with a `*` (lte IE7 hack). See issue [#2528146](http://yuilibrary.com/projects/yuicompressor/ticket/2528146)
* Newlines before and/or after a preserved comment `/*!` are not removed (we leave just 1 newline). YUI removes all newlines making it really hard to spot an important comment.
* Spaces surrounding the `+` operator in `calc()` calculations are not removed. YUI removes them and that is wrong.
* Fix for issue [#2528093](http://yuilibrary.com/projects/yuicompressor/ticket/2528093).
* Fixes for `!important` related issues.
* Fixes @keyframes 0% step bug.
* Some units should not be removed when the value is 0, such as `0s` or `0ms`.
* Fixes replacing 0 length values in selectors such as `.span0px { ... }`.
* And some more...

<a name="unittests"></a>

## 3. Unit Tests

Unit tests from YUI have been modified to fit this port.

How to run the test suite:

* You need a server with PHP 5.3.2+ installed.
* Download the repository and upload it to a folder in your server.
* Run `php composer.phar install` in project's root to install dependencies.
* Open your favourite browser and enter the URL to the file `tests/run.php`.

<a name="api"></a>

## 4. API Reference

### __construct([ bool *$raisePhpLimits* ])

**Description**

Class constructor, creates a new CSSmin object.

**Parameters**

*raisePhpLimits*

If TRUE, CSSmin will try to raise the values of some php configuration options.
Set to FALSE to keep the values of your php configuration options.
Defaults to TRUE.

### run(string *$css* [, int *$linebreakPos* ])

**Description**

Minifies a string of uncompressed CSS code.
`run()` may be called multiple times on a single CSSmin instance.

**Parameters**

*css*

A string of uncompressed CSS code.
Defaults to an empty string `''`.

*linebreakPos*

Some source control tools don't like it when files containing lines longer than, say 8000 characters, are checked in.
The linebreak option is used in that case to split long lines after a specific column.
Defaults to FALSE (1 long line).

**Return Values**

A string of compressed CSS code or an empty string if no string is passed.

### setChunkLength(int *$length*)

**Description**

Sets the the approximate number of characters to use when splitting a string in chunks.

CSSmin default value: `5000`

**Parameters**

*length*

Values & notes: Minimum value supported: `100`

### setMemoryLimit(mixed *$limit*)

**Description**

Sets the `memory_limit` configuration option for this script

CSSmin default value: `128M`

**Parameters**

*limit*

Values & notes: [memory_limit documentation](http://php.net/manual/en/ini.core.php#ini.memory-limit)

### setMaxExecutionTime(int *$seconds*)

**Description**

Sets the `max_execution_time` configuration option for this script

CSSmin default value: `60`

**Parameters**

*seconds*

Values & notes: [max_execution_time documentation](http://php.net/manual/en/info.configuration.php#ini.max-execution-time)

### setPcreBacktrackLimit(int *$limit*)

**Description**

Sets the `pcre.backtrack_limit` configuration option for this script

CSSmin default value: `1000000`

**Parameters**

*limit*

Values & notes: [pcre.backtrack_limit documentation](http://php.net/manual/en/pcre.configuration.php#ini.pcre.backtrack-limit)

### setPcreRecursionLimit(int *$limit*)

**Description**

Sets the `pcre.recursion_limit` configuration option for this script.

CSSmin default value: `500000`

**Parameters**

*limit*

Values & notes: [pcre.recursion_limit documentation](http://php.net/manual/en/pcre.configuration.php#ini.pcre.recursion-limit)


<a name="whousesit"></a>

## 5. Who uses this port

* [Magento](https://magento.com/) eCommerce platforms and solutions for selling online.
* [Minify](https://github.com/mrclay/minify) Minify is an HTTP content server. It compresses sources of content (usually files), combines the result and serves it with appropriate HTTP headers.
* [Autoptimize](http://wordpress.org/plugins/autoptimize/) is a Wordpress plugin. Autoptimize speeds up your website and helps you save bandwidth by aggregating and minimizing JS, CSS and HTML.
* [IMPRESSPAGES](http://www.impresspages.org/) PHP framework with content editor.

<a name="changelog"></a>

## 6. Changelog

### v3.1.2 17 Apr 2017

* Improved compression of long named colors: now all long named colors get compressed to their shorter HEX counterpart.
* Fixes cases such as [#39](https://github.com/tubalmartin/YUI-CSS-compressor-PHP-port/issues/39)
* Huge performance improvement after code profiling. See table below for results when running the whole test suite:

PHP version used: 5.3.29

| chunkLength | v3.1.1 | v3.1.2 |
| --- | --- | --- |
| 100 | 6.8s | 2.6s |
| 1000 | 5.3s | 2s |
| 2000 | 5.2s | 1.95s |
| 5000 | 5.1s | 1.9s |

PHP version used: 7.0.8

| chunkLength | v3.1.1 | v3.1.2 |
| --- | --- | --- |
| 100 | 2s | 0.72s |
| 1000 | 1s | 0.37s |
| 2000 | 0.8s | 0.33s |
| 5000 | 0.7s | 0.3s |


### v3.1.1 11 Apr 2017

* Regexes improved.
* Small performance improvements.
* Blocks such as `@media` blocks with empty rules are removed too.
* Quoted unquotable attribute selectors get unquoted now i.e. from `col[class*="col-"]` to `col[class*=col-]`. Covers most common cases. Safe approach.

### v3.1.0 9 Apr 2017

* Code deeply analyzed. Some areas rewritten from the ground up with maximum performance in mind. No change in compressor behavior.
* Fixed some hidden bugs discovered along the way that affected performance negatively.
* IE5/Mac comment hack removed from minifier logic. Those comments will no longer be preserved.
* The table below displays the performance optimization done in this version in comparison with the previous one running the whole test suite:

PHP version used: 5.3.29

| chunkLength | v3.0.0 | v3.1.0 |
| --- | --- | --- |
| 100 | 38s | 6.9s |
| 1000 | 8.5s | 5.4s |
| 2000 | 7.3s | 5.3s |
| 5000 | 5.8s | 5.2s |

PHP version used: 7.0.8

| chunkLength | v3.0.0 | v3.1.0 |
| --- | --- | --- |
| 100 | 22.8s | 2.1s |
| 1000 | 2.9s | 1.1s |
| 2000 | 2s | 0.9s |
| 5000 | 1.3s | 0.8s |

### v3.0.0 4 Apr 2017

* New API compliant with PSR-1, PSR-2 & PSR-4. PHP 5.3.2+ required. I think it was time!
* Many tests added, strengthened and fixed. Big, real life, stylesheets included such as Bootstrap or Foundation.
* Fixed some critical and minor issues, such as:
   * Chunking system breaking some stylesheets (broken at rules block) or leaving some bits off.
   * Backreferences in replacement strings breaking stylesheets.
   * [#23](https://github.com/tubalmartin/YUI-CSS-compressor-PHP-port/issues/23)
   * Others...
* Color compression improved. Now all named colors are supported i.e. from `white` to `#fff`.
* Shortening zero values is back but in a safe manner, shortening values assigned to "safe" properties only i.e. from `margin: 1px 0.0em 0rem 0%` to `margin:1px 0 0`. Check the code to see the list of "safe" properties.
* `padding` and `margin` properties are shortened to the bare minimum i.e. from `margin: 3px 2.1em 3px 2.1em` => `margin:3px 2.1em`
* Upgrading to v3 is strongly recommended for users enjoying PHP 5.3.2+. 

### v2.4.8-p10 4 Apr 2017

* This is the last v2 release. v3 onwards will only support PHP 5.3.2+.
* This patch has all improvements and fixes v3.0.0 has. See v3.0.0 notes for further info (no API change in this version of course).
* Updating to this patch is strongly recommended for users stuck with PHP versions lower than PHP 5.3.

### v2.4.8-p9 28 Mar 2017

* Rolling back property declaration with scalar expressions (>= PHP 5.6) introduced in v2.4.8-p8 to support PHP 5.0. No change in compressor behavior.

### v2.4.8-p8 27 Mar 2017

* Fixed issue [#18](https://github.com/tubalmartin/YUI-CSS-compressor-PHP-port/pull/18)
* Added `set_chunk_length` method.
* `bold` & `normal` values get compressed to `700` & `400` respectively for `font-weight` property.
* GUI updated.
* FineDiff library loaded through Composer.
* README updated.

### v2.4.8-p7 26 Mar 2017

* Fixed many issues [#20](https://github.com/tubalmartin/YUI-CSS-compressor-PHP-port/issues/20), [#22](https://github.com/tubalmartin/YUI-CSS-compressor-PHP-port/issues/22), [#24](https://github.com/tubalmartin/YUI-CSS-compressor-PHP-port/issues/24), [#25](https://github.com/tubalmartin/YUI-CSS-compressor-PHP-port/issues/25), [#26](https://github.com/tubalmartin/YUI-CSS-compressor-PHP-port/issues/26) reported by contributors and others that I'm sure haven't been reported, at least yet. Sorry for the long delay guys.
* This release is all about stability and reliability and as such I've had to take some controversial decisions such as:
   * Not minifying `none` property value to `0` because in some subtle scenarios the resulting output may render some styles differently.
   * Not removing units from zero length values because in many cases the output will break the intended behavior. Patching every single case after someone finds a new breaking case is not good IMHO taking into account CSS is a live spec and browsers differ in some cases.
* Hope you agree with me removing those conflicting parts. Enjoy this release :)
   
### v2.4.8-p6 21 Mar 2017

* Fixed PHP CLI issues. See [#36](https://github.com/tubalmartin/YUI-CSS-compressor-PHP-port/pull/36)

### v2.4.8-p5 27 Feb 2017

* Fixed PHP 7 issues.

### v2.4.8-p4 22 Sep 2014

* Composer support. The package is [tubalmartin/cssmin](https://packagist.org/packages/tubalmartin/cssmin)
* Fixed issue [#17]

### v2.4.8-p3 26 Apr 2014

* Fixed all reported bugs: See issues [#11], [#13] (first case only) and [#14].
* LESS compiler upgraded to version 1.7.0

### v2.4.8-p2 13 Nov 2013

* Chunk length reduced to 5000 chars (previously 25.000 chars) in an effort to avoid PCRE backtrack limits (needs feedback).
* Improvements for the `@keyframes 0%` step bug. Tests improved.
* Fix IE7 issue on matrix filters which browser accept whitespaces between Matrix parameters
* LESS compiler upgraded to version 1.4.2

### v2.4.8-p1 8 Aug 2013

* Fix for the `@keyframes 0%` step bug. Tests added.
* LESS compiler upgraded to version 1.4.1

[#11]: https://github.com/tubalmartin/YUI-CSS-compressor-PHP-port/issues/11
[#13]: https://github.com/tubalmartin/YUI-CSS-compressor-PHP-port/issues/13
[#14]: https://github.com/tubalmartin/YUI-CSS-compressor-PHP-port/issues/14
[#17]: https://github.com/tubalmartin/YUI-CSS-compressor-PHP-port/issues/17
