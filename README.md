# Exzeitable

[![Build Status](https://github.com/alanvardy/exzeitable/workflows/Elixir%20ex_check/badge.svg)](https://github.com/alanvardy/exzeitable) [![Build Status](https://github.com/alanvardy/exzeitable/workflows/Elixir%20Dialyzer/badge.svg)](https://github.com/alanvardy/exzeitable) [![hex.pm](http://img.shields.io/hexpm/v/exzeitable.svg?style=flat)](https://hex.pm/packages/exzeitable)

Dynamic searchable, sortable datatable that takes a database query and a module, and makes the magic happen. Uses Phoenix Liveview and Postgres. Bootstrap friendly and easily configured for other CSS frameworks.
  
![Exzeitable](assets/screenshot.png)

## Installation

This package requires a Postgres database and Phoenix.

The package can be installed by adding [exzeitable](https://github.com/alanvardy/exzeitable) and [Phoenix Live View](https://github.com/phoenixframework/phoenix_live_view) to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [
    {:exzeitable, "~> 0.1.0"}
  ]
end
```

Documentation can be found at [https://hexdocs.pm/exzeitable](https://hexdocs.pm/exzeitable).


## Getting Started

### Module

Add the boilerplate to a new module.

```elixir
defmodule YourAppWeb.Live.File do
  @moduledoc "User's File table"
  alias YourAppWeb.Router.Helpers, as: Routes
  import Ecto.Query

  use Exzeitable,
    # Required
    repo: YourApp.Repo,
    routes: Routes,
    path: :file_path,
    query: from(f in File)
    fields: [
      image: [virtual: true],
      title: [hidden: true],
      description: [hidden: true],
      category: [hidden: true],
      filesize: [hidden: true, function: true, search: false],
      inserted_at: [hidden: true, function: true, label: "Created"],
      updated_at: [hidden: true, function: true, label: "Updated"]
    ],
    
    # Optional
    debounce: 300

  # The callback that renders your table
  def render(assigns), do: ~L"<%= build_table(assigns) %>"

  # Field functions, called when virtual: true or function: true
  def image(socket, file) do
    img_tag(file.url, class: "w-100")
    |> link(to: Routes.file_path(socket, :show, file))
  end
  
  def filesize, do: ...
  def inserted_at, do: ...
  def updated_at, do: ...

```

Options can be added to either your module (as seen above), or in the template (As seen below) or both.
If an option is defined in both the template option will replace the module option. The only exception is `:fields` which must be specified in the module.

#### Required options

  - `repo` The module for your repository. Example: `YourApp.Repo`
  - `routes` Your route module. Example: `YourAppWeb.Router.Helpers`
  - `path` The base path for your resource. Example: `:site_path`
  - `query` A Ecto.Query struct, the part before you give it to the Repo. Example: `from(s in Site, preload: [:users`

#### Defining your fields

Under the fields key, you can define a keyword list of atoms with keyword values. The map holds the options for that field.

```elixir
fields: [
          name: [function: true],
          age: [order: false],
          metadata: [label: "Additional Information", virtual: true, hidden: true],
        ]
```

The following field options are available (with their defaults):

- `label: nil` Set a custom string value for the column heading
- `function: false` Pass (socket, entry) to a function with the same name as the field
- `hidden: false` Hide the column by default (user can click show button to reveal)
- `search: true` Do not include the column in searches
- `order: true` Do not allow the column to be sorted (hide the sort button)
- `virtual: false` This is shorthand for [function: true, search: false, order: false] and will override those settings. Intended for creating fields that are not database backed.



Optional... options (with defaults)

  - `action_buttons: [:new, :edit, :show, :delete]` A list of atoms representing action buttons avaliable for the user to use. This does not do authorization, the routes will still be available.
  - `per_page: 20` Integer representing number of entries per page.
  - `debounce: 300` Sets how many miliseconds between responding to user input on the search field.

#### Options for nested routes

If you dont know what this is then you likely don't need to worry about it. You can look at [The official docs](https://hexdocs.pm/phoenix/routing.html#nested-resources) if you would like to learn more.

If you define one of the below options you then need to define the other.

For this example we will be using the boring example of users and posts

```elixir
resources "/users", UserController do
  resources "/posts", PostController
end
```

The users Exzeitable does not need the 2 options below, but the posts Exzeitable does. Because all of its routes are different. The following will be needed to make the posts Exzeitable work:

- `belongs_to: :user`
- `parent: @user`

Make sure that you include the :user_id in your query.

You will need to pass the parent option in from the template.

### Controller

This is where you can specify the query that forms the default data of the table.
Everything the table does is with a subset of this data.

```elixir
query = from(f in Files)
render(conn, "index.html", query: query)
```

### Template

And call the table from your template

```elixir
<h1> My Awesome Files </h1>
<%= YourAppWeb.Live.File.live_table(@conn, query: @query, action_buttons: [:show, :edit]) %>
```

## Contributing

Contributions are very welcome! Open an issue or a pull request, the code, tests and documentation can always be improved and I welcome the help :)