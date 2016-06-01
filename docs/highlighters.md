zsh-syntax-highlighting / highlighters
======================================

Syntax highlighting is done by pluggable highlighters:

* `main` - the base highlighter, and the only one [active by default][1].
* `brackets` - [matches brackets][2] and parenthesis.
* `pattern` - matches [user-defined patterns][3].
* `cursor` - matches [the cursor position][4].
* `root` - highlights the whole command line [if the current user is root][5].
* `line` - applied to [the whole command line][6].

[1]: highlighters/main.md
[2]: highlighters/brackets.md
[3]: highlighters/pattern.md
[4]: highlighters/cursor.md
[5]: highlighters/root.md
[6]: highlighters/line.md


How to activate highlighters
----------------------------

To activate an highlighter, add it to the `ZSH_HIGHLIGHT_HIGHLIGHTERS` array in
`~/.zshrc`, for example:

    ZSH_HIGHLIGHT_HIGHLIGHTERS=(main brackets pattern cursor)

By default, `$ZSH_HIGHLIGHT_HIGHLIGHTERS` is unset and only the `main`
highlighter is active.


How to tweak highlighters
-------------------------

Highlighters look up styles from the `ZSH_HIGHLIGHT_STYLES` associative array.
Navigate into the [individual highlighters' documentation](highlighters/) to
see what styles (keys) each highlighter defines; the syntax for values is the
same as the syntax of "types of highlighting" of the zsh builtin
`$zle_highlight` array, which is documented in [the `zshzle(1)` manual
page][zshzle-Character-Highlighting].

[zshzle-Character-Highlighting]: http://zsh.sourceforge.net/Doc/Release/Zsh-Line-Editor.html#Character-Highlighting

Styles may be set directly or by themes. If no theme is specified in
`ZSH_HIGHLIGHT_THEME` the `default` theme will be loaded. Additional themes
may be layered on top (overriding previous theme's settings) by calling
`_zsh_highlight_load_theme`.  `_zsh_highlight_load_theme` takes the path to
a theme as an argument. Relative paths are relative to the themes directory in
zsh-syntax-highlighting's root directory and the themes directory (if any) in
each highlighter's directory.

The `default` theme is a colorful theme that preserves the defaults the
highlighters originally had. The `error-only` theme is also available for
highlighting only syntax errors. If `ZSH_HIGHLIGHT_THEME` is set but empty, no
theme will be loaded.

Some highlighters support additional configuration parameters; see each
highlighter's documentation for details and examples.


How to implement a new highlighter
----------------------------------

To create your own `myhighlighter` highlighter:

* Create your script at
    `highlighters/${myhighlighter}/${myhighlighter}-highlighter.zsh`.

* Implement the `_zsh_highlight_myhighlighter_highlighter_predicate` function.
  This function must return 0 when the highlighter needs to be called and
  non-zero otherwise, for example:

        _zsh_highlight_myhighlighter_highlighter_predicate() {
          # Call this highlighter in SVN working copies
          [[ -d .svn ]]
        }

* Implement the `_zsh_highlight_myhighlighter_highlighter` function.
  This function does the actual syntax highlighting, by calling
  `_zsh_highlight_add_highlight` with the start and end of the region to
  be highlighted and the `ZSH_HIGHLIGHT_STYLES` key to use.

  FIXME: How should third-party highlighters hook provide themes?

  Define the default
  style for that key in the highlighter script outside of any function with
  `: ${ZSH_HIGHLIGHT_STYLES[key]:=value}`, being sure to prefix
  the key with your highlighter name and a colon. For example:

        : ${ZSH_HIGHLIGHT_STYLES[myhighlighter:aurora]:=fg=green}

        _zsh_highlight_myhighlighter_highlighter() {
          # Colorize the whole buffer with the 'aurora' style
          _zsh_highlight_add_highlight 0 $#BUFFER myhighlighter:aurora
        }

* Activate your highlighter in `~/.zshrc`:

        ZSH_HIGHLIGHT_HIGHLIGHTERS+=(myhighlighter)

* [Write tests](../tests/README.md).
