# Phoenix Setup

## Install Erlang

```
wget https://packages.erlang-solutions.com/erlang-solutions_1.0_all.deb && sudo dpkg -i erlang-solutions_1.0_all.deb
sudo apt-get update
sudo apt-get install esl-erlang
```

## Install kiex

```
\curl -sSL https://raw.githubusercontent.com/taylor/kiex/master/install | bash -s
```

- add to rc files

```
test -s "$HOME/.kiex/scripts/kiex" && source "$HOME/.kiex/scripts/kiex"
```

## Install Elixir using kiex

```
kiex list known
kiex install <ELIXIR_VERSION>
```

## Install hex (Elixir package manager)

```
mix local.hex
```

## Install nodejs & npm

## Install Phoenix

```
mix archive.install https://github.com/phoenixframework/archives/raw/master/phx_new.ez
```

## Create new project

```
mix phx.new myapp
mix deps.get
```

## Run Phoenix application

```
cd myapp
mix ecto.create
mix phx.server
```

## Run IEx (Elixir REPL)

```
iex -S mix phx.server
iex -S mix
```

## Create migration

```
mix ecto.gen.migration create_user
mix ecto.migrate
mix phx.routes
mix phoenix.gen.html Video videos user_id:references:users url:string title:string description:text
```
