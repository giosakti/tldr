# tmux

## Managing Session
$ tmux new -s {name}
$ tmux ls
$ tmux attach -t {session_name}
$ tmux kill-session -t {session_name}
<C-b$>          rename session
<C-bd>          detach

## Pane Management
<C-b%>          split vertically
<C-b">          split horizontally
<C-bo>          goto next pane
<C-bq{num}>     goto pane
<C-b{direct}>   switch window focus
<C-bz>          toggle current pane into fullscreen

<C-b{{,}}>      cycle current pane
<C-b>:resize-pane -{U,L,R,D} {amount}

## Window Management
<C-bc>          add a new window
<C-b{num}>      switch between windows
<C-b>:rename-window rename window
