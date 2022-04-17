---
title: Shooting Yourself in the Foot with GenServers
date: 2021-12-02
layout: post
categories: ["Programming", "Elixir"]
authors: ['tyler']
image: "/assets/images/2018-12-09-17.36.58-benjamin-finger-in-icing.jpg"
---

Elixir’s GenServers are great. Their fault tolerance makes them a natural choice for situations where you need to store some state over time in a resilient way. They’re not without their gotchas, though. In particular, it’s quite easy to fall into traps with respect to scheduling work within the GenServer’s process.

Consider this toy example:

```elixir
defmodule Greeter do
  use GenServer

  def start_link(opts) do
    name = Access.get(opts, :name, __MODULE__)
    GenServer.start_link(__MODULE__, name, name: name)
  end

  def greet(server \\ __MODULE__, name) do
    GenServer.call(server, {:greet, name})
  end

  @impl GenServer
  def init(server_name) do
    :timer.send_interval(20_000, self(), :sleep)
    {:ok, server_name}
  end

  # Depending on where we are in the :sleep handler, we'll time out
  # before getting into the body of this!
  @impl GenServer
  def handle_call({:greet, name}, _from, server_name) do
    {:reply, "Hello #{name}, I'm #{server_name}", server_name}
  end

  @impl GenServer
  def handle_info(:sleep, server_name) do
    Process.sleep(10_000)
    {:noreply, server_name}
  end
end
```

If you call `Greeter.greet()` while the `:sleep` message is being handled in the “background,” you’ll either get:

*   a much slower response than you might have expected (if you call it with less than 5 seconds left in the sleep), or
*   a timeout after 5 seconds for what _should_ have been a very simple, low-computational overhead request.

Now, obviously that’s a _super_ artificial example, and no one would write something like that. What about this one?

```elixir
defmodule Greeter2 do
  use GenServer

  # . . .

  @impl GenServer
  def handle_info(:sleep, server_name) do
    %{minute: minute} = DateTime.utc_now()

    timeout =
      if minute == 0 do
        60_000
      else
        250
      end

    Process.sleep(timeout)

    {:noreply, server_name}
  end
end
```

This is even _more_ brain-dead!

But… you actually see code that’s equivalent to this all the time in the wild. The `:sleep` handler is what it looks like when you interact with a remote API that is _usually_ fast, but occasionally times out. (It’s no coincidence I have it timing out at the top of every hour—I’ve worked with a few different government APIs that refresh their data hourly, and for the first couple minutes of the hour requests to those endpoints often just time out.)

A couple other ways this can occur:

*   Doing a database query within your GenServer’s process (in response to a `handle_call/3` for instance)—it’ll be fine _most_ of the time, but when the database gets overloaded, you could see cascading timeouts.
*   Doing computations that are usually cheap, but with juuuuuust the right data sets (or worse, the typical data set of your heaviest users) will start taking substantial amounts of time.
*   Making requests or doing computations that are actually all reasonably fast (say, 100 ms), but which can pile up within your GenServer under the right conditions. (If you get 51 requests queued up that each take 100 ms, your 52nd request will crash with a 5 second timeout.)

The fun thing about all these examples is that restarting the GenServer probably won’t help, because you often (re-)schedule the same kind of work upon restart, and the GenServer will _keep_ crashing. (You may even be making the problem _worse_ with retries!) And of course if the GenServer keeps crashing in just the right way, either its Supervisor will give up and just let it die or you’ll bring down the whole Supervisor.

The right solution here really depends on your situation. The simplest fix (where it’s appropriate) is to spawn the sometimes-slow work into a `Task`, then send your GenServer process a message when it’s done. Other possibilities include:

*   Turning your single GenServer into a pool of GenServers
*   Setting up a pipeline for your long-running jobs (something like Oban or Broadway), such that you can remove those jobs from GenServers entirely
*   Using circuit breakers like `:fuse` to stop retrying operations that fail repeatedly to bring down the overall load on the system