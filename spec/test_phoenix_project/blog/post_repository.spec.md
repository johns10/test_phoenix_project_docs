# TestPhoenixProject.Blog.PostRepository

**Type**: logic

Provides data access layer for blog posts with user-scoped security. All operations are filtered by user ownership to ensure users can only access their own posts.

## Functions

### list_posts/1

Returns all posts owned by the scoped user.

```elixir
@spec list_posts(Scope.t()) :: [Post.t()]
```

**Process**:
1. Query posts filtering by user_id from scope
2. Return all matching posts

**Test Assertions**:
- list_posts/1 returns all posts for scoped user
- list_posts/1 returns empty list when user has no posts
- list_posts/1 does not return posts from other users

### get_post!/2

Retrieves a single post by ID for the scoped user, raising if not found or not owned.

```elixir
@spec get_post!(Scope.t(), id :: integer()) :: Post.t()
```

**Process**:
1. Query post by id and user_id from scope
2. Raise Ecto.NoResultsError if not found
3. Return the post

**Test Assertions**:
- get_post!/2 returns post when found and owned by scoped user
- get_post!/2 raises Ecto.NoResultsError when post not found
- get_post!/2 raises Ecto.NoResultsError when post owned by different user

### create_post/2

Creates a new post with automatic user ownership assignment.

```elixir
@spec create_post(Scope.t(), attrs :: map()) :: {:ok, Post.t()} | {:error, Ecto.Changeset.t()}
```

**Process**:
1. Build changeset with scope to set user_id
2. Insert post into database
3. Return {:ok, post} on success or {:error, changeset} on failure

**Test Assertions**:
- create_post/2 creates post with valid attributes
- create_post/2 sets user_id from scope
- create_post/2 returns error changeset for invalid attributes
- create_post/2 validates required fields

### update_post/3

Updates an existing post after verifying ownership.

```elixir
@spec update_post(Scope.t(), Post.t(), attrs :: map()) :: {:ok, Post.t()} | {:error, Ecto.Changeset.t()}
```

**Process**:
1. Verify post.user_id matches scope.user.id via pattern matching
2. Build changeset with scope and attributes
3. Update post in database
4. Return {:ok, post} on success or {:error, changeset} on failure

**Test Assertions**:
- update_post/3 updates post with valid attributes
- update_post/3 returns error changeset for invalid attributes
- update_post/3 raises MatchError when user does not own post

### delete_post/2

Deletes a post after verifying ownership.

```elixir
@spec delete_post(Scope.t(), Post.t()) :: {:ok, Post.t()} | {:error, Ecto.Changeset.t()}
```

**Process**:
1. Verify post.user_id matches scope.user.id via pattern matching
2. Delete post from database
3. Return {:ok, post} on success

**Test Assertions**:
- delete_post/2 deletes post when user owns it
- delete_post/2 raises MatchError when user does not own post

### change_post/3

Returns a changeset for tracking post changes without persistence.

```elixir
@spec change_post(Scope.t(), Post.t(), attrs :: map()) :: Ecto.Changeset.t()
```

**Process**:
1. Build changeset with scope and attributes
2. Return changeset for form generation and validation

**Test Assertions**:
- change_post/3 returns changeset for post
- change_post/3 includes validation errors in changeset

## Dependencies

- TestPhoenixProject.Repo
- post.spec.md
- TestPhoenixProject.Accounts.Scope
- Ecto.Query
