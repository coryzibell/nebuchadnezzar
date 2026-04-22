# nebuchadnezzar

*The ship that carries the crew.*

Reusable CI workflows for Rust projects.

## Versioning

Nebuchadnezzar uses **semver tags** following the GitHub Actions convention:

| Pin | Example | What you get |
|-----|---------|--------------|
| `@v1` | `@v1` | Latest `v1.x.x` -- recommended for most consumers |
| `@v1.2.3` | `@v1.2.3` | Exact version -- use when you need a reproducible pin |
| `@main` | `@main` | Bleeding edge -- not recommended for production |

**Always pin to `@v1`** (or the current major) unless you have a specific reason not to. The floating major tag is updated automatically on every semver release, so you get bug fixes and non-breaking improvements without changing your workflows.

### Consumers

- [coryzibell/mx](https://github.com/coryzibell/mx)
- [coryzibell/base-d](https://github.com/coryzibell/base-d)
- [coryzibell/fettle](https://github.com/coryzibell/fettle)
- [coryzibell/rust-template](https://github.com/coryzibell/rust-template)

## Workflows

| Workflow | Trigger | Purpose |
|----------|---------|--------|
| `wake-up.yml` | PR/push to main | Gate, test, bump version, tag |
| `the-matrix-has-you.yml` | Called by wake-up | Format, clippy, test (Linux/Mac/Windows) |
| `follow-the-white-rabbit.yml` | Tag push | Build binaries, create release |
| `knock-knock.yml` | Release published | Publish to crates.io |
| `free-your-mind.yml` | Semver tag push (`v*.*.*`) | Create GitHub Release, update floating major tag |

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
    uses: coryzibell/nebuchadnezzar/.github/workflows/wake-up.yml@v1
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
    uses: coryzibell/nebuchadnezzar/.github/workflows/follow-the-white-rabbit.yml@v1
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
    uses: coryzibell/nebuchadnezzar/.github/workflows/knock-knock.yml@v1
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

## Cutting a Release

1. **Push a semver tag** from main:
   ```bash
   git tag v1.0.0
   git push origin v1.0.0
   ```

2. The `free-your-mind.yml` workflow will automatically:
   - Create a GitHub Release with auto-generated notes
   - Update the floating `v1` tag to point at the new release

3. All consumers pinned to `@v1` pick up the change immediately.

**Breaking changes** require a new major version (e.g., `v2.0.0`). Bump the major tag so existing `@v1` consumers are unaffected.
