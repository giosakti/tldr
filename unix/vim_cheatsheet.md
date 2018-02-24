# Vim

## Switch between modes
<:>             Ex command
<i>             insert mode
<a>             append after cursor
<A>             append after eol
<o>             append after newline
<R>             replace mode
<v>             visual mode
<V>             visual line mode
<C-v>           visual block mode
:!{shell_cmd}   execute shell commands

## Motion
<{h,j,k,l}>     left, down, up, right
<{w,W,b,B}>     next, prev word
<{e,E,ge,gE}>   next, prev end word
<*>             next word under the cursor
<0>             bol
<^>             bol (nonblank)
<$>             eol
<{gg,G}>        bof, eof
<{line}G>       jump to line
<%>             switch between matching parantheses

<{f,F}{char}>   next, prev char
<{t,T}{char}>   before next, prev char
<;>             repeat
<,>             reverse

## Text Object
<{i,a}w>        inside, around word
<{i,a}W>        inside, around WORD
<{i,a}s>        inside, around sentence
<{i,a}p>        inside, around paragraph

## Operator
<c{text_obj}>     change*
<d{text_obj}>     delete*
<g~{text_obj}>    swap case*
<{<,>}{h,l}> shift left/right
<x>             deletes char under the cursor
<r{char}>       replace current char

## Register
<.>             repeat last action
<u>             undo
<C-r>           redo

<y{text_obj}>   yank*
<"{register}y>  yank to specified register
<p>             put
<"{register}p>  put from specified register
<iC-r=> expression register

Copy/paste in Mac
:{range}!pbcopy copy range to clipboard
:r !pbpaste "Paste clipboard content to current line

## Search and Replace
<{/,?}>         forward, backward search
<n>             repeat
<N>             reverse

:%s/target/replacement/gc
<&>             repeat

## Window Management
<C-{y,e}>       move down/up one line
<C-{d,u}>       move down/up 1/2 page
<C-{f,b}>       move down/up 1 page
<z{z,t,b}>      move current line to middle, top, bottom

:e {path}       open path
:sp {path}      horizontal split
:vs {path}      vertical split
<C-w{h,j,k,l}>  switch window focus
<C-w{H,J,K,L}>  switch window position
<C-w=>          equalize width and height of all windows
<{size}C-w_>    set height of active window
<{size}C-w|>    set width of active window

## Buffer Management
:ls
<C-^>, :bfirst, :blast, :bnext, :bprev, :buffer {num}
:bdelete

## Mark
<m{a-z}>        set mark
<`{a-z}>        goto mark

## Jumps
:jumps
<C-o>           backward
<C-i}           forward

## Autocompletion
<C-{p,n}>       previous/next autocompletion
<C-y>           accept autocompletion
<C-e>           revert autocompletion

## Ex Command
:{range}delete {register}
:{range}yank {register}
:{line}put {register}
:{range}copy {address}
:{range}move {address}
:{range}normal {commands}
:{range}s/{pattern}/{string}/[flags]

List of allowed range
{num}           absolute line number
.               cursor location
0               above bof
1               bof
$               eof
%               entire file
*               visual selection
'<              start visual
'>              end visual
'{mark}         mark

List of allowed pattern in substitute command
\r              carriage return
\t              tab character
\\              single backlash
~               pattern from previous invocation
\={vim_script}  vim script

## Insert Mode
<C-h>           delete back one char (bspace)
<C-w>           delete back one word
<C-u>           delete back until bol

<C-[>           normal mode
<C-o>           normal mode for one cmd

<C-r{register}> paste
<C-r=>          paste calculation

## Visual Mode
<gv>            reselect last
<o>             toggle selection start

## netrw
<%> create a new file
<d> create a new directory
<R> rename the file/directory under the cursor
<D> delete the file/directory under the cursor
