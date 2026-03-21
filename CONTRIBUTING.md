# Contributing to Loki

Thank you for your interest in contributing to Loki! This guide covers everything you need to get started, from setting up a development environment to submitting your first pull request.

> **Note:** Promtail is considered feature-complete. Future log collection development happens in [Grafana Alloy](https://github.com/grafana/alloy).

## Table of contents

- [Getting help](#getting-help)
- [Before you contribute](#before-you-contribute)
- [Ways to contribute](#ways-to-contribute)
- [Development setup](#development-setup)
- [Submitting a pull request](#submitting-a-pull-request)
- [Dependency management](#dependency-management)
- [Coding standards](#coding-standards)
- [Documentation contributions](#documentation-contributions)
- [Helm chart contributions](#helm-chart-contributions)

## Getting help

Before opening an issue or pull request, check whether your question or idea has already been discussed:

- **Slack**: Join the `#loki` channel on [Grafana Labs Slack](https://slack.grafana.com/)
- **Community forum**: Post in the [Grafana Loki category](https://community.grafana.com) on community.grafana.com
- **Community call**: Monthly on the first Thursday, alternating EU (12:00 UTC) and US (17:00 UTC) — refer to the [agenda doc](https://docs.google.com/document/d/1MNjiHQxwFukm2J4NJRWyRgRIiK7VpokYyATzJ5ce-O8/edit?usp=sharing) for the calendar invite
- **Mailing list**: [loki-developers](https://groups.google.com/forum/#!forum/loki-developers) for development discussion

## Before you contribute

- Read the [Code of Conduct](CODE_OF_CONDUCT.md). All contributors are expected to follow it.
- Check [open issues](https://github.com/grafana/loki/issues) and [pull requests](https://github.com/grafana/loki/pulls) to avoid duplicating work.
- For questions about project direction and governance, refer to [governance](docs/sources/community/governance.md) and [MAINTAINERS.md](MAINTAINERS.md).

## Ways to contribute

- **Bug reports**: Use the [GitHub issue tracker](https://github.com/grafana/loki/issues/new/choose) and fill in the appropriate template.
- **Feature requests**: For small improvements, open an issue. For significant changes, create a [Loki Improvement Document (LID)](#loki-improvement-documents-lids) first.
- **Code changes**: Refer to [Development setup](#development-setup) and [Submitting a pull request](#submitting-a-pull-request).
- **Documentation**: Refer to [Documentation contributions](#documentation-contributions).
- **Helm chart**: Refer to [Helm chart contributions](#helm-chart-contributions).

## Development setup

### Prerequisites

- **Go** 1.25 or later (check `go.mod` for the exact minimum version in use)
- **Docker** or **Podman** (for integration tests and docs preview)
- **Git**

### Clone and build

```bash
git clone https://github.com/grafana/loki.git
cd loki
```

The preferred way to build is with `make`:

| Command | Output |
|---|---|
| `make loki` | `./cmd/loki/loki` |
| `make logcli` | `./cmd/logcli/logcli` |
| `make loki-canary` | `./cmd/loki-canary/loki-canary` |
| `make all` | all of the above |
| `make loki-image` | Docker image for Loki |

### Running tests

```bash
make test              # unit tests
make test-integration  # integration tests (requires Docker, takes ~15 min)
make lint              # run linters (golangci-lint)
```

### Working with local dependency overrides

Use [Go workspaces](https://go.dev/ref/mod#workspaces) to use a locally modified version of a dependency without touching `go.mod`:

```bash
go work init
go work use -r .   # recursively add sub-modules
```

The `go.work` file is gitignored and does not affect other contributors.

## Submitting a pull request

### Loki Improvement Documents (LIDs)

Before opening a large pull request to add or significantly change functionality, create a _Loki Improvement Document (LID)_. LIDs allow the community to discuss and vet ideas in an open, transparent way, inspired by Python's [PEP](https://peps.python.org/pep-0001/) and Kafka's [KIP](https://cwiki.apache.org/confluence/display/KAFKA/Kafka+Improvement+Proposals) processes.

Create a LID as a pull request using the [LID template](docs/sources/community/lids/template.md).

### Commit messages

Loki uses [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/). Every commit message must follow the format `<type>: description`, for example:

```
fix: correct chunk iterator off-by-one error
feat(querier): add partition-aware query path
docs: update upgrade guide for 3.x
```

### PR checklist

Before marking a PR as ready for review:

1. **Title** follows conventional commits format, uses sentence case, and starts with an imperative verb. The title appears in the changelog — write it for a reader without context (for example, `Fix latency spike when querying across multiple ingesters`).
2. **Description** clearly explains what the change does and why.
3. **Branch** is synced with `main`.
4. **Tests** are added or updated where appropriate.
5. **Upgrade guide** at `docs/sources/setup/upgrade/_index.md` is updated if the change affects any of:
   - Default configuration values
   - Metric or label names
   - Log lines used in dashboards or alerts (e.g., lines in `metrics.go` files)
   - Configuration parameters
   - HTTP or gRPC API endpoints
   - Any other change requiring operator attention during an upgrade
6. **Deprecated/deleted config**: If a configuration option is deprecated or removed, update `tools/deprecated-config-checker/deprecated-config.yaml` or `deleted-config.yaml` respectively ([example PR](https://github.com/grafana/loki/pull/10840/commits/0d4416a4b03739583349934b96f272fb4f685d15)).
7. **Documentation** is added or updated for any user-visible change, and follows the [Grafana Writers' Toolkit](https://grafana.com/docs/writers-toolkit/write/).

**Note:** A maintainer must approve and trigger CI for community contributions.

**Note:** For automated agent PRs, append 🤖🤖🤖 to the PR title to opt into the dedicated agent review process.

## Dependency management

Loki uses [Go modules](https://golang.org/cmd/go/#hdr-Modules__module_versions__and_more) for dependency management.

To add or update a dependency:

```bash
# Pick the latest tagged release
go get example.com/some/module/pkg

# Pin a specific version
go get example.com/some/module/pkg@vX.Y.Z

# Tidy and vendor
go mod tidy
go mod vendor
git add go.mod go.sum vendor
git commit
```

Always commit changes to `go.mod`, `go.sum`, and `vendor/` together in the same commit.

## Coding standards

Refer to [CODING_STANDARDS.md](CODING_STANDARDS.md) for the full style guide. The most important rule at a glance:

**Go import grouping** — three groups separated by blank lines: standard library, external packages, internal packages:

```go
import (
    "fmt"
    "math"

    "github.com/prometheus/common/model"
    "github.com/prometheus/prometheus/pkg/labels"

    "github.com/grafana/loki/v3/pkg/logproto"
    "github.com/grafana/loki/v3/pkg/logql"
)
```

Run `make lint` before submitting — it enforces import ordering and other style rules automatically.

## Documentation contributions

Loki's documentation lives in `docs/sources/` and is published to [grafana.com/docs/loki](https://grafana.com/docs/loki/latest/).

- The Grafana team's [Writers' Toolkit](https://grafana.com/docs/writers-toolkit/) covers writing style, the [Style Guide](https://grafana.com/docs/writers-toolkit/write/style-guide/), and templates.
- Documentation is written in CommonMark Markdown with Hugo shortcodes.
- PRs are merged to `main`. To publish to the current release immediately, add a backport label such as `backport-release-3.x`.

**To preview docs locally:**

```bash
cd docs
make docs   # requires Docker or Podman
# open http://localhost:3002/docs/loki/latest/
```

> If `make docs` fails with a path-sharing error, add `/tmp` to Docker's shared paths via Docker Desktop → Settings → Resources → File Sharing. The command is memory-intensive — increase Docker's memory limit if it crashes.

## Helm chart contributions

Refer to [production/helm/loki/CONTRIBUTING.md](production/helm/loki/CONTRIBUTING.md) for chart-specific guidelines.
