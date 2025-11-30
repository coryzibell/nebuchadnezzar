# nebuchadnezzar

*The ship that carries the crew.*

Reusable CI workflows for Rust projects.

## Workflows

| Workflow | Trigger | Purpose |
|----------|---------|--------|
| `wake-up.yml` | PR/push to main | Gate, test, bump version, tag |
| `the-matrix-has-you.yml` | Called by wake-up | Format, clippy, test (Linux/Mac/Windows) |
| `follow-the-white-rabbit.yml` | Tag push | Build binaries, create release |
| `knock-knock.yml` | Release published | Publish to crates.io |

## Usage

Create minimal workflow files in your repo that call the ship:

### `.github/workflows/wake-up.yml`
```yaml
name: wake up.
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened, ready_for_review]
jobs:
  ci:
    uses: coryzibell/nebuchadnezzar/.github/workflows/wake-up.yml@main
    secrets:
      GH_PAT: ${{ secrets.GH_PAT }}
```

### `.github/workflows/follow-the-white-rabbit.yml`
```yaml
name: follow the white rabbit.
on:
  push:
    tags: ['v*']
jobs:
  release:
    uses: coryzibell/nebuchadnezzar/.github/workflows/follow-the-white-rabbit.yml@main
    secrets:
      GH_PAT: ${{ secrets.GH_PAT }}
```

### `.github/workflows/knock-knock.yml`
```yaml
name: knock, knock.
on:
  release:
    types: [published]
jobs:
  publish:
    uses: coryzibell/nebuchadnezzar/.github/workflows/knock-knock.yml@main
    secrets:
      CARGO_REGISTRY_TOKEN: ${{ secrets.CARGO_REGISTRY_TOKEN }}
```

## Required Secrets

- `GH_PAT` - GitHub PAT with repo access (for tagging and releases)
- `CARGO_REGISTRY_TOKEN` - crates.io API token (optional, for publishing)

## Build Targets

Releases include binaries for:
- Linux: x86_64, aarch64, x86_64-musl, aarch64-musl
- FreeBSD: x86_64
- Windows: x86_64, aarch64
- macOS: x86_64, aarch64
