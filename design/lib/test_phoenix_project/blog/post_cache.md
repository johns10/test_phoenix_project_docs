# PostCache

## Purpose
Provides in-memory ETS-based caching for blog posts with user-scoped access control, improving read performance while maintaining security boundaries and cache invalidation on post mutations.

## Architecture Overview

### ETS Table Structure
```elixir
defmodule TestPhoenixProject.Blog.PostCache do
  use GenServer
  
  @table_name :post_cache
  @type cache_key :: {user_id :: integer(), post_id :: integer()}
  @type cache_value :: %{post: Post.t(), inserted_at: DateTime.t()}
end
```

### Key Design Decisions
- **User-scoped keys**: Cache keys include user_id to prevent cross-user data leakage
- **GenServer supervision**: Manages ETS table lifecycle and provides serialized access
- **TTL-based expiration**: Automatic cleanup of stale entries
- **Write-through pattern**: Cache updates happen synchronously with database writes

## Cache Key Strategy

### Primary Cache Keys
```elixir
# Individual post lookup
{user_id, post_id} -> %{post: %Post{}, inserted_at: DateTime.t()}

# User's post list (optional optimization)
{user_id, :all_posts} -> %{posts: [%Post{}], inserted_at: DateTime.t()}
```

### Security Considerations
- All cache keys must include user_id as first element
- Cache lookups validate user_id matches scope.user.id
- No shared cache entries between users to prevent data leakage

## Cache Operations

### Read Operations
```elixir
@spec get_post(Scope.t(), post_id :: integer()) :: {:ok, Post.t()} | :miss
def get_post(%Scope{} = scope, post_id) do
  case :ets.lookup(@table_name, {scope.user.id, post_id}) do
    [{_key, %{post: post, inserted_at: inserted_at}}] ->
      if cache_fresh?(inserted_at), do: {:ok, post}, else: :miss
    [] -> :miss
  end
end

@spec list_posts(Scope.t()) :: {:ok, [Post.t()]} | :miss
def list_posts(%Scope{} = scope) do
  case :ets.lookup(@table_name, {scope.user.id, :all_posts}) do
    [{_key, %{posts: posts, inserted_at: inserted_at}}] ->
      if cache_fresh?(inserted_at), do: {:ok, posts}, else: :miss
    [] -> :miss
  end
end
```

### Write Operations
```elixir
@spec put_post(Scope.t(), Post.t()) :: :ok
def put_post(%Scope{} = scope, %Post{} = post) do
  key = {scope.user.id, post.id}
  value = %{post: post, inserted_at: DateTime.utc_now()}
  :ets.insert(@table_name, {key, value})
  invalidate_user_list(scope.user.id)
  :ok
end

@spec invalidate_post(Scope.t(), post_id :: integer()) :: :ok
def invalidate_post(%Scope{} = scope, post_id) do
  :ets.delete(@table_name, {scope.user.id, post_id})
  invalidate_user_list(scope.user.id)
  :ok
end
```

## Integration with Repository Layer

### Cache-Aside Pattern
```elixir
# In BlogRepository
def get_post!(%Scope{} = scope, id) do
  case PostCache.get_post(scope, id) do
    {:ok, post} -> 
      post
    :miss -> 
      post = Repo.get_by!(Post, id: id, user_id: scope.user.id)
      PostCache.put_post(scope, post)
      post
  end
end

def create_post(%Scope{} = scope, attrs) do
  with {:ok, post} <- create_post_in_db(scope, attrs) do
    PostCache.put_post(scope, post)
    {:ok, post}
  end
end
```

### Invalidation Strategy
- **Single post updates**: Invalidate specific post cache entry
- **Post deletion**: Remove from cache immediately
- **User list cache**: Invalidate on any post mutation
- **Bulk operations**: Clear entire user cache scope

## GenServer Implementation

### State Management
```elixir
defmodule PostCache do
  use GenServer
  
  defstruct [:table_name, :ttl_seconds, :cleanup_interval]
  
  def start_link(opts \\ []) do
    GenServer.start_link(__MODULE__, opts, name: __MODULE__)
  end
  
  def init(opts) do
    table_name = Keyword.get(opts, :table_name, :post_cache)
    ttl_seconds = Keyword.get(opts, :ttl_seconds, 300) # 5 minutes
    cleanup_interval = Keyword.get(opts, :cleanup_interval, 60_000) # 1 minute
    
    :ets.new(table_name, [:set, :public, :named_table, read_concurrency: true])
    schedule_cleanup(cleanup_interval)
    
    {:ok, %__MODULE__{
      table_name: table_name,
      ttl_seconds: ttl_seconds,
      cleanup_interval: cleanup_interval
    }}
  end
end
```

### Supervision Tree Integration
```elixir
# In application.ex
children = [
  # ... other children
  {TestPhoenixProject.Blog.PostCache, [ttl_seconds: 600]}
]
```

## Performance Considerations

### Memory Management
- **TTL-based expiration**: Automatic cleanup prevents memory growth
- **Selective invalidation**: Only clear affected cache entries
- **Read concurrency**: ETS table configured for concurrent reads
- **Size limits**: Monitor cache size and implement LRU if needed

### Cache Hit Optimization
- **Warm cache strategy**: Pre-populate frequently accessed posts
- **Batch operations**: Cache user's full post list for list operations
- **Smart invalidation**: Preserve cache entries when possible

## Monitoring and Observability

### Metrics to Track
```elixir
# Cache performance metrics
:telemetry.execute([:post_cache, :hit], %{count: 1}, %{user_id: user_id})
:telemetry.execute([:post_cache, :miss], %{count: 1}, %{user_id: user_id})
:telemetry.execute([:post_cache, :eviction], %{count: 1}, %{reason: :ttl})

# Memory usage tracking
:telemetry.execute([:post_cache, :size], %{bytes: cache_size, entries: entry_count})
```

### Health Checks
- Cache hit ratio monitoring (target: >80%)
- Memory usage alerts
- Cache operation latency tracking
- Eviction rate monitoring

## Configuration Options

### Runtime Configuration
```elixir
config :test_phoenix_project, TestPhoenixProject.Blog.PostCache,
  ttl_seconds: 300,           # Cache entry lifetime
  cleanup_interval: 60_000,   # Cleanup process interval
  max_entries_per_user: 1000, # Prevent runaway memory usage
  enable_list_cache: true     # Cache user post lists
```

### Development vs Production
- **Development**: Shorter TTL for faster development cycles
- **Production**: Longer TTL, more aggressive cleanup
- **Test**: Synchronous cache operations for deterministic tests