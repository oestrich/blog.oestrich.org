---
layout: post
categories:
- java
- hypermedia
title: Java Hypermedia Client
---

Recently I've been working on an Android app that uses a hypermedia API. This has presented some challenges because the JSON library I'm using turns JSON into java objects. Using a HashMap is out because you get into generic hell land.

Up until now I haven't had to make a hypermedia client in anything but ruby. In ruby it's very simple because the JSON you receive from the server gets turned into a hash. I think I have come across a nice way to get around it in java though.

Below are my java objects that [Jackson](https://github.com/FasterXML/jackson-core) parses into.

##### Link
```java
public class Link {
    @JsonProperty("href")
    public String href;
}
```

##### HalRoot
```java
public class HalRoot {
    @JsonProperty("_links")
    protected RootLinks links;

    public String getSelfLink() {
        return links.self.href;
    }

    public String getOrdersLink() {
        return links.orders.href;
    }

    public class RootLinks {
        @JsonProperty("self")
        Link self;
        @JsonProperty("http://example.com/rels/orders")
        Link orders;
    }
}
```

##### HalOrders
```java
public class HalOrders {
    @JsonProperty("_embedded")
    protected Embedded embedded;

    @JsonProperty("_links")
    protected Links links;

    public List<HalOrder> orders() {
        return embedded.orders;
    }

    public class Embedded {
        @JsonProperty("orders")
        List<HalOrder> orders;
    }

    public class Links {
        @JsonProperty("self")
        Link self;
    }
}
```

##### HalOrder
```java
public class HalOrder {
    @JsonProperty("_embedded")
    protected Embedded embedded;

    @JsonProperty("_links")
    protected Links links;

    public List<HalItem> getItems() {
        return embedded.items;
    }

    public String getSelfLink() {
        return links.self.href;
    }

    public String getItemsLink() {
        return links.items.href;
    }

    public class Embedded {
        @JsonProperty("items")
        List<HalItem> items;
    }

    public class Links {
        @JsonProperty("self")
        Link self;
        @JsonProperty("http://nerdwordapp.com/rels/items")
        Link items;
    }
}
```

##### HalItem
```java
public class HalItem {
    @JsonProperty("name")
    public String name;
}
```
