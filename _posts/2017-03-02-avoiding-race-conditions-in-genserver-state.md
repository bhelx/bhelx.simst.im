---
layout: post
title: Avoiding race conditions in GenServer
category: articles
tags: [elixir, OTP, concurrency]
---

Suppose we are making a multi-player game. We are choosing to store the state of the game in
a struct (`Game`). The game will start without any players and players will
individually choose to join the game. We've determined that there must be a maximum number of players
(`@max_players`) and the players will be represented by a list of strings (player nicknames).

{% highlight elixir %}
defmodule Game do
  @max_players 4
  defstruct players: []

  def add_player(game, nick) do
    Map.update!(game, :players, &([nick | &1]))
  end

  def max_players?(%{players: ps}) when length(ps) < @max_players, do: false
  def max_players?(_game), do: true
end
{% endhighlight %}

Let's try this out in `iex`.

{% highlight elixir %}
iex(1)> game = %Game{}
%Game{players: []}
iex(2)> game = Game.add_player(game, "ben")
%Game{players: ["ben"]}
iex(3)> game = Game.add_player(game, "harry")
%Game{players: ["harry", "ben"]}
iex(4)> game = Game.add_player(game, "ralph")
%Game{players: ["ralph", "harry", "ben"]}
iex(5)> Game.max_players?(game)
false
iex(6)> game = Game.add_player(game, "tom")
%Game{players: ["tom", "ralph", "harry", "ben"]}
iex(7)> Game.max_players?(game)
true
{% endhighlight %}

Seems to be working! Now we need a way to wrap this in a `process`. `OTP` provides a number of ways
to do this. In this case we are going to use a GenServer. Each game will get it's own
isolated process which can be identified by a `PID`. Let's create our GenServer which we will call `GameServer`.

{% highlight elixir %}
defmodule GameServer do
  use GenServer

  def start_link do
    GenServer.start_link(__MODULE__, %Game{})
  end

  def get_state(pid) do
    GenServer.call(pid, :get_state)
  end

  def add_player(pid, nick) do
    game = get_state(pid)
    if Game.max_players?(game) do
      {:error, :max_players}
    else
      GenServer.call(pid, {:add_player, nick})
    end
  end

  # Callbacks

  def handle_call(:get_state,  _from, game) do
    {:reply, game, game}
  end

  def handle_call({:add_player, nick}, _from, game) do
    {:reply, :ok, Game.add_player(game, nick)}
  end
end
{% endhighlight %}

We have two calls here

  1. `get_state` returns the current game state.
  2. `add_player` returns `{:error, reason}` if the player couldn't be added and `:ok` otherwise.

Let's try this out in iex now:

{% highlight elixir %}
iex(1)> {:ok, pid} = GameServer.start_link
{:ok, #PID<0.91.0>}
iex(2)> GameServer.add_player(pid, "ben")
:ok
iex(3)> GameServer.add_player(pid, "harry")
:ok
iex(4)> GameServer.add_player(pid, "ralph")
:ok
iex(5)> GameServer.add_player(pid, "tom")
:ok
iex(6)> GameServer.add_player(pid, "richard")
{:error, :max_players}
{% endhighlight %}

### A hidden race condition

This seems to be working in the repl, but if you deployed this into production you may find that every
once in a while a 5th or even 6th player will sneak in. This code has a race condition! Let's prove this by adding
a temporary function to `GameServer` which adds `n` players simultaneously:

{% highlight elixir %}
defmodule GameServer do
  # ...
  def add_players_simultaneously(pid, nicks) do
    Enum.each(nicks, fn nick ->
      Task.async(fn ->
        GameServer.add_player(pid, nick)
      end)
    end)
  end
  # ...
end
{% endhighlight %}

This function takes a list of nicks and calls `add_player` simultaneously from one process per nick.
Let's try it out with 5 players attempting to join.

{% highlight elixir %}
iex(1)> {:ok, pid} = GameServer.start_link
{:ok, #PID<0.100.0>}
iex(2)> nicks =  ["ben", "harry", "ralph", "tom", "richard"]
["ben", "harry", "ralph", "tom", "richard"]
iex(3)> GameServer.add_players_simultaneously(pid, nicks)
:ok
iex(4)> GameServer.get_state(pid)
%Game{players: ["richard", "tom", "ralph", "harry", "ben"]}
{% endhighlight %}

Wait, how did `richard` get in there?

The erlang process model is a great way to handle concurrent operations on state and
if done right it's quite hard to mess up your state in this way. You just need to think a
little up front when you are mutating state.

In order to demonstrate how this is happening, let's add some `puts` into our `GameServer.add_player` function.

{% highlight elixir %}
defmodule GameServer do
  # ...
  def add_player(pid, nick) do
    game = get_state(pid)
    IO.puts("Players in game: #{inspect game.players}")
    if Game.max_players?(game) do
      {:error, :max_players}
    else
      IO.puts("Add player #{nick}")
      GenServer.call(pid, {:add_player, nick})
    end
  end
  # ...
end
{% endhighlight %}

And when we run again.

{% highlight elixir %}
iex(1)> {:ok, pid} = GameServer.start_link
{:ok, #PID<0.119.0>}
iex(2)> nicks =  ["ben", "harry", "ralph", "tom", "richard"]
["ben", "harry", "ralph", "tom", "richard"]
iex(3)> GameServer.add_players_simultaneously(pid, nicks)
Players in game: []
Players in game: []
Players in game: []
Players in game: []
Players in game: []
:ok
Add player ben
Add player harry
Add player ralph
Add player tom
Add player richard
{% endhighlight %}

This is a classic **check-then-act** race condition. It's important to remember that the way GenServers work is
they operate on each message in their mailbox one at a time and in the order they are received.
The 5 `get_state` messages come in and the server processes those and returns the current state. Then every player
sees a stale state and 5 new messages come in to add the players to the game state.

In order to fix this, we need to make `add_player` an [atomic operation](http://wiki.osdev.org/Atomic_operation).
GenServer makes this easy for us because the process model ensures that everything that happens in the callback is atomic!
So all we need to do is move this logic into the `add_player` callback.

{% highlight elixir %}
defmodule GameServer do
  # ...

  # Removed the max_players? check
  def add_player(pid, nick) do
    GenServer.call(pid, {:add_player, nick})
  end

  # ...

  # Checking inside the callback now
  def handle_call({:add_player, nick}, _from, game) do
    if Game.max_players?(game) do
      {:reply, {:error, :max_players}, game}
    else
      {:reply, :ok, Game.add_player(game, nick)}
    end
  end

  # ...
end
{% endhighlight %}

Trying again now.

{% highlight elixir %}
iex(1)> {:ok, pid} = GameServer.start_link
{:ok, #PID<0.137.0>}
iex(2)> nicks =  ["ben", "harry", "ralph", "tom", "richard"]
["ben", "harry", "ralph", "tom", "richard"]
iex(3)> GameServer.add_players_simultaneously(pid, nicks)
:ok
iex(4)> GameServer.get_state(pid)
%Game{players: ["ralph", "tom", "harry", "ben"]}
{% endhighlight %}

Now our state is consistent like we'd expect! When richard tried to join he would see the `max_players` error.
The nice thing about the semantics of GenServer is that you can think about your code in terms of `clients` and `servers`.
All public API functions on the top of the module happen in the calling process or **client side**.
All `handle_*` callbacks happen in the server process or **server side**.
If you maintain this mental model it becomes obvious which side your code belongs in.

### On bottlenecks

While you may be tempted to throw everything in the server side for safety,
you need to consider the performance implications of doing so.
When writing callbacks you should be asking yourself

`What are the minimal operations I need to determine the next state?`

It's important to remember that callbacks can only happen one at a time so they can easily become a bottleneck. Be
careful about performing long operations (particularly network operations) in the callbacks if you can get away with
doing them somewhere else.

### A holistic approach

Although this works as an example on GenServers and state, I'd suggest that this problem has a deeper, root cause. There is a design flaw
with the `Game` datastructure. When designing a new datastructure, it is a good idea to guard it from ever getting
into an invalid state. The logic of keeping the structure consistent belongs in the module itself. You should then encourage
the programmers to only use that interface to manipulate the structure and return descriptive errors when something is wrong.
In this case, I might change the interface of the `Game` module instead to something like this:

{% highlight elixir %}
defmodule Game do
  @max_players 4
  defstruct players: []

  def add_player(game, nick) do
    if max_players?(game) do
      {:error, :max_players}
    else
      {:ok, Map.update!(game, :players, &([nick | &1]))}
    end
  end

  defp max_players?(%{players: ps}) when length(ps) < @max_players, do: false
  defp max_players?(_game), do: true
end
{% endhighlight %}

This will leave your GenServers pretty bare but I believe this is a good thing. You can consider something like
[exactor](https://github.com/sasa1977/exactor) to make them even smaller.
