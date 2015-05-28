---
layout: post
categories:
- elasticsearch
- ruby
- chewy
title: Elasticsearch Suggesters with Chewy
---

We recently started migrating a page with filtering over to [elasticsearch][elasticsearch]. One of the filtering options allows you to search for someone's name, to make this a nicer experience we added autocompletion for that name. To help with this we used the [chewy][chewy] gem.

Before we get started, this is a `prefix` autocompletion. So if you search for `compl` it will suggest `complete` and not `autocomplete`.

##### PersonIndex

``` ruby
class PersonIndex < Chewey::Index
  define_type Person do
    field :first_name
    field :last_name
    field :name_autocomplete, {
      value: -> {
        {
          input: [first_name, last_name],
          output: full_name,
          context: {
            group_id: group.id,
          },
        }
      },
      type: "completion",
      context: {
        group_id: {
          type: "category",
        },
      },
    }
  end
end
```

First we need to set up the index. With chewy we define `PersonIndex` and fill in fields we want the index to have.

To create a `completion` field we give it a special value that contains the input to match against and the output elasticsearch will give after matching. We can also specify a `context` because we wanted to limit the completion to certain groupings and not suggest everything.

We also define `type` as `completion` and let elasticsearch know that this field has a context of `group_id`.

##### Suggestions

``` ruby
PersonIndex.suggest({
  suggest: {
    text: "Eri",
    completion: {
      field: "name_autocomplete",
      context: {
        group_id: group.id,
      },
    },
  },
}).suggest
```

In order to get suggestions out of elasticsearch we use the `.suggest` method off of the index. The top level key is required and can be anything you want. It will match up with the output key.

Inside the top level key you pass in the text you want to complete against along with what field and any context. Elasticsearch will return JSON similar to below:

##### Suggestion JSON

``` json
{
   "_shards": {
      "total": 5,
      "successful": 5,
      "failed": 0
   },
   "suggest": [
      {
         "text": "Eri",
         "offset": 0,
         "length": 3,
         "options": [
            {
               "text": "Eric Oestrich",
               "score": 1
            }
         ]
      }
   ]
}
```

The `options` array will contain any records that matched against your text. The `text` inside of `options` will be the `output` you specified when creating the value for that record.

If you don't want to use chewy for suggestions, after setting up the index you can query elasticsearch with the `_suggest` route off of your index:

```
GET person/_suggest
{
  "suggest": {
    "text": "Eri",
    "completion": {
      "field": "name_autocomplete",
      "context": {
        "group_id": 1
      }
    }
  }
}
```

[elasticsearch]: https://www.elastic.co/
[chewy]: https://github.com/toptal/chewy
