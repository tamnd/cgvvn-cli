# cgvvn

A command line for cgvvn.

`cgvvn` is a single pure-Go binary. It reads public cgvvn data
over plain HTTPS, shapes it into clean records, and prints output that pipes
into the rest of your tools. No API key, nothing to run alongside it.

The same package is also a [resource-URI driver](#use-it-as-a-resource-uri-driver),
so a host program like [ant](https://github.com/tamnd/ant) can address
cgvvn as `cgvvn://` URIs.

## Install

```bash
go install github.com/tamnd/cgvvn-cli/cmd/cgvvn@latest
```

Or grab a prebuilt binary from the [releases](https://github.com/tamnd/cgvvn-cli/releases), or run
the container image:

```bash
docker run --rm ghcr.io/tamnd/cgvvn:latest --help
```

## Usage

```bash
cgvvn page <path>                      # fetch one page as a record
cgvvn page <path> -o json              # as JSON, ready for jq
cgvvn page <path> --template '{{.Body}}'  # just the readable body text
cgvvn links <path>                     # the pages it links to, one per line
cgvvn --help                           # the whole command tree
```

Every command shares one output contract: `-o table|json|jsonl|csv|tsv|url|raw`,
`--fields` to pick columns, `--template` for a custom line, and `-n` to limit.
The default adapts to where output goes (a table on a terminal, JSONL in a
pipe), so the same command reads well by hand and parses cleanly downstream.

This is a fresh scaffold. It ships one example resource type, `page`, wired end
to end. Model the real cgvvn records in `cgvvn/` and declare their
operations in `cgvvn/domain.go`; each one becomes a command, an HTTP
route, and an MCP tool at once.

## Serve it

The same operations are available over HTTP and as an MCP tool set for agents,
with no extra code:

```bash
cgvvn serve --addr :7777    # GET /v1/page/<path>  returns NDJSON
cgvvn mcp                   # speak MCP over stdio
```

## Use it as a resource-URI driver

`cgvvn` registers a `cgvvn` domain the way a program registers a
database driver with `database/sql`. A host enables it with one blank import:

```go
import _ "github.com/tamnd/cgvvn-cli/cgvvn"
```

Then [ant](https://github.com/tamnd/ant) (or any program that links the package)
dereferences `cgvvn://` URIs without knowing anything about cgvvn:

```bash
ant get cgvvn://page/<path>   # fetch the record
ant cat cgvvn://page/<path>   # just the body text
ant ls  cgvvn://page/<path>   # the pages it links to, each addressable
ant url cgvvn://page/<path>   # the live https URL
```

## Development

```
cmd/cgvvn/   thin main: hands cli.NewApp to kit.Run
cli/                 assembles the kit App from the cgvvn domain
cgvvn/                the library: HTTP client, data models, and domain.go (the driver)
docs/                tago documentation site
```

```bash
make build      # ./bin/cgvvn
make test       # go test ./...
make vet        # go vet ./...
```

## Releasing

Push a version tag and GitHub Actions runs GoReleaser, which builds the
archives, Linux packages, the multi-arch GHCR image, checksums, SBOMs, and a
cosign signature:

```bash
git tag v0.1.0
git push --tags
```

The Homebrew and Scoop steps self-disable until their tokens exist, so the first
release works with no extra secrets.

## License

Apache-2.0. See [LICENSE](LICENSE).
