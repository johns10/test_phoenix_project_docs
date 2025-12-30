# TestPhoenixProject.Blog.PostCache

**Type**: state

In-memory cache for blog posts to improve read performance.

## Functions

### start_link/1

Starts the cache process.

```elixir
@spec start_link(keyword()) :: GenServer.on_start()
```

**Process**:
1. Initialize GenServer with empty cache state
2. Register process with module name
3. Return start result

**Test Assertions**:
- starts the cache process
- registers process with module name

### store/2

Stores a post in the cache.

```elixir
@spec store(integer(), Post.t()) :: :ok
```

**Process**:
1. Accept post ID and post struct
2. Store post in cache state keyed by ID
3. Return :ok

**Test Assertions**:
- caches a post by ID
- overwrites existing cached post with same ID

### get/1

Retrieves a post from the cache.

```elixir
@spec get(integer()) :: {:ok, Post.t()} | {:error, :not_found}
```

**Process**:
1. Look up post by ID in cache state
2. Return {:ok, post} if found
3. Return {:error, :not_found} if not found

**Test Assertions**:
- returns cached post when it exists
- returns error tuple when post not cached

### delete/1

Removes a post from the cache.

```elixir
@spec delete(integer()) :: :ok
```

**Process**:
1. Remove post with given ID from cache state
2. Return :ok regardless of whether post existed

**Test Assertions**:
- removes cached post
- returns :ok when post not found

### clear/0

Clears all posts from the cache.

```elixir
@spec clear() :: :ok
```

**Process**:
1. Reset cache state to empty map
2. Return :ok

**Test Assertions**:
- removes all cached posts

## Dependencies

- TestPhoenixProject.Blog.Post
- GenServer
