# CLAUDE.md

## Repository

Desired state for `clickstack-vultr`: the ClickStack observability stack —
ClickHouse, MongoDB, the HyperDX OpenTelemetry collector, and the HyperDX UI —
on one Vultr instance in Amsterdam, published at
`https://clickstack.bigconfig.online` through Cloudflare and Caddy. Behavior
lives in `../clickstack`.

Tracked source is `colors.yml`, toolchain and documentation, the installed
Package Skill, lockfile, and a root launcher copied from its payload.
`.colors/` is generated private state and `.envrc.private` contains
credentials; never read, edit or commit either.

## The machine keypair is not in this checkout

This deployment runs in **keygen mode**: `colors.yml` carries no
`vultr-ssh-keys`, so the package owns the keypair per the workspace SSH Keypair
Standard. It lives at `~/.ssh/clickstack-vultr`(`.pub`) — outside this
repository — and the Vultr account key named `clickstack-vultr` belongs to this
deployment's OpenTofu state.

Consequences:

- Cloning this repository on another workstation does **not** carry machine
  access. Copy `~/.ssh/clickstack-vultr`(`.pub`) deliberately, or `create` will
  refuse rather than regenerate a key that cannot reach the live host.
- Never delete the Vultr key named `clickstack-vultr` while the instance lives.
- Adding `vultr-ssh-keys` to `colors.yml` switches to opt-out mode and would
  orphan the generated key; a live deployment changes modes only by rebuild.

## Commands

```sh
./green build
./green create --dry-run
./green create
./green delete
```

Build and dry-run require no credentials and never touch `~/.ssh`. Never export
`COLORS_PAR_PROFILE`. Keep `compute-prevent-destroy: true`; deletion requires
separate authorization and a one-run `COLORS_PAR_COMPUTE_PREVENT_DESTROY=false`
override.

## The ssh alias

Convergence writes a `~/.ssh/config` block per the workspace SSH Config
Standard, so `ssh clickstack-vultr` reaches the host with no address, user or
`-i` flag. Delete removes the block before destroying the instance.

The block is inserted at the top of `~/.ssh/config`. If that file ever grows an
option above its first `Host` line, `create` refuses rather than capturing a
global setting into this deployment's stanza; move it into a trailing
`Host *` stanza and retry.

## Credentials generated on the server

Three values live only on the host, all mode 0600 and none supplied here.
Convergence creates the initial HyperDX team for `clickstack-admin-email` —
the collector binds no OTLP receivers until one exists — and publishes that
team's ingestion key. `EXPRESS_SESSION_SECRET` is generated too, because
HyperDX otherwise signs session cookies with a constant published in its own
repository.

```sh
ssh clickstack-vultr 'cat /etc/clickstack/admin.env'
ssh clickstack-vultr 'sed -n s/HYPERDX_API_KEY=//p /etc/clickstack/ingestion.env'
```

Point any OTLP/HTTP exporter at `https://clickstack.bigconfig.online` with that
value as its `authorization` header.

All three are written under `creates:`, so a re-converge never rotates them. A
password changed in the UI makes `admin.env` stale and nothing corrects it.

## Documentation

`index.html` is this repository's landing page and carries two analytics tags:
GA4 measurement ID `G-4VKP1WY4QJ`, whose explicit `page_title` must exactly
equal the decoded HTML `<title>` and stay distinct and stable so one Analytics
property can separate repositories, and the self-hosted Rybbit snippet
`<script src="https://rybbit.getcolors.ai/api/script.js" data-site-id="9fb9c41a6d49" defer></script>`,
which shares one site ID across every page because `getcolors.github.io/<repo>/`
paths already encode the repository. Never add one tag without the other.

## Git

The root `green` is a copy, not a symlink. After a Package Skill update run
`npx skills update -p -y` and copy
`.agents/skills/package-clickstack-green/green` over it. Never hand-edit its
SHA.

Work on the current branch. Do not commit or push unless explicitly asked.
