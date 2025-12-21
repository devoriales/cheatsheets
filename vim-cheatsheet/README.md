# Devoriales - Vim Cheatsheet

## The Basics: Modes

Vim is a modal editor. The most important concept is understanding which mode you are in.

- **Normal Mode:** The default mode for navigation and manipulation. Press `Esc` to return here.
- **Insert Mode:** For typing text. Press `i` to enter.
- **Visual Mode:** For selecting text. Press `v` to enter.
- **Command-Line Mode:** For saving, exiting, and complex commands. Press `:` to enter.

## 1. Navigation (Normal Mode)

**Stop using arrow keys!**

### Basic Movement

| Command | Action |
|---------|--------|
| `h` | Move Left |
| `j` | Move Down |
| `k` | Move Up |
| `l` | Move Right |

### Word Movement

| Command | Action |
|---------|--------|
| `w` | Jump forward to start of next word |
| `W` | Jump forward to start of next WORD (ignoring punctuation) |
| `e` | Jump forward to end of word |
| `E` | Jump forward to end of WORD |
| `b` | Jump backward to start of word |
| `B` | Jump backward to start of WORD |

### Line Movement

| Command | Action |
|---------|--------|
| `0` (zero) | Jump to start of line |
| `^` | Jump to first non-blank character of line |
| `$` | Jump to end of line |
| `gg` | Go to first line of file |
| `G` | Go to last line of file |
| `:n` or `nG` | Go to line number `n` (e.g., `:42` or `42G`) |

### Screen Movement

| Command | Action |
|---------|--------|
| `Ctrl + u` | Scroll up half-page |
| `Ctrl + d` | Scroll down half-page |
| `Ctrl + b` | Scroll up full page |
| `Ctrl + f` | Scroll down full page |
| `zz` | Center screen on cursor |

## 2. Entering Insert Mode

| Command | Action |
|---------|--------|
| `i` | Insert before cursor |
| `I` | Insert at beginning of line |
| `a` | Append after cursor |
| `A` | Append at end of line |
| `o` | Open new line below cursor and insert |
| `O` | Open new line above cursor and insert |

## 3. Editing Text

### Deleting (Cutting)

| Command | Action |
|---------|--------|
| `x` | Delete character under cursor |
| `X` | Delete character before cursor |
| `dw` | Delete from cursor to next word |
| `db` | Delete from cursor to previous word |
| `dd` | Delete (cut) entire line |
| `D` | Delete from cursor to end of line |
| `dt{char}` | Delete up until `{char}` |

### Changing (Delete + Insert Mode)

| Command | Action |
|---------|--------|
| `r{char}` | Replace character under cursor with `{char}` |
| `cw` | Change word (delete word and enter Insert Mode) |
| `cc` | Change entire line |
| `C` | Change from cursor to end of line |
| `s` | Delete character and enter Insert Mode |

### Undo / Redo

| Command | Action |
|---------|--------|
| `u` | Undo |
| `Ctrl + r` | Redo |

## 4. Copy and Paste (Registers)

Vim calls copying "yanking."

| Command | Action |
|---------|--------|
| `yy` | Yank (copy) current line |
| `yw` | Yank word |
| `y$` | Yank to end of line |
| `p` | Paste after cursor/line |
| `P` | Paste before cursor/line |

## 5. Visual Mode (Selection)

| Command | Action |
|---------|--------|
| `v` | Visual char mode (select characters) |
| `V` | Visual line mode (select whole lines) |
| `Ctrl + v` | Visual block mode (select vertical columns) |
| `d` | Delete selection |
| `y` | Yank (copy) selection |
| `>` or `<` | Indent/Outdent selection |

## 6. Search and Replace

### Search

| Command | Action |
|---------|--------|
| `/{text}` | Search forward for `{text}` |
| `?{text}` | Search backward for `{text}` |
| `n` | Repeat search in same direction |
| `N` | Repeat search in opposite direction |
| `*` | Search for word under cursor (forward) |
| `#` | Search for word under cursor (backward) |

### Replace (Substitute)

| Command | Action |
|---------|--------|
| `:s/old/new` | Replace first 'old' with 'new' in current line |
| `:s/old/new/g` | Replace all 'old' with 'new' in current line |
| `:%s/old/new/g` | Replace all 'old' with 'new' in entire file |
| `:%s/old/new/gc` | Replace with confirmation |

## 7. Macros (The Power of Vim)

Record a sequence of keystrokes to replay later.

1. Press `q{register}` to start recording (e.g., `qa` records to register 'a').
2. Perform your actions.
3. Press `q` to stop recording.
4. Press `@{register}` to replay (e.g., `@a`).
5. Press `@@` to replay the last executed macro.

## 8. Windows and Tabs

### Splits

| Command | Action |
|---------|--------|
| `:split` or `:sp` | Horizontal split |
| `:vsplit` or `:vsp` | Vertical split |
| `Ctrl + w, h/j/k/l` | Move cursor between windows |
| `Ctrl + w, =` | Equalize window sizes |

### Tabs

| Command | Action |
|---------|--------|
| `:tabnew` | Open new tab |
| `gt` | Go to next tab |
| `gT` | Go to previous tab |

## 9. File Management

| Command | Action |
|---------|--------|
| `:w` | Write (save) file |
| `:q` | Quit |
| `:q!` | Quit without saving (force) |
| `:wq` or `:x` | Save and quit |
| `:e {filename}` | Open a file |

## 10. Advanced Tricks

- **`.` (Dot Command):** Repeats the last change. This is incredibly powerful.
- **`ci"`:** Change inside quotes. Works with `(`, `[`, `{`, etc. (e.g., `ci(`).
- **`%`:** Jump to matching brace/parenthesis.
- **`~`:** Toggle case of character under cursor.

## 11. Useful Snippets

### Replace a word in the file

The following command will replace a word in the file:

```vim
:%s/<word>/<new_word>/g
```

### Comment out lines in the file

The following command will comment out all lines in the file if they do not start with a `#`:

```vim
:%s/^[^#]/#&/
```

Uncommenting of the lines that start with `#` can be done with:

```vim
:%s/^#//
```
