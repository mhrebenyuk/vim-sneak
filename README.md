# sneak.vim :shoe:

**Sneak** is a Vim plugin that fills the gap between `f` and `/` without
discarding Vim's many useful built-in motions.

See [Overview](#overview) for concept and [FAQ](#faq) for answers to common
questions.

<a href="http://imgur.com/Jke0mIJ" title="Click to see a short demo"><img src="https://raw.github.com/justinmk/vim-sneak/fluff/assets/readme_diagram.png"></a>

### Usage (assuming default mappings)

Sneak is invoked with `s` (sneak forward) or `S` (sneak backwards), followed by exactly two 
characters:

    s{char}{char}

* Press `sab` to **move the cursor** immediately to the next instance of the text "ab".
    * Additional matches, if any, are highlighted until the cursor is moved.
* Press `;` to go to the next match.
* Press `ctrl-o` or `` to go back to the starting point.
    * This is a built-in Vim motion; Sneak adds to Vim's [jumplist](http://vimdoc.sourceforge.net/htmldoc/motion.html#jumplist)
      *only* on `s` invocation, *not* repeated matches, so you can 
      always abandon a trail of `;` or `,` by a single `ctrl-o` or ``).

Sneak can be **scoped** to a column of width 2×{number} by prefixing `s`
with a number.

* Press `5sxy` to go immediately to the next instance of the text "xy" 
  scoped to the *5 columns left and right of the cursor*. 
    * This *vertical scope* is indicated by a highlight block.

Sneak is invoked with [**operators**](http://vimdoc.sourceforge.net/htmldoc/motion.html#operator)
by using `z` (because `s` is taken by surround.vim).

* Press `dzqt` to delete the text from the cursor to the next instance of the text "qt".
    * Press `.` to repeat the `dzqt` operation.
* Press `gUz }` to upper-case the text from the cursor to the next instance of
  the text " }".
    * Press `.` to repeat the `gUz }` operation.

### Overview

    l  f  t  %  'm  }  ]m  ]]  M  L     /
                                     ^
                                     |
                                   sneak

Vim's built-in motions cover many special cases, but it's not always easy to move across 
several lines—or a long line—to an arbitrary position. The `f` motion is restricted to 
the current line, and the `/` search is [clunky](#faq) for medium-distance 
motion . `H`, `M`, and `L` are great for bisecting a distance, but from there
the cursor is potentially several lines and columns away from the target.

Sneak covers that distance in three keystrokes:

    s{char}{char}

Sneak was inspired by [Seek](https://github.com/goldfeld/vim-seek) and 
[EasyMotion](https://github.com/Lokaltog/vim-easymotion), but it's written from scratch and uses stock 
Vim functions for its core behavior. Unlike Seek, Sneak works across **multiple lines**, 
works with **operators** (including **repeat** with dot `.`), works in **visual mode**, 
and supports **motion-repeat** via `;` and `,`. Unlike EasyMotion, Sneak is 
designed to *complement*—not replace—Vim's built-in motions; and
EasyMotion by default requires five (5) keystrokes to move to a position, while
the common case for Sneak is **three (3) keystrokes**.

### Details

* Sneak chooses sane defaults out-of-the-box, but the defaults can be changed using the various 
  `<Plug>` mappings. E.g., if you would rather just replace Vim's built-in `f` completely with 
  Sneak, put this in your .vimrc: 
```
      nmap f <Plug>SneakForward
      nmap F <Plug>SneakBackward
      let g:sneak#nextprev_f = 0
```

* Default mappings:
    * `s` (and `S`) waits for two characters, then immediately moves to the next (previous) match. 
      Additional matches are highlighted until the cursor is moved. **Works across multiple lines.**
        * `s` works in visual mode, too (use `Z` to go backward, because `S` is taken by surround.vim).
    * `{number}s` enters *vertical scope* mode which restricts the search to 2× the column width specified by `{number}`. 
    * `;` and `,` repeat the last `s` and `S`. **They also work correctly with `f` and `t`.**
        * If your mapleader is `,` then sneak.vim maps `\` instead of `,`. You can 
          override this by specifying some other mapping: `nmap ? <Plug>SneakPrevious`
    * Use `z` for operations; for example, `dzab` deletes from the cursor to the next instance of "ab". 
      `dZab` deletes backwards to the previous instance. `czab` `cZab` `yzab` and `yZab` also work as expected.
        * **Repeat the operation** with dot `.` (requires [repeat.vim](https://github.com/tpope/vim-repeat))


If you, or one of your plugins, already maps `s` and `S` to some feature, Sneak 
provides `<Plug>` mappings for you to specify alternative key maps. Keep in mind, 
however, that *motion* mappings should absolutely be the *least friction* mappings 
in your editor, because motion is the most common editor task. Remapping Sneak to 
something like `<leader>s` is really not recommended; consider moving your existing `s` 
and `S` mappings to some other corner of your keyboard. 

### Motivation

Here's how Sneak differs from Vim's built-in `/` search and other plugins:

  - move to any location with `s` followed by exactly two characters
  - move anywhere, even offscreen (unlike EasyMotion)
  - jump back to the point of `s` invocation via `ctrl-o` or `` (backtick backtick)
    - only the *initial* invocation adds to the jumplist
        - repeat-motion via `;` or `,` does *not* add to the jumplist
  - jumps immediately to first match (unlike EasyMotion)
  - gets out of your way as soon as you move the cursor (highlights and autocmds are cleared)
  - common case requires 3-char key sequence (EasyMotion requires 5: `,,fab`)
  - repeat the motion with `;` or `,`
  - does not break expected behavior of `f t F T ; ,`
  - preserves the `/` register, does not add noise to `/` history
  - does not wrap
  - highlights additional matches until a key other than ; or , is pressed
  - *vertical scope* with `{number}s{char}{char}` restricts search column to 2× `number` size
  - always literal, for example `s\*` jumps to the literal `\*`

### Bugs

Sneak is designed to be well-behaved and bug-free. There should be no surprises except pleasant 
surprises—like "OMG, it actually works".

If you find a bug, please report it, and perhaps include the output of:

    :call sneak#debug#report()

Sneak is tested on 100k+ line syntax-highlighted file, with Vim 7.2.330, 7.3, 7.4.

### Installation

To install Sneak manually, just place the files directly in your `.vim` directory 
(`_vimfiles` on Windows).

Or, use a plugin manager:

- [Pathogen](https://github.com/tpope/vim-pathogen)
  - `cd ~/.vim/bundle && git clone git://github.com/justinmk/vim-sneak.git`
- [Vundle](https://github.com/gmarik/vundle)
  1. Add `Bundle 'justinmk/vim-sneak'` to .vimrc
  2. Run `:BundleInstall`
- [NeoBundle](https://github.com/Shougo/neobundle.vim)
  1. Add `NeoBundle 'justinmk/vim-sneak'` to .vimrc
  2. Run `:NeoBundleInstall`
- [vim-plug](https://github.com/junegunn/vim-plug)
  1. Add `Plug 'justinmk/vim-sneak'` to .vimrc
  2. Run `:PlugInstall`

If you want to be able to repeat Sneak *operations* (like `dzab`) with dot `.`,
then [repeat.vim](https://github.com/tpope/vim-repeat) is required. However, to repeat 
Sneak *motions* via `;` and `,` you don't need to install anything except Sneak.

### FAQ

#### Why not use `/` with a two-character search?

* `/ab<cr>` requires 33% more keystrokes than `sab`
  * `f` and `t` exist for a similar reason
  * common operations should use as few keystrokes as possible
* clutters your search history
* requires escaping of special characters
  * you could `nnoremap / /\V`, but then you have a different problem
* highlight madness

This is why Vim has [motions](http://vimdoc.sourceforge.net/htmldoc/motion.html#left-right-motions).

#### Why not use `f` and mash `;` ?

Sneak is like `f` with these advantages:

* fifty times (50×) *more precise* than `f` or `t`
* moves vertically, too
* remembers the initial position in the Vim jumplist
  * this is more useful than you think: you can explore a trail of matches
    by mashing `;`, and then at any point return to the initial position via
    `ctrl-o` or `` (backtick backtick)
* highlights additional matches *in the direction of your search* 
  * gives a visual impression of how far you are from a target

#### How dare you remap `s`?

You can specify any mapping for Sneak (see [help doc](doc/sneak.txt)).

### Related
* [Seek](https://github.com/goldfeld/vim-seek)
* [EasyMotion](https://github.com/Lokaltog/vim-easymotion)
* [clever-f](https://github.com/rhysd/clever-f.vim)
* [vim-extended-ft](https://github.com/svermeulen/vim-extended-ft)
* [Fanf,ingTastic; ](https://github.com/dahu/vim-fanfingtastic)

### TODO
* repeat-operation should take a count
* `n;` should skip to *nth* occurrence
* vertical scope for built-in `/`
* use `strwidth()` instead of `len()` to support multibyte (vim-7.3 only) 
* operator-pending mode should accept registers
* add to VAM pool https://github.com/MarcWeber/vim-addon-manager

### License

Copyright © Justin M. Keyes. Distributed under the MIT license.


[![Bitdeli Badge](https://d2weczhvl823v0.cloudfront.net/justinmk/vim-sneak/trend.png)](https://bitdeli.com/free "Bitdeli Badge")

