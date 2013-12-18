# codedit
Code editor engine in Lua

##  THIS IS WORK IN PROGRESS / NOTHING TO SEE HERE (YET)

v1.0 | [code](http://code.google.com/p/lua-files/source/browse/codedit.lua) | [demo](http://code.google.com/p/lua-files/source/browse/codedit_demo.lua) | LuaJIT 2, Lua 5.1, Lua 5.2

## `local codedit = require'codedit'`

Codedit is a source code editor engine written in Lua. It has a minimalist design, striving to capture the essence of source code editing, making it easy to explore, understand and extend. It also comes with some cool features out-of-the-box.

## Highlights
  * utf8-ready, using a small [string module](http://code.google.com/p/lua-files/source/browse/codedit_str.lua) over [utf8].
  * cross-platform: written in Lua and has no dependencies
  * simple interface for integrating with rendering and input APIs ([https://code.google.com/p/lua-files/source/browse/cplayer/code_editor.lua example])
  * highly modular, with separate buffer, cursor, selection, view and controller objects, allowing multiple cursors and multiple selections.

## Features
  * *Buffers* ([code](http://code.google.com/p/lua-files/source/browse/codedit_buffer.lua))
    * *File format autodetection* ([code](http://code.google.com/p/lua-files/source/browse/codedit_detect.lua))
      * loading files with mixed line endings
      * detecting the most common line ending used in the file and using that when saving the file
    * *Normalization* ([code](http://code.google.com/p/lua-files/source/browse/codedit_normal.lua))
      * removing spaces past end-of-line before saving
      * removing empty lines at end-of-file before saving, or ensuring that the file ends with at least one empty line before saving
    * undo/redo stack ([code](http://code.google.com/p/lua-files/source/browse/codedit_undo.lua))
  * *Selections* ([code](http://code.google.com/p/lua-files/source/browse/codedit_selction.lua))
    * block (column) selection mode ([code](http://code.google.com/p/lua-files/source/browse/codedit_blocksel.lua))
    * indent/outdent (also for block selections)
  * *Cursors* ([code](http://code.google.com/p/lua-files/source/browse/codedit_cursor.lua))
    * insert and overwrite insert modes, with wide overwrite caret
    * smart tabs: use tabs only when indenting, and use spaces inside the lines
    * option to allow or restrict the cursor past end-of-line
    * option to allow or restrict the cursor past end-of-file
    * auto-indent: copy the indent of the line above when pressing enter
    * moving through words
  * *Rendering* ([code](http://code.google.com/p/lua-files/source/browse/codedit_render.lua))
    * syntax highlighting using [scintillua](http://foicica.com/scintillua/) lexers
    * simple rendering and measuring API for monospace fonts ([code](http://code.google.com/p/lua-files/source/browse/codedit_metrics.lua))
    * user-defined margins ([code](http://code.google.com/p/lua-files/source/browse/codedit_margin.lua))
      * line numbers margin ([code](http://code.google.com/p/lua-files/source/browse/codedit_line_numbers.lua))
  * *Controller* ([code](http://code.google.com/p/lua-files/source/browse/codedit_editor.lua))
    * configurable key bindings and commands ([code](http://code.google.com/p/lua-files/source/browse/codedit_keys.lua))
    * simple clipboard API (stubbed to an in-process clipboard)
    * scrolling, one line/char at a time or smooth scrolling ([code](http://code.google.com/p/lua-files/source/browse/codedit_scroll.lua))
    * selecting with the mouse


## Limitations
  * fixed char width (monospace fonts only)
  * fixed line height
  * no incremental repaint
  * mixed line terminators are not preserved

## Usage

	local codedit = require'codedit'
	local glue = require'glue'

	--subclass codedit to hook in input and rendering APIs

	local myeditor = glue.inherit({}, codedit) --use glue.update() for static inheritance

	function myeditor:draw_char(x, y, c, color)
		--implement drawing a utf8 character at (x, y) coordinates.
		--the coordinates are relative to the editor's client area.
		--the color is always 'text' (it will be 'number', 'string' etc. when syntax highlighting will be implemented).
	end

	function myeditor:draw_rect(x, y, w, h, color)
		--draw a rectangle. the color can be 'selection', 'cursor', 'background'.
	end

	--create an editor instance

	local editor = myeditor:new{}


	local editor = codedit:new{option = value, ...}


## Buffers

Buffers are at the core of text editing. A buffer stores the text as a list of lines and contains methods for analyzing, navigating, selecting and editing the text at a logical level, independent of how the text is rendered. The buffer contains methods that deal with text at various levels of abstraction. At the bottom we have lines of utf8 codepoints (chars), let's call that the binary space. Over that there's the char space, the space of lines and columns, where any char can found by the pair (line, col). Since the chars are stored as utf8, the correspondence between char space and binary space is not linear. We don't deal much in binary space, only in char space (we use the utf8 library to traverse the codepoints). The space outside of the available text is called the unclamped char space. We cannot select text from this space, but we can navigate it as if it was made of empty lines. Higher up there's the visual space, which is how the text looks after tab expansion, for a fixed tab size. Again, the correspondence between char space (let's call it real space) and visual space is not linear. Since we don't support automatic line wrapping, lines have a 1:1 correspondence between all these spaces, only the columns are different.


## API

*utils*
----------------------------------------------------------------------------------------- ----------------------------------------------------------------------------------------------
`editor:detect_line_terminator(s)`                                                                            class method that returns the most common line terminator in a string, or `self.default_line_terminator`
*undo/redo*
`editor:undo()`                                                                            undo
`editor:redo()`                                                                            redo
*line interface*
`editor:getline(line) -> s`                                                                            get the contents of a line
`editor:last_line() -> line`                                                                            get the last line number (or the number of lines)
`editor:contents([lines]) -> s`                                                                            get all the lines in the line buffer concatenated with `self.line_terminator`
`editor:insert_line(line, s)`                                                                            insert a new line
`editor:remove_line(line)`                                                                            remove a line
`editor:setline(line, s)`                                                                            change a line's contents
*(line,col) interface*
`editor:last_col(line) -> col`                                                                            last column of a line
`editor:indent_col(line) -> col`                                                                            column of the first non-space character of a line
`editor:isempty(line) -> true|false`                                                                            line is empty or contains only spaces
`editor:sub(line, col1, col2) -> s`                                                                            chop a line
`editor:clamp(line, col) -> line, col`                                                                            clamp a (line, col) pair to the available text
`editor:select_string(line1, col1, line2, col2) -> s`                                                                            select the string between two subsequent positions in the text
`editor:insert_string(line, col, s) -> line, col`                                                                            insert a string into the text, returning the position right after it
`editor:remove_string(line1, col1, line2, col2) -> line, col`                                                                            remove the string between two positions in the text
`editor:extend(line, col)`                                                                            extend the text up to (line,col-1) so we can edit there
*normalization*
`editor:remove_eol_spaces()`                                                                            remove any spaces past end-of-line
`editor:ensure_eof_line()`                                                                            add an empty line at eof if there is none
`editor:remove_eof_lines()`                                                                            remove any empty lines at eof, except the first one
`editor:normalize()`                                                                            normalize the text following current normalization options
*tab expansion*
`editor:tabstop_distance(vcol) -> n`                                                                            how many spaces from a visual column to the next tabstop, for a specific tabsize
`editor:visual_col(line, col) -> vcol`                                                                            real column -> visual column, for a fixed tabsize. the real column can be past string's end, in which case vcol will expand to the same amount.
`editor:real_col(line, vcol) -> col`                                                                            visual column -> real column, for a fixed tabsize. if the target vcol is between two possible vcols, return the vcol that is closer.
`editor:max_visual_col() -> vcol`                                                                            max. visual column
`editor:aligned_col(target_line, line, col) -> col`                                                                            real col on a line vertically aligned to the real col on a different line
`editor:select_block(line1, col1, line2, col2) -> s`                                                                            select the visually rectangular block between two subequent positions in the text
`editor:remove_block(line1, col1, line2, col2)`                                                                            remove the visually rectangular block between two subequent positions in the text
*selections*
`editor.selection`                                                                            selection class
`editor:create_selection([visible]) -> selection`                                                                            create a new selection object
`selection:free()`                                                                            unregister a visible selection from the editor
`selection.line1`, `selection.col1`, `selection.line2`, `selection.col2`                                                                            (line1,col1) is the position of the first selected char and (line2,col2) the position of the char just after the last selected char.
`selection:isempty() -> true|false`                                                                            check if selection is empty
`selection:move(line, col[, selecting])`                                                                            reset and re-anchor a selection, or, if `selecting` is true, extend it
`selection:lines() -> iterator<line, col1, col2`                                                                            iterate the lines of a selection, giving the start and end column of each line
`selection:contents() -> s`                                                                            selection as string, lines concatenated with `self.line_terminator`
`selection:remove()`                                                                            remove the selected text from the buffer
`selection:indent(levels)`                                                                            indent or outdent a selection
*cursors*
`editor.cursor`                                                                            in an editor class, it is the cursor class. in an editor instance, it is the default cursor that is bound to the keyboard.
`editor:create_cursor([visible]) -> cursor`                                                                            create a new cursor object
`cursor:free()`                                                                            unregister a visible cursor from the editor
`cursor.selection`                                                                            each cursor has a selection object attached
`cursor.insert_mode`                                                                            insert or overwrite when typing characters
`cursor.auto_indent`                                                                            pressing enter copies the indentation of the current line over to the following line
`cursor.restrict_eol`                                                                            don't allow caret past end-of-line
`cursor.restrict_eof`                                                                            don't allow caret past end-of-file
`cursor.tabs`                                                                            when to use `'\t'` when pressing tab: 'never', 'indent', 'always'
`cursor.tab_align_list`                                                                            align to the next word on the above line ; incompatible with `tabs = 'always'`
`cursor.tab_align_args`                                                                            align to the char after '(' on the above line; incompatible with `tabs = 'always'`
`cursor.keep_on_page_change`                                                                            preserve cursor position through page-up/page-down
`cursor.line`, `cursor.col`                                                                            cursor position
*cursors/navigation*
`cursor:move(line, col, [selecting], [store_vcol], [keep_screen_location])`                                                                            move cursor to a new location, optionally selecting text, storing the wanted visual column and/or scrolling the view to preserve the screen location.
`cursor:move_left(cols, selecting, keep_screen_location) `                                                                            move left
`cursor:move_right(cols, selecting, keep_screen_location) `                                                                            move right
`cursor:move_up(lines, selecting, keep_screen_location) `                                                                            move up
`cursor:move_down(lines, selecting, keep_screen_location) `                                                                            move down
`cursor:move_left_word(selecting)`                                                                            move left one word
`cursor:move_right_word(selecting)`                                                                            move right one word
`cursor:move_home(selecting)`                                                                            move home
`cursor:move_end(selecting)`                                                                            move to eof
`cursor:move_bol(selecting)`                                                                            move to beginning of line
`cursor:move_eol(selecting)`                                                                            move to end of line
`cursor:move_up_page(selecting)`                                                                            move up a page
`cursor:move_down_page(selecting)`                                                                            move down a page
* cursors/editing *
`cursor:newline()`                                                                            add a new line, optionally copy the indent of the current line, and carry the cursor over
`cursor:insert_char(c)`                                                                            insert printable char
`cursor:delete_before()`                                                                            delete the char before cursor
`cursor:delete_after()`                                                                            delete the char after cursor
