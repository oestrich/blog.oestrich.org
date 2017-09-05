---
layout: post
categories:
- elixir
- phoenix
title: Adding Phoenix to an OTP Elixir App
description: How I added a Phoenix layer to a plain OTP Elixir app and how to have Phoenix talk to your OTP processes.
date: 2017-09-05 9:30AM
---

For my side project, [ex_venture][ex_venture], I wanted to add a web client that allowed players to connect not just via normal telnet. This meant I needed to add [Phoenix][phoenix]. I was excited to try this out because the phoenix devs keep saying how you should just think of it as a layer for your app, not the app itself. Since I had an app already this was the perfect trial.

Brief note: ex_venture is a [MUD][mud] engine, the protocol up until now was only telnet.

## Adding Phoenix

Adding phoenix was a pretty simple affair. It took about 2 hours to get it up and running in a simple form. [Here is the commit][phoenix_commit] that adds it entirely. I mostly copied from a new phoenix project and took what I wanted.

The fun part was the [TelnetChannel][telnet_channel] that talked with my [session][session] module similar to the normal [telnet socket][telnet].

## Phoenix <-> OTP communication

Another interesting part of this addition was I could add a web admin panel to the game. With this I wanted live updates to the game. If I added a new room to a zone, then I should see it reflected immediately in the game. I achieved this by creating bounded contexts that talk to the data layer then push the update into the OTP layer.

### Samples

#### Updating a zone

This shows off updating a zone and pushing the change live into the game. The controller knows nothing about the OTP. `Web.Zone` is a layer between Phoenix and the game.

##### Web.Admin.ZoneController

```elixir
alias Web.Zone

def update(conn, %{"id" => id, "zone" => params}) do
  case Zone.update(id, params) do
    {:ok, zone} -> conn |> redirect(to: zone_path(conn, :show, zone.id))
    {:error, changeset} ->
      zone = Zone.get(id)
      conn |> render("edit.html", zone: zone, changeset: changeset)
  end
end
```

##### Web.Zone

```elixir
alias Data.Zone

def update(id, params) do
  zone = id |> get()
  changeset = zone |> Zone.changeset(params)
  case changeset |> Repo.update do
    {:ok, zone} ->
      Game.Zone.update(zone.id, zone)
      {:ok, zone}
    anything -> anything
  end
end
```

##### Game.Zone

```elixir
# pid expands to the elixir Registry
def update(id, zone) do
  GenServer.cast(pid(id), {:update, zone})
end

def handle_cast({:update, zone}, state) do
  {:noreply, Map.put(state, :zone, zone)}
end
```

#### Adding a new room to a zone

This shows off how the Room admin will spawn a new room in a zone after being created. The controller knows nothing about OTP. `Web.Room` is a layer between Phoenix and the game.

##### Web.Admin.RoomController

```elixir
alias Web.Room

def create(conn, %{"zone_id" => zone_id, "room" => params}) do
  zone = Zone.get(zone_id)
  case Room.create(zone, params) do
    {:ok, room} -> conn |> redirect(to: room_path(conn, :show, room.id))
    {:error, changeset} -> conn |> render("new.html", zone: zone, changeset: changeset)
  end
end
```

##### Web.Room

```elixir
alias Data.Room

def create(zone, params) do
  changeset = zone |> Ecto.build_assoc(:rooms) |> Room.changeset(params)
  case changeset |> Repo.insert() do
    {:ok, room} ->
      Game.Zone.spawn_room(zone.id, room)
      {:ok, room}
    anything -> anything
  end
end
```

##### Game.Zone

When the [Room.Supervisor][room_supervisor] comes online it lets the zone know it's PID to spawn new rooms in.

```elixir
def spawn_room(id, room) do
  GenServer.cast(pid(id), {:spawn_room, room})
end

def handle_cast({:spawn_room, room}, state = %{room_supervisor_pid: room_supervisor_pid}) do
  Room.Supervisor.start_child(room_supervisor_pid, room)
  {:noreply, state}
end
```

##### Game.Room.Supervisor

The `Room.Supervisor` starts the new room in the supervision tree.

```elixir
def start_child(pid, room) do
  child_spec = worker(Room, [room], id: room.id, restart: :permanent)
  Supervisor.start_child(pid, child_spec)
end
```

## Conclusion

Adding Phoenix to an regular OTP app was incredibly simple and the Phoenix team did what they set out to. I hope you explore the [rest of the app][ex_venture] to find more examples of Phoenix <-> OTP communication.

[ex_venture]: https://github.com/oestrich/ex_venture
[phoenix]: http://phoenixframework.org/
[mud]: https://en.wikipedia.org/wiki/MUD
[phoenix_commit]: https://github.com/oestrich/ex_venture/commit/12952900e00eb730ceb94d26aafc3a06057c731c
[telnet_channel]: https://github.com/oestrich/ex_venture/blob/master/lib/web/channels/telnet_channel.ex
[session]: https://github.com/oestrich/ex_venture/blob/master/lib/game/session.ex
[telnet]: https://github.com/oestrich/ex_venture/blob/master/test/support/networking.ex
[room_supervisor]: https://github.com/oestrich/ex_venture/blob/master/lib/game/room/supervisor.ex
