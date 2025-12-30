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
- start_link/1 starts the cache process
- start_link/1 registers process with module name

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
- store/2 caches a post by ID
- store/2 overwrites existing cached post with same ID

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
- get/1 returns cached post when it exists
- get/1 returns error tuple when post not cached

### delete/1

Removes a post from the cache.

```elixir
@spec delete(integer()) :: :ok
```

**Process**:
1. Remove post with given ID from cache state
2. Return :ok regardless of whether post existed

**Test Assertions**:
- delete/1 removes cached post
- delete/1 returns :ok when post not found

### clear/0

Clears all posts from the cache.

```elixir
@spec clear() :: :ok
```

**Process**:
1. Reset cache state to empty map
2. Return :ok

**Test Assertions**:
- clear/0 removes all cached posts

## Dependencies

- TestPhoenixProject.Blog.Post
- GenServer
