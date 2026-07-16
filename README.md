# conn

A tiny connection registry + credential helper for homelab and ops work. One command to reach any server, database, or device you manage:

```
conn go db-prod -Q "SELECT TOP 10 * FROM orders"
conn go web-vm1 "df -h"
conn go living-room-tv on
```

## Design

- **Registry** (`~/.config/conn/registry.tsv`): plain TSV, one row per connection: `name type host port user notes`. Greppable, diffable, no lock-in.
- **Secrets**: never in the registry. Passwords live in [pass](https://www.passwordstore.org/) (GPG-encrypted) under `conn/<name>`. The script pulls them at connect time.
- **Launchers**: `conn go <name>` picks the right client by type: `ssh` (with sshpass when a password is stored), `postgres`, `mysql`, `mssql` (auto-adds encryption flags for Azure SQL), `redis`, `rdp`.
- **Plugin handlers**: unknown types fall back to `~/.config/conn/handlers/<type>.sh`, called with `host port user password notes [extra args...]`. Drop in a script, `chmod +x`, done, no edits to `conn` itself. See `handlers/wiz.sh.example` for a smart-bulb (WiZ UDP protocol) handler.

## Commands

```
conn list                                  show all connections
conn show <name>                           row + password
conn get <name> <field>                    host|port|user|type|notes|password
conn add <name> <type> <host> <port> <user> [notes]
conn rm <name>
conn go <name> [client args...]            launch the right client
```

## Conventions that pay off

- Path-style names nest nicely in the pass store: `db/azure/shop-dev`, `ssh/vps/web1`, `iot/local/tv`.
- Put `db=<dbname>` in the notes column and `conn go` passes it as the default database for SQL types.

## Requirements

`bash`, `pass` (with a GPG key), plus whichever clients you actually use (`sshpass`, `psql`, `mysql`, `sqlcmd`, `redis-cli`, `xfreerdp`). Install lazily: the script tells you what is missing.

## Install

```
curl -o ~/bin/conn https://raw.githubusercontent.com/JoviAndreas/conn/main/conn
chmod +x ~/bin/conn
```

## License

MIT
