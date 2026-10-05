# Go fixture with 100,000 SCIP occurrences

This standalone Git repository reproduces the per-file occurrence limit in
[Sourcegraph PR #16587](https://github.com/sourcegraph/sourcegraph/pull/16587).
It has one Go file, no dependencies, and exactly **100,000 occurrences in
`occurrences.go`** when indexed with the locally installed scip-go 0.2.7:

- 3 definitions: the package, `Value`, and `Values`.
- 99,997 references to `Value` in a slice literal.

The source file is about 700 KB, with 20 references per line. Hover or navigate
from any `Value` reference to exercise code navigation. The last lines are beyond
the 50,000-occurrence cap. Generated `index.scip` files are ignored by Git.

## Regenerate and index

```sh
git clone https://github.com/christoph-sg/scip-go-100k.git
cd scip-go-100k
python3 generate.py
gofmt -w occurrences.go
go test ./...

scip-go index --repository-remote=github.com/christoph-sg/scip-go-100k
```

To generate a different total, use e.g. `python3 generate.py 50001`, format,
commit the changed file, and index again. Commit before indexing so the inferred
module version matches the source revision.

## Verify the actual index

```sh
scip print --json index.scip | jq '.documents[] | {
  file: .relative_path,
  occurrences: (.occurrences | length),
  definitions: ([.occurrences[] | select((.symbol_roles // 0) % 2 == 1)] | length)
}'
```

Expected: one document named `occurrences.go`, 100000 occurrences, 3 definitions.
`go test ./...` checks that the generated package compiles; there are no tests.

## Use with a local Sourcegraph instance

Sync `github.com/christoph-sg/scip-go-100k` through your instance's GitHub code
host connection. After indexing locally, authenticate `src` to your instance
and upload the index from this repository's root:

```sh
SRC_ENDPOINT=https://sourcegraph.test:3443 src code-intel upload \
  -repo=github.com/christoph-sg/scip-go-100k -file=index.scip
```

Adjust the endpoint to your development instance. The upload uses the current
Git commit. Once processing completes, open `occurrences.go` on `main` to test
the warning and on-demand code navigation beyond the 50,000-occurrence limit.
