---
layout: post
categories:
- elixir
- guardian
title: Generating a Guardian Secret Key
description: How to generate a secret key for use in Guardian an Elixir authentication package.
---

I've been working on a [Phoenix][phoenix] project at work and I added authentication for the API via [Guardian][guardian]. Guardian is easy to use except for setting up the secret key. It took a few tries to finally get a key that worked, and it wasn't easy to find out how.


### Generate the key

To start, generate a key in `iex -S mix`.

```elixir
jwk_384 = JOSE.JWK.generate_key({:ec, "P-384"})
JOSE.JWK.to_file("password", "file.jwk", jwk_384)
```

I placed this file in `priv/repo/dev.jwk` for development purposed. In staging/production I stick the file contents in an environment variable.

### Configure Guardian

Once the key is generated, configure the application to load it.

##### config/config.ex

```elixir
config :guardian, Guardian,
  allowed_algos: ["ES384"],
  issuer: "MytApp",
  ttl: { 30, :days },
  secret_key: fn ->
    secret_key = MyApp.config(Application.get_env(:my_app, :secret_key))
    secret_key_passphrase = MyApp.config(Application.get_env(:my_app, :secret_key_passphrase))

    {_, jwk} = secret_key_passphrase |> JOSE.JWK.from_binary(secret_key)
    jwk
  end,
  serializer: MyApp.GuardianSerializer
```

##### lib/my_app.ex

```elixir
defmodule MyApp do
  def config({:system, env}), do: System.get_env(env)
  def config(value), do: value
end
```

I use this set of functions to allow loading config via environment variables.

### Configure Environments

##### config/dev.exs

```elixir
config :my_app, :secret_key_passphrase, "password"
config :my_app, :secret_key, File.read!("priv/repo/dev.jwk")
```

Configure the `dev` environment.


##### config/prod.exs

```elixir
config :venu, :secret_key_passphrase, {:system, "SECRET_KEY_PASSPHRASE"}
config :venu, :secret_key, {:system, "SECRET_KEY"}
```

### Conclusion

I hope this helps better explain how to set up Guardian for your Phoenix project.

[phoenix]: http://www.phoenixframework.org/
[guardian]: https://github.com/ueberauth/guardian
