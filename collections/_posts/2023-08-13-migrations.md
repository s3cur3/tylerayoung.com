---
title: A microframework for backfill migrations in Elixir's Ecto
layout: post
authors: ['tyler']
categories: ["Elixir", "Databases"]
excerpt: This is a tiny little to-purpose framework for data migrations we've used at Felt. It's based on David Bernheisel's outstanding [Safe Ecto Migrations](https://fly.io/phoenix-files/safe-ecto-migrations/) series, specifically his [Batching Deterministic Data](https://fly.io/phoenix-files/backfilling-data/#batching-deterministic-data) pattern for backfill migrations.
image: /assets/images/migrating-elephants.jpg
image_credit: Photo by <a href="https://unsplash.com/@sergiferrete" rel="nofollow">Sergi Ferrete</a> on <a href="https://unsplash.com/photos/YXwt-vJ3szA" rel="nofollow">Unsplash</a>
---

This is a tiny little to-purpose framework for data migrations we've used at Felt. It's based on David Bernheisel's outstanding [Safe Ecto Migrations](https://fly.io/phoenix-files/safe-ecto-migrations/) series, specifically his [Batching Deterministic Data](https://fly.io/phoenix-files/backfilling-data/#batching-deterministic-data) pattern for backfill migrations.

(I'm going to be speaking at greater length about this in [my ElixirConf 2023 talk](https://2023.elixirconf.com/presenters#speakers)... until then, this is probably a little cryptic.)

Here's what a simple migration looks like using the framework (really just a single [`Behaviour`](https://elixir-lang.org/getting-started/typespecs-and-behaviours.html#behaviours))—we'll break down each component of this below.

```elixir
defmodule MyApp.Repo.Migrations.BackfillOnboardingStep do
  use DataMigration
  
  @impl DataMigration
  def base_query do
    from(u in "users", 
      where: is_nil(u.onboarding_step),
      select: %{id: u.id}
    )
  end

  @impl DataMigration
  def config do
    %DataMigration.Config{batch_size: 100, throttle_ms: 1_000}
  end

  @impl DataMigration
  def migrate(results) do
    Enum.each(results, fn %{id: user_id} ->
      user_id
      |> Ecto.UUID.cast!()
      |> MyApp.Accounts.set_onboarding_step!()
    end)
  end
end
```

The first bit, the `base_query/0`, is the "guts" of the query we'll use to fetch the data to be migrated. In the above case, it's as simple as pulling out the IDs of the users in need of migration. In other cases, you might also pull out data to be used in the migration (by, say, joining on an [`assoc/3`](https://hexdocs.pm/ecto/3.10.3/Ecto.html#assoc/2) of the `users` table).

The next callback we implement, `config/0`, provides a few required configuration parameters that the migration runner will use to modify the query. The `batch_size` is the number of rows to fetch at a time from the database (applied as a [`limit/2`](https://hexdocs.pm/ecto/3.10.3/Ecto.Query.html#limit/2) to the `base_query/0`). The `throttle_ms` is the number of milliseconds to sleep after migrating each batch. By throttling, you can ensure the migration doesn't lock the database entirely and deny service to normal application traffic.

Finally, `migrate/1` is where all the magic happens. It receives the results from the `base_query/0` (a list of maps) and either migrates each row or throws an error, aborting the whole migration.

These migration files live by default in `priv/repo/data_migrations/*.exs`. Because of the way the behaviour hooks into the migration runner when you `use DataMigration` (described below), each script is self-executing exactly as you'd expect—you write a migration, then run `$ mix ecto.migrate --migrations-path priv/repo/data_migrations` to perform the migration.


## The `MigrationRunner`

So far so good. Now what does the runner that executes these migrations look like? 

As in the aforementioned [Batching Deterministic Data](https://fly.io/phoenix-files/backfilling-data/#batching-deterministic-data) template, the migration runner is a recursive function that fetches a batch of rows, runs the migration on them, sleeps for the configured amount of time, then fetches the next batch. It's also where we keep the `Config` struct that the migrations use to configure themselves.

```elixir
defmodule DataMigration.Runner do
  @moduledoc """
  Runs a `DataMigration`
  """
  import Ecto.Query
  alias DataMigration

  @spec run(module()) :: :ok | no_return()
  def run(migration_module) do
    throttle_change_in_batches(migration_module)
  end

  # The very first ID when sorting UUIDs in ascending order.
  # If you use integer IDs instead, this would be 0.
  @first_id Ecto.UUID.dump!("00000000-0000-0000-0000-000000000000")

  defp throttle_change_in_batches(migration_module, last_id \\ @first_id)
  defp throttle_change_in_batches(_migration_module, nil), do: :ok

  defp throttle_change_in_batches(migration_module, last_id) do
    %DataMigration.Config{batch_size: batch_size, throttle_ms: throttle_ms, repo: repo} =
      migration_module.config()

    query =
      migration_module.base_query()
      |> where([i], i.id > ^last_id)
      |> order_by([i], asc: i.id)
      |> limit(^batch_size)

    case repo.all(query, log: :info, timeout: :infinity) do
      # Occurs when no more elements match the query; the migration is done!
      [] ->
        :ok

      query_results ->
        migration_module.migrate(query_results)
        Process.sleep(throttle_ms)

        last_processed_id = List.last(query_results).id
        throttle_change_in_batches(migration_module, last_processed_id)
    end
  end
end
```

## The `DataMigration` behaviour

The behaviour itself is pretty bare bones. There are the three callbacks we've already seen in the sample migration, plus a teeny tiny bit of magic to automatically write the `up/0` and `down/0` functions that `Ecto.Migration` requires.


```elixir
defmodule DataMigration do
  @moduledoc """
  A behaviour implemented by our data migrations (generally backfills).
  
  Based on David Bernheisel's [template for deterministic backfills](https://fly.io/phoenix-files/backfilling-data/#batching-deterministic-data).
  """

  @callback config() :: DataMigration.Config.t()

  @doc """
  The core of the query you want to use to SELECT a map of your data.
  The `DataMigration.Runner` will take care of limiting this to a batch size, ordering
  it by row ID, and restricting it to rows you haven't yet handled.
  The query *must* select a map, and that map must have an `:id` key for the
  migration runner to reference as the last-modified row in your table.
  """
  @callback base_query() :: Ecto.Query.t()

  @doc """
  The callback to operate on a result set from your query.
  Implementers should `raise` an error if you're unable to process the batch.
  """
  @callback migrate([map]) :: :ok | no_return()

  defmacro __using__(_) do
    quote do
      use Ecto.Migration
      import Ecto.Query
      alias DataMigration

      @behaviour DataMigration

      @disable_ddl_transaction true
      @disable_migration_lock true

      @spec up :: :ok | no_return()
      def up do
        DataMigration.Runner.run(__MODULE__)
      end

      @spec down :: :ok
      def down, do: :ok
    end
  end
end
```

Finally, there's that `Config` struct that the migrations use to set their parameters. (Note that I've used the delightful [`TypedStruct`](https://github.com/ejpcmac/typed_struct) library here to combine the struct definition with the typespec, but you could forego that if it annoys you.)

```elixir
defmodule DataMigration.Config do
  use TypedStruct

  typedstruct enforce: true do
    @typedoc """
    Configuration for a `DataMigration` behaviour module, used by the `DataMigration.Runner`.
    Specified the batch size (how many elements from your table to migrate at a time)
    and throttle time in milliseconds (the amount of downtime the runner should sleep
    between batches).
    """
    field :batch_size, integer
    field :throttle_ms, integer
    field :repo, atom, default: MyApp.Repo
  end
end
```

And there you have it!
