---
layout: post
categories:
- elixir
- ecto
title: Filtering Ecto with a Behaviour
description: Showing a behaviour and module for filtering an ecto query
date: 2018-02-02 9:00AM EST
---

For my project [ExVenture][exventure] I wanted to have a very nice admin interface and with that comes filtering of tables. This is what I came up with, which I think has been very extendable and worked out well.

This is what we'll end up with:

<div style="text-align: center">
  <img src="/images/ecto-filtering-end-result.png" />
</div>

## Web.Admin.ItemController

Let's first start at a controller to see what it looks like there. The controller is incredibly simple, pulling out the filter parameters and passing it into a web specific module, [Web.Item][item]. You can view the full file [here on github][item_controller].

```elixir
def index(conn, params) do
  %{page: page, per: per} = conn.assigns
  filter = Map.get(params, "item", %{})
  %{page: items, pagination: pagination} = Item.all(filter: filter, page: page, per: per)
  conn |> render("index.html", items: items, filter: filter, pagination: pagination)
end
```

This also includes my [pagination module][pagination], but I will be glossing over that part. We will see it again in the `Web.Item` module.

### Web.Item

Next let's look at the [Web.Item][item] module. This module contains the `all/1` function used above. It implements the Filter's behaviour that requires a single function `filter_on_attribute/2`.

The map of filters passed in from the controller is looped over and passed into this function of the module. Inside the `filter_on_attribute/2` function you can tweak the query to do whatever is needed to perform a filter based on that attributes.

```elixir
defmodule Web.Item do
  alias Data.Item
  alias Web.Filter

  @behaviour Filter

  @doc """
  Load all items
  """
  @spec all(Keyword.t()) :: [Item.t()]
  def all(opts \\ []) do
    opts = Enum.into(opts, %{})

    Item
    |> order_by([i], i.id)
    |> preload([:item_aspects])
    |> Filter.filter(opts[:filter], __MODULE__)
    |> Pagination.paginate(opts)
  end

  @impl Filter
  def filter_on_attribute({"level_from", level}, query) do
    query |> where([i], i.level >= ^level)
  end

  def filter_on_attribute({"level_to", level}, query) do
    query |> where([i], i.level <= ^level)
  end

  def filter_on_attribute({"tag", value}, query) do
    query |> where([n], fragment("? @> ?::varchar[]", n.tags, [^value]))
  end

  def filter_on_attribute(_, query), do: query
end
```

You can see here that I'm doing a fragment to look into an array of strings. You could also do [a join][npc-join] or other more complicated query manipulation.

## Web.Filter

Finally the [Web.Filter][filter] module itself. This is very simple, it defines a callback and a single function. The function also requires the using module to be passed in as the last argument. This is very simple with the `__MODULE__` macro.

```elixir
defmodule Web.Filter do
  @moduledoc """
  Filter an `Ecto.Query` by a map of parameters. Modules
  that use this should follow it's behaviour.
  """

  @doc """
  This will be reduced from the query params that are
  passed into `filter/3`.
  """
  @callback filter_on_attribute(
              {attribute :: String.t(), value :: String.t()},
              query :: Ecto.Query.t()
            ) :: Ecto.Query.t()

  @doc """
  Common elements of filtering a query
  """
  @spec filter(Ecto.Query.t(), map(), atom()) :: Ecto.Query.t()
  def filter(query, nil, _), do: query

  def filter(query, filter, module) do
    filter
    |> Enum.reject(&(elem(&1, 1) == ""))
    |> Enum.reduce(query, &module.filter_on_attribute/2)
  end
end
```

There are two other small things that this does, it pattern matches on a nil map and simply passes the query through. Lastly it filters out values that are an empty string since that is an empty form field.

## The Template

I won't go into detail about this, but you can view it [on github][item-index-template].

## Conclusion

This has worked out really well in crafting my admin panel. Adding a new filter is as simple as adding the template code and a single function, no extra magic needed.

[exventure]: https://github.com/oestrich/ex_venture
[filter]: https://github.com/oestrich/ex_venture/blob/master/lib/web/filter.ex
[item]: https://github.com/oestrich/ex_venture/blob/master/lib/web/item.ex
[item_controller]: https://github.com/oestrich/ex_venture/blob/master/lib/web/controllers/admin/item_controller.ex
[pagination]: https://github.com/oestrich/ex_venture/blob/master/lib/web/pagination.ex
[npc-join]: https://github.com/oestrich/ex_venture/blob/3832b895b3b73aa1a7547686e7d22dacd2888802/lib/web/npc.ex#L38-L41
[item-index-template]: https://github.com/oestrich/ex_venture/blob/3832b895b3b73aa1a7547686e7d22dacd2888802/lib/web/templates/admin/item/index.html.eex#L63
