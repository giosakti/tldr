# Phoenix Deployment

https://hexdocs.pm/phoenix/deployment.html

## Fetch only production dependencies

```
mix deps.get --only prod
```

## Managing secrets

- Create secrets

```
create prod.secret.exs
mix phx.gen.secret
```

- Fetching secrets from environment variables

```
Map.fetch!(System.get_env(), "SECRET_KEY_BASE")
System.get_env("DATABASE_URL")
String.to_integer(System.get_env("POOL_SIZE") || "10")
```

## Compile in production mode

```
MIX_ENV=prod mix compile
```

## Run in production mode

```
PORT=4001 MIX_ENV=prod mix phx.server
PORT=4001 MIX_ENV=prod iex -S mix phx.server
```

## Using distillery

- add dependencies

```
defp deps do
  [{:distillery, "~> 1.5", runtime: false}]
end
```

- initialize release

```
mix release.init
```

- modify endpoint as necessary

```
  config :myapp_api, MyappApiWeb.Endpoint,
    load_from_system_env: true,
M    url: [host: "api.myapp.com", port: 50080],
+    server: true,
+    root: ".",
+    version: Mix.Project.config[:version]
```

- release

```
MIX_ENV=prod mix release --env=prod
```

- copy to deployment folder

```
cp _build/prod/rel/myapp_api/releases/0.0.1/myapp_api.tar.gz /opt/deploys/myapp_api/
```

- create user and database (postgresql)

```
CREATE USER myapp_api WITH
  LOGIN
  SUPERUSER
  CREATEDB
  NOCREATEROLE
  INHERIT
  NOREPLICATION
  CONNECTION LIMIT -1
  PASSWORD 'myapp_api';

CREATE DATABASE myapp_api
    WITH 
    OWNER = myapp_api
    ENCODING = 'UTF8'
    LC_CTYPE 'en_US.UTF-8'
    LC_COLLATE 'en_US.UTF-8'
    CONNECTION LIMIT = -1;

WITH ENCODING 'UTF8'  ;
```

- run migration script

```
bin/myapp_api migrate
```

- run the server

```
PORT=50080 bin/myapp_api foreground
```

## References

- https://github.com/bitwalker/distillery
- https://hexdocs.pm/distillery/walkthrough.html
- https://hexdocs.pm/distillery/use-with-phoenix.html
- https://hexdocs.pm/distillery/phoenix-walkthrough.html
- https://hexdocs.pm/distillery/running-migrations.html
