<img alt="an egg slicer" src="http://i.imgur.com/GmQNEMG.jpg" align="right">

Breakpoint Slicer
=================

**Slice media queries with no effort**

Along with Respond To, Breakpoint Slicer is an alternative syntax for [Breakpoint][1]. It offers a powerful yet very simple way to slice your media queries.

Just list your breakpoints and Breakpoint Slicer will magically turn them into slices. You can address the with their sequence numbers:

    $slicer-breakpoints: 0 400px 600px 800px 1050px;
    // Slice numbers:    |  1 |  2 |  3  |  4  |  5  →
    
    // Set a media query
    @include at(3) {
      // Styles inside here will be applied when
      // browser window width is somewhere between 600px and 800px
    }


Concept
-------

Imagine the breakpoints of your site on a scale:

     Breakpoint:   0                 400px     600px     800px       1050px
                   ├───────────────────┼─────────┼─────────┼───────────┼─────────>

See those spans between breakpoints? Let's name them "slices".

Breakpoint Slicer numbers the slices sequentially:

     Breakpoint:   0                 400px     600px     800px       1050px
                   ├───────────────────┼─────────┼─────────┼───────────┼─────────>
     Slice #:                1              2         3          4          5
     
The goal of Breakpoint Slicer is to let you quickly set breakpoints using these numbers instead of px/em values.


### Enter the mixins

Breakpoint Slicer offers four handy mixins that let you set breakpoint ranges easily: `at`, `from`, `to` and `between`:

Styles under `at(3)`        are applied when browser window width is inside the 3rd slice.

Styles under `from(3)`      are applied when browser window width is inside the 3rd slice or larger.

Styles under `to(3)`        are applied when browser window width is inside the 3rd slice or smaller.

Styles under `between(2,4)` are applied when browser window width is inside the 2nd, 4rd slice or any slice between the two (if any).

     Breakpoint:   0                 400px     600px     800px       1050px
                   ├───────────────────┼─────────┼─────────┼───────────┼─────────>
     Slice #:                1         ·    2         3          4     ·    5
                   ·                   ·         ·         ·           ·
                   ·                   ·         ·  at(3)  ·           ·
                   ·                   ·         ├─────────┤           ·
                   ·                   ·         ·         ·           ·
                   ·                   ·         · from(3) ·           ·
                   ·                   ·         ├───────────────────────────────>
                   ·                   ·                   ·           ·
                   ·                   ·            to(3)  ·           ·
                   ├───────────────────────────────────────┤           ·
                                       ·                               ·
                                       ·         between(2, 4)         ·
                                       ├───────────────────────────────┤


### Comparison between Breakpoint Slicer and vanilla Breakpoint

Slicer usage       | Breakpoint equivalent         | The resulting media query
------------------ | ----------------------------- | ------------------------------------------
at(2)              | breakpoint(400px 600px)       | (min-width: 400px) and (max-width: 600px)
from(2)            | breakpoint(400px)             | (min-width: 400px)
to(2)              | breakpoint(max-width 600px)   | (max-width: 600px)
between(2, 4)      | breakpoint(400px 1050px)      | (min-width: 400px) and (max-width: 1050px)


### Edge cases

Note that the last slice does not have a right edge. When it is invoked, the media query will have no max-width value.

Some mixins become synonomous when used for the last slice:

     Breakpoint:   0                 400px     600px     800px       1050px
                   ├───────────────────┼─────────┼─────────┼───────────┼─────────>
     Slice #:                1         ·    2         3          4     ·    5
                                       ·                               ·
                                       ·                               ·  at(5)
                                       ·                               ├─────────>
                                       ·                               ·
                                       ·                               · from(5)
                                       ·                               ├─────────>
                                       ·
                                       ·                  from(2)
                                       ├─────────────────────────────────────────>
                                       ·
                                       ·               between(2, 5)
                                       ├─────────────────────────────────────────>

…and `to` becomes meaningless (in this case the media query is omitted):

     Breakpoint:   0                 400px     600px     800px       1050px
                   ├───────────────────┼─────────┼─────────┼───────────┼─────────>
     Slice #:      ·         1              2         3          4          5
                   ·
                   ·                                                      to(5)
                   ├─────────────────────────────────────────────────────────────>

Note that the max-width of your site's container should be somewhere in the fifth slice.


### Demo

There's a small page testing Breakpoint Slicer in action: http://lolmaus.github.io/breakpoint-slicer/

The code behind the demo page resides here: https://github.com/lolmaus/breakpoint-slicer/tree/gh-pages


Installation
------------

Breakpoint Slicer is installed as any other Compass extension.

Run `gem install breakpoint-slicer` in a terminal, then add `require 'breakpoint-slicer` to `config.rb` and `@import 'breakpoint-slicer';` in th beginning of your SCSS code.

Instead of installing the gem manually, consider using [Bundler][3].


Code examples
-------------

Enlist your breakpoints in the `$slicer-breakpoints` variable:

    $slicer-breakpoints: 0 400px 600px 800px 1050px;
    
If you don't do that, defaults are used (these were the defaults).

    
### Basic usage

Then you can use Breakpoint Slicer's mixins the same way as you use the Breakpoint mixins:

    .element {
      @include at(2) {
        // Code from this block will only be applied to .element
        // when browser window width is between 400px and 600px.
        background-color: red;
        
        // This is a mixin from Singularity
        @include grid-span(2, 4); } }
        

### Creating a responsive grid

With Breakpoint Slicer you can easily cycle through scices and apply column spans accordingly. The example below makes use of [Susy][2].

    $slicer-breakpoints: 0 400px 600px 800px 1050px;
    $amount-of-slices:   length($slicer-breakpoints);

    // Say we have a grid of thumbnails and we position them with nth-child
    .thumbnail {
      
      // Cycling through all slices
      @for $i from 1 through $amount-of-slices {

        // Setting a media query for each slice
        @at($i) {

          // Using the Susy grid framework
          @include span-columns(1, $context: $i); // Set the number of columns equal to the slice number
                                                  // and make each thumbnail one column wide.
          @include nth-omega(#{$i}n); }}}
          

### Leveraging Singularity

[Singularity][3] is the most modern and versatile SASS grid system. It [requires][4] breakpoints to be provided individually.

Breakpoint Slicer offers a function `bp()` that returns the left breakpoint of a slice asked for. E. g. `bp(2)` would return `600px`. Effectively, `bp()` is a shortcut for `nth()`.

    // Define breakpoints in a Slicer list
    $slicer-breakpoints: 0 400px 600px 800px 1050px;

    // Define Singularity responsive grids
    $grids: 3;
    $grids: add-grid(6         at bp(2));
    $grids: add-grid(12        at bp(3));
    $grids: add-grid(2 8 2     at bp(4));
    $grids: add-grid(1 3 5 7 9 at bp(5));

It's very convenient to set the number of Singularity columns equal to the number of slices:

    $slicer-breakpoints: 0 400px 600px 800px 1050px;
    
    $grids: 1;
    @for $i from 2 through total-slices() {
      $grids: add-grid($i at bp($i)); }
      

Using Breakpoint Slicer together with vanilla Breakpoint or Respond To
----------------------------------------------------------------------

Breakpoint Slicer only works with min/max width. When you need some other media queries like media types, resolution, orientation, etc — use Breakpoint in a conventional way, Breakpoint Slicer won't interfere.

You can even have different sets of min/max widths defined with Breakpoint Slicer and Respond To (or vanilla Breakpoint).


Credit
------

The initial idea of Breakpoint Slicer came when i (Andrey Mikhaylov aka lolmaus) got to know Susy and learned that the number of columns in a grid can be different for different browser widths. Susy gave me an idea that it's convenient to have a number of breakpoints and for each breakpoint the number of columns is equal to that breakpoint's number.

This Compass extension was made possible with kind support from Eric Meyer, Sam Richard aka Snugug and Scott Kellum.


License
-------

Licensed under MIT/GPL.

- GPL2 license: http://www.gnu.org/licenses/gpl-2.0.html
- MIT license: http://www.opensource.org/licenses/mit-license.php


  [1]: https://github.com/Team-Sass/breakpoint
  [2]: http://susy.oddbird.net/
  [3]: https://github.com/Team-Sass/Singularity
  [4]: https://github.com/Team-Sass/Singularity/wiki/Creating-Grids#responsive-grids

