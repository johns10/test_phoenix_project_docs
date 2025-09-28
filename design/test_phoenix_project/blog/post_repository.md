# PostRepository

## Purpose
Provides data access layer for blog posts with user-scoped security, implementing CRUD operations and scope-based filtering to ensure users can only access their own posts.

## Public API
```elixir
@spec list_posts(Scope.t()) :: [Post.t()]
@spec get_post!(Scope.t(), id :: integer()) :: Post.t()
@spec create_post(Scope.t(), attrs :: map()) :: {:ok, Post.t()} | {:error, Ecto.Changeset.t()}
@spec update_post(Scope.t(), Post.t(), attrs :: map()) :: {:ok, Post.t()} | {:error, Ecto.Changeset.t()}
@spec delete_post(Scope.t(), Post.t()) :: {:ok, Post.t()} | {:error, Ecto.Changeset.t()}
@spec change_post(Scope.t(), Post.t(), attrs :: map()) :: Ecto.Changeset.t()
```

## Execution Flow

It does the thing

## Function Descriptions

### list_posts/1
Returns all posts for the scoped user using `Repo.all_by/2` with `user_id` filtering.

### get_post!/2
Retrieves a single post by ID, raising `Ecto.NoResultsError` if not found or not owned by scoped user. Uses `Repo.get_by!/2` with both `id` and `user_id` filters.

### create_post/2
Creates a new post with scope-validated changeset. The scope automatically sets `user_id` in the changeset to ensure proper ownership.

### update_post/3
Updates an existing post with ownership validation via pattern matching `post.user_id == scope.user.id` before applying changeset.

### delete_post/2
Deletes a post after verifying ownership through scope validation. Returns standard `{:ok, post}` tuple on success.

### change_post/3
Returns changeset for tracking post changes with ownership validation. Used for form generation and validation without persistence.

## Error Handling

### Validation Errors
- `{:error, %Ecto.Changeset{}}` - Invalid post attributes or validation failures
- Changeset contains field-specific error details for form rendering

### Not Found Errors
- `Ecto.NoResultsError` - Post doesn't exist or user lacks access (from `get_post!/2`)
- No distinction between non-existent and unauthorized access for security

### Authorization Errors
- `MatchError` - Ownership validation fails in update/delete operations
- Raises exception when `post.user_id != scope.user.id`

## Usage Patterns

### Context Integration
Repository functions are delegated from `TestPhoenixProject.Blog` context, maintaining clean separation between business logic and data access.

### Scope-First Security
All functions require `Scope` struct as first parameter, ensuring user context is always available for filtering and validation.

### Changeset Composition
Functions pass scope to `Post.changeset/3` for consistent user_id assignment and validation across all operations.

## Dependencies
- **TestPhoenixProject.Repo** - Database interface using custom `all_by/2` and `get_by!/2` functions
- **TestPhoenixProject.Blog.Post** - Post schema with scope-aware changeset
- **TestPhoenixProject.Accounts.Scope** - User scoping struct for authorization
- **Ecto.Query** - Query composition and database operations