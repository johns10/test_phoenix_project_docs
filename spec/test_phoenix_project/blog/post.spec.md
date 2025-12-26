# TestPhoenixProject.Blog.Post

**Type**: schema

Represents blog posts with user ownership and publishing capabilities. Posts maintain a simple structure focused on content creation with user-scoped access control.

## Functions

### changeset/3

Creates a changeset for post creation and updates with scope-aware user assignment.

```elixir
@spec changeset(t(), map(), Scope.t()) :: Ecto.Changeset.t()
```

**Process**:
1. Cast user-provided attributes for title, content, and published fields
2. Validate that title, content, and published are present
3. Automatically set user_id from the provided user scope
4. Return the changeset

**Test Assertions**:
- changeset/3 casts title, content, and published fields
- changeset/3 validates title is required
- changeset/3 validates content is required
- changeset/3 validates published is required
- changeset/3 sets user_id from scope.user.id
- changeset/3 does not allow user_id in attributes

## Fields

| Field       | Type         | Required   | Description            | Constraints    |
| ----------- | ------------ | ---------- | ---------------------- | -------------- |
| id          | integer      | Yes (auto) | Primary key            | Auto-generated |
| title       | string       | Yes        | Post title for display |                |
| content     | string       | Yes        | Main post content      |                |
| published   | boolean      | Yes        | Publication status     | Default: false |
| user_id     | integer      | Yes        | Post owner reference   | Set via scope  |
| inserted_at | utc_datetime | Yes (auto) | Creation timestamp     | Auto-generated |
| updated_at  | utc_datetime | Yes (auto) | Last update timestamp  | Auto-generated |

## Dependencies

- Ecto.Schema
- Ecto.Changeset
