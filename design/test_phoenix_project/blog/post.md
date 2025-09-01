# Post Schema

## Schema Structure

The Post schema represents blog posts with user ownership and publishing capabilities. It maintains a simple structure focused on content creation and user-scoped access control.

```elixir
defmodule TestPhoenixProject.Blog.Post do
  use Ecto.Schema
  import Ecto.Changeset

  @type t :: %__MODULE__{
    id: integer() | nil,
    title: String.t(),
    content: String.t(),
    published: boolean(),
    user_id: integer(),
    inserted_at: DateTime.t() | nil,
    updated_at: DateTime.t() | nil
  }

  schema "posts" do
    field :title, :string
    field :content, :string
    field :published, :boolean, default: false
    field :user_id, :id

    timestamps(type: :utc_datetime)
  end
end
```

## Field Descriptions

### Core Content Fields
- **title**: Post title, required string field for display purposes
- **content**: Main post content, required string field supporting rich text or markdown
- **published**: Publication status boolean with `default: false` to ensure posts start as drafts

### Ownership and Metadata
- **user_id**: Foreign key reference to user table, establishes post ownership for scoping
- **timestamps**: Standard `inserted_at` and `updated_at` fields using UTC datetime

## Changeset Function

The changeset implements scope-aware validation ensuring user ownership is automatically assigned:

```elixir
@spec changeset(t(), map(), Scope.t()) :: Ecto.Changeset.t()
def changeset(post, attrs, user_scope) do
  post
  |> cast(attrs, [:title, :content, :published])
  |> validate_required([:title, :content, :published])
  |> put_change(:user_id, user_scope.user.id)
end
```

### Validation Rules
- **Required fields**: title, content, and published status must be provided
- **Scope integration**: user_id is automatically set from the provided user scope
- **Cast fields**: Only title, content, and published are user-modifiable
- **Security**: user_id cannot be directly set by user input, only through scope

## Usage Patterns

### Creating Posts
```elixir
%Post{}
|> Post.changeset(attrs, scope)
|> Repo.insert()
```

### Updating Posts
```elixir
post
|> Post.changeset(updated_attrs, scope)
|> Repo.update()
```

## Business Rules

### Publication Status
- Posts default to unpublished (`published: false`) to support draft workflow
- Published status can be toggled through standard changeset operations
- No automatic publishing or scheduling logic implemented

### User Ownership
- Every post must belong to a user (non-nullable user_id)
- User ownership is immutable once set via scope
- Ownership validation handled at repository layer, not in changeset

### Content Constraints
- No length restrictions implemented on title or content fields
- No format validation for content (supports plain text, markdown, etc.)
- All string fields accept any valid UTF-8 content

## Relationship Patterns

### Loose Coupling Approach
The schema uses ID-only references rather than Ecto associations:
- **user_id** field instead of `belongs_to :user` association
- Reduces coupling between Blog and Accounts contexts
- Allows flexible user system changes without schema migrations
- Repository layer handles user validation and scope filtering

## State Transitions

### Publishing Workflow
```
Draft (published: false) -> Published (published: true)
Published (published: true) -> Draft (published: false)
```

No complex state machine - simple boolean toggle supports basic publishing needs.