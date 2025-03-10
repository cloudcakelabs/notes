# Database

## Sybase

- Show all databases

```sh
sp_helpdb
```

- Show a single database

```sh
sp_helpdb N'<db_name>'
```

- Get all tables

```sh
sp_tables '%'
```

- Filter by database

```sh
sp_tables '%', '%', 'master', "'TABLE'"
```

- Get all Sybase procedures

```sh
sp_stored_procedures '%'
```

- Filter by owner and db

```sh
sp_stored_procedures '%', 'dbo', 'master'
```
