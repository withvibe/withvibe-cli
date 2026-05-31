# withvibe

CLI to **install and manage a [WithVibe](https://withvibe.dev) server stack**,
and to run a withvibe environment locally — clones the repos defined by the
env, boots its `docker-compose` stack, and opens it in VSCode.

Binary name: `withvibe`.

## Quickstart — install WithVibe on this machine

You need **Node.js ≥ 20** and **Docker ≥ 24** (with Docker Compose v2).

```bash
# 1. Install the CLI
npm install -g withvibe

# 2. Sanity-check your machine (docker, ports, disk, etc.)
withvibe doctor

# 3. Install WithVibe — one-click, all defaults
withvibe init -y
```

`withvibe init -y` picks the default install mode (pull images from the
registry), writes config to `~/.withvibe`, prompts only for the things that
*can't* be defaulted (your Anthropic API key), pulls images, and starts the
stack. When it's done it prints the URL — open it, sign in, you're running.

### Custom install (interactive)

Drop `-y` to be prompted for everything:

```bash
withvibe init
```

The interactive flow asks for: install mode, install directory, ports,
Anthropic API key, optional Google OAuth, optional Traefik (public HTTPS via
Let's Encrypt). Sensible defaults at every step; you can mash Enter through
most of them.

### Install modes

| Mode | When to use | Flag |
| --- | --- | --- |
| **from-registry** (default) | Easiest path. Pulls prebuilt images from `ghcr.io/withvibe`. | `--mode from-registry` |
| **from-bundle** | Air-gapped install. You have a `withvibe-deploy-X.Y.Z.tar.gz` bundle. | `--mode from-bundle --bundle-path <dir>` |
| **from-source** | You're hacking on the WithVibe monorepo and want to build locally. | `--mode from-source` |

### Common install options

```bash
withvibe init -y                              # one-click, all defaults
withvibe init                                 # interactive
withvibe init --mode from-bundle --bundle-path ./withvibe-deploy-0.2.5
withvibe init -d ~/withvibe-prod              # custom install directory
withvibe init --no-start                      # write config + pull images, don't boot
withvibe init -f                              # re-init (DESTROYS the existing database)
```

See `withvibe init --help` for the full flag list.

## Managing a running install

```bash
withvibe status            # what's up, what's not, plus the URLs
withvibe start             # docker compose up -d, gated on health
withvibe stop              # docker compose down (data preserved)
withvibe restart
withvibe logs              # tail all services
withvibe logs api          # one service
withvibe configure         # edit/add features (Traefik, OAuth, secrets, QA browser…)
withvibe upgrade           # postgres dump → refresh images → restart, with rollback
withvibe uninstall         # remove the stack (asks first; --keep-data to spare the DB)
```

## Using a withvibe install (env workflow)

Once a withvibe server is running (yours or your team's), this is how a
teammate uses it to spin up an environment on their laptop.

### `withvibe login`

Authorize this machine against a withvibe server. Stores a CLI token in your
OS keyring (or falls back to a config file).

```bash
withvibe login
withvibe login --force         # re-auth even if a token exists
withvibe --server <url> login  # point at a non-default server
```

### `withvibe env <envId>`

Pre-flight checks, GitHub-account confirmation, repo cloning, port
allocation, `docker compose up -d`, then VSCode opens on the workspace
folder.

```bash
withvibe env env_abc123
withvibe env env_abc123 --no-open   # skip VSCode launch
```

## How `withvibe env` works under the hood

1. **Preflight** — checks `git`, `docker`, `gh`, and `code` are on `PATH`.
2. **GitHub auth** — uses `gh auth status` to show which account git will use, with a prompt to switch.
3. **Repo clone** — clones each repo declared by the env into a per-env workspace folder.
4. **Ports** — allocates host ports and wires them into the compose project.
5. **`docker compose up -d`** — starts the env's services.
6. **VSCode** — opens the workspace folder.

## Configuration

| Flag / env var | Purpose | Default |
| --- | --- | --- |
| `--server <url>` / `WITHVIBE_SERVER` | Withvibe web server URL | `http://localhost:3000` |

## Build from source

```bash
git clone https://github.com/withvibe/withvibe-cli.git
cd withvibe-cli
npm install
npm run build
node dist/index.js --help
```

## Links

- Website / docs: <https://withvibe.dev>
- WithVibe server: <https://github.com/withvibe/withvibe>

## License

Apache License 2.0 — see [LICENSE](./LICENSE) and [NOTICE](./NOTICE).
