---
layout: post
categories:
- elixir
- ex_venture
title: Compiling External Resources with Elixir
description: Showing how to use external resources in Elixir
date: 2018-03-05 9:00AM EST
---

The other week I started playing around with the `@external_resource` tag in Elixir. I wanted to do something similar to a translations file for the admin panel in [ExVenture][exventure-github] for help text. I didn't want to default to a YAML file as I've heard the elixir community isn't thrilled with it, so I took a look around to see what I could do.

What I ended up with was a macro that could compile a text file into Elixir functions. It also live reloads with the Phoenix code reloader in development because of `@external_resource`, this is really cool to see working.

## Help File Format

This is the file format I wanted to end up with. Keys and values essentially separated by new lines.

```
room.ecology:
  Room ecology changes the color of the map "icon" in the map grid.

room.feature:
  Room features are appended to the end of a room's description.
```

You can see the full file [here](https://github.com/oestrich/ex_venture/blob/master/priv/help/web.help).

## Help Macro

The end result of the macro will give us an API as follows:

```elixir
iex> Web.Help.get("room.feature")
"Room features are appended to the end of a room's description."
```

This works via a macro ([full file here][macro]) that loads the file and compiles it into quoted functions. The import sections are copied below:

```elixir
defmacro __using__(file) do
  help_file = Path.join(:code.priv_dir(:ex_venture), file)
  {:ok, help_text} = File.read(help_file)
  quotes = generate_gets(help_text)
  quotes = Enum.reverse([default_get() | quotes])
  [external_resource(help_file) | quotes]
end

defp generate_gets(help_text) do
  help_text
  |> String.split("\n")
  |> Enum.reject(&(&1 == ""))
  |> Enum.map(&String.trim/1)
  |> convert_to_map()
  |> Enum.map(fn {key, val} ->
    quote do
      def get(unquote(key)) do
        unquote(val)
      end
    end
  end)
end

defp external_resource(help_file) do
  quote do
    @external_resource unquote(help_file)
  end
end
```

The `__using__` macro reads the file and generates the get quotes and the external resource quote. The `external_resource/1` function is how the code live reloads when editing the text file. The `generate_gets/1` function parses the help file to generate a list of quoted functions, something I've never tried before. It was pretty cool to see you can return a list of quoted functions and get the same result.

## Conclusion

It is really cool to see the live reloading work. This may or may not be the best way to do what I want, but I had a lot of fun writing this. I think if these help files grow to be hundreds of keys long I will want to change up how this is parsing, but for now it works out well.

Lastly, you can check out more ExVenture at [exventure.org][exventure] and see my running instance at [midmud.com][midmud].

[exventure]: http://exventure.org
[exventure-github]: https://github.com/oestrich/ex_venture
[midmud]: https://midmud.com
[macro]: https://github.com/oestrich/ex_venture/blob/master/lib/ex_venture/text_compiler.ex
