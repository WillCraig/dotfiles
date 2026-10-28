# VSCodeVim port of `intellij.ideavimrc`

User-level settings for the [Vim extension](https://marketplace.visualstudio.com/items?itemName=vscodevim.vim) (`vscodevim.vim`). Nothing here needs an administrator, a system directory, a symlink into one, or a shell setup script.

`intellij.ideavimrc` is unchanged.

## Install without admin

1. Open VS Code. In the Extensions view, search for **Vim** and install **Vim** published by vscodevim (extension id `vscodevim.vim`). That is a Marketplace install from the UI. If the Marketplace is blocked but you were given a `.vsix` in your user profile, use **Extensions: Install from VSIX...** instead. Both stay inside your user profile.
2. Command Palette: **Preferences: Open User Settings (JSON)**. Paste the keys from [`settings.vim.jsonc`](settings.vim.jsonc) into the top-level `{ ... }` object. If the file already has settings, merge the keys. Do not replace the file. If a key already exists (`vim.leader`, `editor.lineNumbers`, and so on), keep one copy.
3. Command Palette: **Preferences: Open Keyboard Shortcuts (JSON)**. Paste the objects from [`keybindings.json`](keybindings.json) into the existing array. Do not replace that file when it already has bindings; replacing it deletes them.
4. Command Palette: **Developer: Reload Window**.

The Command Palette opens the settings files VS Code is actually using, including a portable layout or a custom user-data directory. You do not need to write under `Program Files`, `/usr`, or the application bundle.

Optional copy, still user-writable only: keep this repo in your home directory and copy the JSON from there. Do not point a symlink at a protected path.

Leader is Space. After a reload, `Space f f` opens Quick Open.

## Why there is no `vscodevimrc`

`vim.vimrc.enable` and `vim.vimrc.path` load **remap lines only**. `set`, `<expr>`, `<Action>`, autocmds, and Which-Key are ignored. Most of this IdeaVim file is IntelliJ actions, which have to become VS Code command ids in `settings.json`.

A pared-down vimrc would hold only the plain remaps (`jj`, `H`/`L`, the indent reselect, the surround aliases). Those same remaps are already in `settings.vim.jsonc`. Loading both would either duplicate them or let settings win and hide drift. `vim.vimrc.enable` is set to `false` so a home-directory `.vimrc` or `.vscodevimrc` is not applied on top.

Use `vim.vimrc.path` only for a personal remap file you maintain yourself. Do not point it at `intellij.ideavimrc`.

`keybindings.json` is used where `vim.handleKeys` cannot express IdeaVim's `sethandler`. That setting is one switch for every mode. The extension already limits Ctrl-f and Ctrl-b to non-Insert. Ctrl-h/j/k/l and the Ctrl-g inline-completion prefix need a `when` clause, which only keyboard shortcuts can carry.

## What the options became

| IdeaVim | VS Code |
| --- | --- |
| `let mapleader = " "` | `vim.leader`: `<space>` |
| `let maplocalleader = "\\"` | No `vim.localleader`. No mapping uses it. |
| `set number` + `set relativenumber` | `editor.lineNumbers`: `relative` (current line absolute, others relative) |
| `set scrolloff=4` | `editor.cursorSurroundingLines`: `4` |
| `set sidescrolloff=8` | Not ported. No editor setting. |
| `set nowrap` | `editor.wordWrap`: `off` |
| `set incsearch` / `hlsearch` / `ignorecase` / `smartcase` | `vim.incsearch`, `vim.hlsearch`, `vim.ignorecase`, `vim.smartcase` |
| `set showmode` / `showcmd` | `vim.showmodename`, `vim.showcmd` |
| `set timeout` + `timeoutlen=500` | `vim.timeout`: `500` for every mode |
| `set clipboard=unnamedplus,ideaput` | `vim.useSystemClipboard` (register `*`). `ideaput` is not ported. |
| `set ideajoin` | Not ported. `J` is Vim's join. |
| `set idearefactormode=select` | Not ported. Rename uses the VS Code widget. |
| `set commentary` | Built in. `gc`, `gcc`, visual `gc`, and `gC` (block). No flag. |
| `set surround` | `vim.surround` |
| `set mini-ai` | Partial. See the plugin table. |
| `set argtextobj` | Built-in `ia` / `aa`. Delimiters are `(` `[` and separator `,`. |
| `set NERDTree` | No NERDTree. `<leader>e` and `<leader>fe` open the Explorer. |
| `set sneak` (commented out) | `vim.sneak`: `false`. Set it to `true` to turn Sneak on. |
| Which-Key block | Not ported. This file is the popup. |

EasyMotion, highlightedyank, multiple-cursors, and CamelCaseMotion are not in the IdeaVim file, so they are left off.

## Mapping table

Space is `<leader>`. "Same command" means the visual-mode map uses the command in the Normal row.

### Editing

| IdeaVim | VSCodeVim | Notes |
| --- | --- | --- |
| `<Space>` `<Nop>` | Not bound | Space is the leader prefix. |
| `<Esc>`, `<leader>ur` | `:nohl` | Normal mode. Clears search highlight. |
| `jj` | `<Esc>` | Insert, non-recursive. Waits up to `vim.timeout`. |
| `j` / `k` expr `gj`/`gk` | Not bound | No `<expr>` or `v:count`. With wrap off, `j` and `gj` match. Counted `j` stays real lines. |
| `n` / `N` expr | Not bound | `n` follows the last search direction. LazyVim's always-forward `n` is not available. |
| visual `<` / `>` | `<gv` / `>gv` | Reselects after indent. |
| `<C-s>` | `workbench.action.files.saveAll` | Normal, visual, insert. Select mode (`smap`) is not bound. |
| `gsa` / `gsd` / `gsr` | `ys` / `ds` / `cs` | Visual `gsa` is `S`. `ys` `cs` `ds` `S` stay the surround defaults. |
| Alt-j / Alt-k | Commented in the JSON | Same commands as **Move Line Down/Up**. Uncomment if Alt is delivered. |

### Files and editor tabs

| IdeaVim | Command | Notes |
| --- | --- | --- |
| `<leader><Space>`, `<leader>ff` | `workbench.action.quickOpen` | GotoFile |
| `<leader>,`, `<leader>fb`, `<leader>bj` | `workbench.action.showAllEditors` | Switcher |
| `<leader>fr` | `workbench.action.openRecent` | |
| `<leader>fn` | `workbench.action.files.newUntitledFile` | Untitled buffer, not an IDEA scratch-language picker. |
| `<leader>e`, `<leader>fe` | `workbench.view.explorer` | |
| `<leader>fP` | `workbench.files.action.showActiveFileInExplorer` | |
| `H`, `L`, `[b`, `]b` | `workbench.action.previousEditor` / `nextEditor` | Replaces Vim `H`/`L` (top and bottom of the screen). |
| `<leader>bb` and leader then backtick | `workbench.action.openPreviousRecentlyUsedEditor` | Alternate file, not the neighboring tab. |
| `<leader>bd`, `<leader>bD` | `:bdelete` | Both close the current editor. `bD` is not a different operation. |
| `<leader>bo` | `workbench.action.closeOtherEditors` | Other editors in the group. Pinned handling follows VS Code. |
| `<leader>bp` | `workbench.action.pinEditor` | Pins only. Unpin is `workbench.action.unpinEditor`. |
| `<leader>bP` | Not bound | No "close unpinned editors" command. |
| `<leader>bl`, `<leader>br` | `workbench.action.closeEditorsToTheLeft` / `ToTheRight` | Closes every editor that way, not only unpinned ones. |

### Search, code, diagnostics

| IdeaVim | Command | Notes |
| --- | --- | --- |
| `<leader>/`, `<leader>sg` | `workbench.action.findInFiles` | |
| `<leader>sb` | `actions.find` | |
| `<leader>sr` | `workbench.action.replaceInFiles` | Normal and visual. |
| `<leader>ss` | `workbench.action.gotoSymbol` | Current file. |
| `<leader>sS` | `workbench.action.showAllSymbols` | Workspace. |
| `<leader>sC` | `workbench.action.showCommands` | |
| `<leader>sc`, `<leader>:`, `<leader>s/` | Not bound | `:history` is not implemented. |
| `<leader>s"` | `:registers` | |
| `<leader>sj` | `:jumps` | |
| `<leader>sm` | `:marks` | |
| `<leader>sk`, `<leader>?` | Not bound | `:nmap` and Which-Key are not implemented. |
| `gd`, `gD` | `editor.action.revealDefinition` | The IdeaVim file maps both to GotoDeclaration. `editor.action.revealDeclaration` exists if you want a declaration-only jump. |
| `gI` | `editor.action.goToImplementation` | |
| `gy` | `editor.action.goToTypeDefinition` | |
| `gr` | `editor.action.goToReferences` | Replaces VS Code's `gr` chord prefix while Vim is active. |
| `K`, `<leader>cd` | `editor.action.showHover` | Hover is the diagnostic text at the cursor. |
| `gK` | `editor.action.triggerParameterHints` | |
| `<leader>ca` | `editor.action.quickFix` | Normal and visual. Code actions, the Alt+Enter stand-in. |
| `<leader>cr` | `editor.action.rename` | |
| `<leader>cf` | `editor.action.formatDocument` | Normal and visual. |
| `<leader>co` | `editor.action.organizeImports` | No-op when the language has no organize-imports provider. |
| `<leader>cs` | `outline.focus` | |
| `]d`, `[d` | `editor.action.marker.nextInFiles` / `prevInFiles` | Across files. |
| `<leader>xx`, `<leader>sd` | `workbench.actions.view.problems` | |
| `<leader>xq` | `workbench.view.search` | Search view, the Find tool window stand-in. |

### Windows

Splits and focus use editor groups, so they match each other. Vim `:split` windows are a separate system and are not what these keys drive.

| IdeaVim | Where it lives | Command |
| --- | --- | --- |
| `<C-h/j/k/l>` | `keybindings.json`, Normal only | `workbench.action.navigateLeft` / `Down` / `Up` / `Right` |
| `<leader>wh/wj/wk/wl` | settings | same navigate commands |
| `<leader>-`, `<leader>ws` | settings | `workbench.action.splitEditorDown` |
| `<leader><Bar>`, `<leader>wv` | settings | `workbench.action.splitEditorRight` |
| `<leader>wd` | settings | `workbench.action.closeEditorsAndGroup` |
| `<leader>wm`, `<leader>uZ` | settings | `workbench.action.toggleMaximizeEditorGroup` |

Ctrl-f and Ctrl-b stay page-down and page-up inside Vim outside Insert. The extension's own when-clause already skips Insert for those two keys. Ctrl-d and Ctrl-u stay with Vim in every mode that extension binding captures, including Insert, so half-page scrolling keeps working. See gaps.

### Git, debug, run, Copilot, display

| IdeaVim | Command | Notes |
| --- | --- | --- |
| `<leader>gg` | `workbench.view.scm` | Source Control. |
| `<leader>gs` | `workbench.scm.focus` | Changes and commit input. Same view family as `gg`, focused on the commit pane. |
| `<leader>gl` | `workbench.scm.history.focus` | Source Control Graph. |
| `<leader>gf` | `timeline.focus` | File history. |
| `<leader>gb` | `git.blame.toggleEditorDecoration` | Inline blame, not the IDEA annotate gutter. |
| `<leader>gd`, `<leader>ghd` | `git.openChange` | Working tree diff. This `gd` is `<leader>gd`, not `gd`. |
| `]h`, `[h` | `editor.action.dirtydiff.next` / `previous` | Next and previous change. |
| `<leader>dd` | `workbench.action.debug.start` | |
| `<leader>dS` | `workbench.action.debug.selectandstart` | Picks a launch config and starts debugging. |
| `<leader>dc` | `workbench.action.debug.continue` | Resume. Does not start a session. |
| `<leader>db` | `editor.debug.action.toggleBreakpoint` | |
| `<leader>dB` | `editor.debug.action.editBreakpoint` | |
| `<leader>dC` | `editor.debug.action.runToCursor` | |
| `<leader>di` / `do` / `dO` | step into / out / over | |
| `<leader>dP` | `workbench.action.debug.pause` | |
| `<leader>dt` | `workbench.action.debug.stop` | |
| `<leader>du` | `workbench.view.debug` | |
| `<leader>de` | `editor.debug.action.selectionToRepl` | Sends the selection to the Debug Console. Not a modal evaluate dialog. |
| `<leader>rr` | `workbench.action.debug.run` | Run without debugging. |
| `<leader>rc` | Not bound | No Run Class command in core VS Code. |
| `<leader>rs` | `workbench.action.debug.selectandstart` | Same picker as `<leader>dS`. It starts debugging. |
| `<leader>aa` | `workbench.action.chat.open` | Normal and visual. Copilot Chat when Copilot is the chat provider. |
| `<leader>aq` | `inlineChat.start` | Normal and visual. |
| `<leader>as` | `github.copilot.toggleStatusMenu` | Requires the Copilot extension. |
| `<leader>aS` | `workbench.action.openSettings` `@ext:GitHub.copilot` | |
| Ctrl-g then a/w/l/s/n/p | `keybindings.json`, Insert only | Accept, accept word, accept line, trigger, next, previous inline suggestion. |
| `<leader>ft` | `workbench.action.terminal.toggleTerminal` | |
| `<leader>ul` | `:set number!` | Writes `editor.lineNumbers`. See below. |
| `<leader>uL` | `:set relativenumber!` | Toggles `relative` and `on`. |
| `<leader>uw` | `:set wrap!` | |
| `<leader>uz` | `workbench.action.toggleZenMode` | |

### Line number toggles

VS Code stores one value: `on`, `relative`, `off`, or `interval`. Vim's `number` and `relativenumber` are independent, and both setters in VSCodeVim write that one value.

- `<leader>uL` flips between `relative` and `on`.
- `<leader>ul` from `relative` goes to `on`, from `on` goes to `off`, from `off` goes to `on`.

The pasted default is `relative`.

## Plugins

| IdeaVim | VSCodeVim |
| --- | --- |
| commentary | Always on. `gc` / `gcc` / `gC`. |
| surround | `vim.surround`. |
| argtextobj `ia`/`aa` | Built in. `cia`, `daa`. |
| mini-ai `iq`/`aq` | `vim.targets.smartQuotes`. Also `inq` / `ilq` for next and last quote. |
| mini-ai `ib`/`ab` any bracket | Not the same command. `ib` is parentheses. `vim.targets.bracketObjects` adds next/last per delimiter (`in(`, `il{`, and the square-bracket pair). |
| NERDTree | Explorer commands above. No `:NERDTree`. |
| sneak (off) | `vim.sneak`. Operator-pending form is `z`/`Z` while surround is on. |
| which-key | None. Mappings time out after 500ms. Finish a chord inside that window. |
| easymotion, highlightedyank, multiple-cursors | Not in the IdeaVim file. `gb` in VSCodeVim still adds a cursor at the next match of the word; that is the extension default, not a mapping from this port. |

## Gaps

These could not be ported as the IdeaVim file has them:

- Which-Key popup, group titles, and the InsertEnter `timeout` / `notimeout` switch.
- `<expr>` motions: counted `j`/`k` versus `gj`/`gk`, and LazyVim's always-forward `n`/`N`.
- `sethandler` for Ctrl-d and Ctrl-u. They stay with Vim in Insert as well as Normal, because that is how the extension's keybinding is written and half-page scrolling has no separate command. Insert loses VS Code's Ctrl+D (add selection to next match) and Ctrl+U (cursor undo) while the editor is focused.
- Select-mode Ctrl-s (`smap`).
- `<Space>` as `<Nop>` without eating the leader.
- mini-ai any-bracket `ib`/`ab`, NERDTree commands, `ideajoin`, `idearefactormode=select`, clipboard `ideaput`, `sidescrolloff`, and an unused `maplocalleader`.
- `:history` (`<leader>sc`, `<leader>:`, `<leader>s/`) and `:nmap` (`<leader>sk`, `<leader>?`).
- Close all unpinned editors (`<leader>bP`). Pin is not a toggle. Close-left and close-right are not limited to unpinned tabs.
- Run Class (`<leader>rc`). `<leader>rs` starts a debug session from the launch picker; there is no run-without-debugging picker.
- Evaluate is the Debug Console, not IDEA's evaluate dialog. Git blame is the inline decoration. File history is the Timeline. The commit UI and the Git view are one Source Control area.
- Ctrl+K chords do not run in Normal mode, because Ctrl+K moves to the group above. They still run in Insert and Visual. Outside Insert, Ctrl+G is Go to Line; a Ctrl+G chord in the shortcut list can make that key wait briefly.
- No Java or Spring command was in the IdeaVim file, so none is bound. `editor.action.organizeImports` and the debug commands are the language-neutral stand-ins.

Commented-out Sneak and Alt-j/Alt-k are preserved as comments or as `vim.sneak: false`, not turned on.
