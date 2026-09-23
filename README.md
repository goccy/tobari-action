# tobari-action

GitHub Action for [tobari](https://github.com/goccy/tobari) - a scoped coverage measurement tool for Go.

## Actions

### Setup (`setup/`)

Installs the tobari CLI and configures `GOFLAGS` for coverage-instrumented builds.

> **Note:** The Go toolchain used for the build must satisfy the `go` directive
> of the installed tobari release, because tobari compiles its runtime package
> with the same toolchain as your project. tobari v0.13.0 and later require Go
> 1.26 or later; v0.12.x requires Go 1.24 or later. Pin `version` if you need
> to stay on an older Go release.

#### Inputs

| Name | Description | Default |
|------|-------------|---------|
| `version` | Version of tobari to install (e.g., `v0.9.0` or `latest`) | `latest` |
| `embed-code` | Embed source code into instrumented binaries | `false` |
| `tags` | Build tags (same as `go build -tags`). Multiple tags can be specified with newlines or commas. | |
| `exclude-analysis` | Package path prefixes to exclude from the dependency analysis. Multiple prefixes can be specified with newlines or commas. | |
| `passed-blocks-only` | Record only the blocks that were actually passed and skip the dependency analysis (tobari v0.13.0 or later) | `false` |

> **Note:** `exclude-analysis` and `passed-blocks-only` are mutually exclusive. Specify at most one.

#### Usage

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: actions/setup-go@v7
        with:
          go-version: '1.26'
      - uses: goccy/tobari-action/setup@v1
      - run: go build ./...
        # GOFLAGS is automatically set, so go build runs with coverage instrumentation
```

With build tags:

```yaml
      - uses: goccy/tobari-action/setup@v1
        with:
          tags: |
            timetzdata
            netgo
```

With packages excluded from the dependency analysis (useful when the dependency
closure contains a large amount of generated code, e.g. gRPC/protobuf clients,
that slows down the analysis):

```yaml
      - uses: goccy/tobari-action/setup@v1
        with:
          exclude-analysis: |
            github.com/org/repo
            github.com/org/other/pb
```

> **Note:** Only exclude packages that never call back into a coverage-target
> package. Coverage targets and the main package are never excluded, even if a
> prefix matches them.

With only the passed blocks recorded (the "places that should be passed" are
left for the consumer to derive, so the dependency analysis is skipped entirely
and builds are faster; requires tobari v0.13.0 or later):

```yaml
      - uses: goccy/tobari-action/setup@v1
        with:
          passed-blocks-only: true
```

> **Note:** Every entry of `counts` in the resulting `tobari.json` carries
> `"passedBlocksOnly": true`, and zero-count blocks are absent. `tobari html`
> (and the `report` action) still use all instrumented blocks of the program as
> the denominator. See the
> [tobari README](https://github.com/goccy/tobari#recording-only-the-passed-blocks)
> for details.

With embed-code enabled:

```yaml
      - uses: goccy/tobari-action/setup@v1
        with:
          embed-code: true
      - run: go build -o myapp .
        # The built binary will have source code embedded
```

### Report (`report/`)

Generates an HTML coverage report from `tobari.json` and uploads it as a directly viewable artifact.

#### Prerequisites

The `setup` action must be run before this action to install the tobari CLI.

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `tobari-json` | Path to tobari.json file(s). Multiple paths can be specified with newlines. | No | `tobari/tobari.json` |
| `source` | Path to source.tar.gz file(s) for source resolution. Multiple paths can be specified with newlines. | No | |
| `binary` | Path to binary with embedded source code | No | |
| `name` | Artifact name for the HTML report | No | `tobari-report` |

> **Note:** `source` and `binary` are mutually exclusive. Specify at most one.

#### Usage

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: actions/setup-go@v7
        with:
          go-version: '1.26'
      - uses: goccy/tobari-action/setup@v1
      - run: go test ./...
      - uses: goccy/tobari-action/report@v1
        # Reads tobari/tobari.json by default
```

With source archive:

```yaml
      - uses: goccy/tobari-action/report@v1
        with:
          source: sources.tar.gz
```

With embedded source binary:

```yaml
      - uses: goccy/tobari-action/setup@v1
        with:
          embed-code: true
      - run: go build -o myapp .
      - run: go test ./...
      - uses: goccy/tobari-action/report@v1
        with:
          binary: myapp
```

With multiple tobari.json files:

```yaml
      - uses: goccy/tobari-action/report@v1
        with:
          tobari-json: |
            service-a/tobari/tobari.json
            service-b/tobari/tobari.json
```

## License

MIT
