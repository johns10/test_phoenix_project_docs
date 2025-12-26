# TestPhoenixProject.Blog

The Blog context manages blog posts including creation, updates, deletion, and real-time notifications.

## Functions

### subscribe_posts/1

```elixir
@spec subscribe_posts(Scope.t()) :: :ok | {:error, term()}
```

Subscribes to scoped notifications about any post changes.

The broadcasted messages match the pattern:
- `{:created, %Post{}}`
- `{:updated, %Post{}}`
- `{:deleted, %Post{}}`

**Test Assertions**:
- subscribes to post notifications
- receives created message when post created
- receives updated message when post updated
- receives deleted message when post deleted

### list_posts/1

```elixir
@spec list_posts(Scope.t()) :: [Post.t()]
```

Returns the list of posts for the given scope.

**Test Assertions**:
- returns all posts for scoped user
- filters posts by user scope

### get_post!/2

```elixir
@spec get_post!(Scope.t(), id :: String.t()) :: Post.t()
```

Gets a single post. Raises `Ecto.NoResultsError` if the Post does not exist.

**Test Assertions**:
- returns the post with given id
- raises when post does not exist

### create_post/2

```elixir
@spec create_post(Scope.t(), attrs :: map()) :: {:ok, Post.t()} | {:error, Ecto.Changeset.t()}
```

Creates a post and broadcasts a creation notification.

**Test Assertions**:
- creates a post with valid data
- broadcasts created notification
- returns error changeset with invalid data

### update_post/3

```elixir
@spec update_post(Scope.t(), Post.t(), attrs :: map()) :: {:ok, Post.t()} | {:error, Ecto.Changeset.t()}
```

Updates a post and broadcasts an update notification.

**Test Assertions**:
- updates the post with valid data
- broadcasts updated notification
- returns error changeset with invalid data

### delete_post/2

```elixir
@spec delete_post(Scope.t(), Post.t()) :: {:ok, Post.t()} | {:error, Ecto.Changeset.t()}
```

Deletes a post and broadcasts a deletion notification.

**Test Assertions**:
- deletes the post
- broadcasts deleted notification

### change_post/3

```elixir
@spec change_post(Scope.t(), Post.t(), attrs :: map()) :: Ecto.Changeset.t()
```

Returns an `%Ecto.Changeset{}` for tracking post changes.

**Test Assertions**:
- returns a post changeset
