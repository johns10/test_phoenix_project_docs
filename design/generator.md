# Phoenix Generator Commands

## Simple Boilerplate App Setup

### Create User Authentication
```bash
mix phx.gen.auth Accounts User users
```

### Generate Blog Post Resource
```bash
mix phx.gen.html Blog Post posts title:string content:text published:boolean user_id:references:users
```

### Generate Comment Resource
```bash
mix phx.gen.html Blog Comment comments content:text post_id:references:posts user_id:references:users
```

### Generate Category Resource
```bash
mix phx.gen.html Blog Category categories name:string description:text
```

## Run After Generation

```bash
mix deps.get
mix ecto.migrate
```

## Additional Setup Commands

```bash
# Generate API endpoints for posts
mix phx.gen.json Api Post posts title:string content:text published:boolean user_id:references:users

# Generate live view for real-time features
mix phx.gen.live Blog Post posts title:string content:text published:boolean user_id:references:users
```