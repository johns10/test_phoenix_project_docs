# TestPhoenixProject.Accounts

The Accounts context manages user authentication, registration, sessions, and email confirmations.

## Functions

### get_user_by_email/1

```elixir
@spec get_user_by_email(email :: String.t()) :: User.t() | nil
```

Gets a user by email.

### get_user_by_email_and_password/2

```elixir
@spec get_user_by_email_and_password(email :: String.t(), password :: String.t()) :: User.t() | nil
```

Gets a user by email and password.

### get_user!/1

```elixir
@spec get_user!(id :: integer()) :: User.t()
```

Gets a single user. Raises `Ecto.NoResultsError` if the User does not exist.

### register_user/1

```elixir
@spec register_user(attrs :: map()) :: {:ok, User.t()} | {:error, Ecto.Changeset.t()}
```

Registers a user.

### change_user_registration/2

```elixir
@spec change_user_registration(User.t(), attrs :: map()) :: Ecto.Changeset.t()
```

Returns an `%Ecto.Changeset{}` for tracking user changes.

### change_user_email/2

```elixir
@spec change_user_email(User.t(), attrs :: map()) :: Ecto.Changeset.t()
```

Returns an `%Ecto.Changeset{}` for changing the user email.

### apply_user_email/3

```elixir
@spec apply_user_email(User.t(), password :: String.t(), attrs :: map()) :: {:ok, User.t()} | {:error, Ecto.Changeset.t()}
```

Emulates that the email will change without persisting it.

### update_user_email/2

```elixir
@spec update_user_email(User.t(), token :: String.t()) :: :ok | :error
```

Updates the user email using the given token.

### deliver_user_update_email_instructions/3

```elixir
@spec deliver_user_update_email_instructions(User.t(), current_email :: String.t(), update_email_url_fun :: function()) :: {:ok, map()}
```

Delivers the update email instructions to the given user.

### change_user_password/2

```elixir
@spec change_user_password(User.t(), attrs :: map()) :: Ecto.Changeset.t()
```

Returns an `%Ecto.Changeset{}` for changing the user password.

### update_user_password/3

```elixir
@spec update_user_password(User.t(), password :: String.t(), attrs :: map()) :: {:ok, User.t()} | {:error, Ecto.Changeset.t()}
```

Updates the user password.

### generate_user_session_token/1

```elixir
@spec generate_user_session_token(User.t()) :: String.t()
```

Generates a session token.

### get_user_by_session_token/1

```elixir
@spec get_user_by_session_token(token :: String.t()) :: User.t() | nil
```

Gets the user with the given signed token.

### delete_user_session_token/1

```elixir
@spec delete_user_session_token(token :: String.t()) :: :ok
```

Deletes the signed token with the given context.

### deliver_user_confirmation_instructions/2

```elixir
@spec deliver_user_confirmation_instructions(User.t(), confirmation_url_fun :: function()) :: {:ok, map()}
```

Delivers the confirmation email instructions to the given user.

### confirm_user/1

```elixir
@spec confirm_user(token :: String.t()) :: {:ok, User.t()} | :error
```

Confirms a user by the given token.

### deliver_user_reset_password_instructions/2

```elixir
@spec deliver_user_reset_password_instructions(User.t(), reset_password_url_fun :: function()) :: {:ok, map()}
```

Delivers the reset password email to the given user.

### get_user_by_reset_password_token/1

```elixir
@spec get_user_by_reset_password_token(token :: String.t()) :: User.t() | nil
```

Gets the user by reset password token.

### reset_user_password/2

```elixir
@spec reset_user_password(User.t(), attrs :: map()) :: {:ok, User.t()} | {:error, Ecto.Changeset.t()}
```

Resets the user password.
