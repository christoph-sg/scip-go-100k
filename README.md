# Go fixture with 100,000 SCIP occurrences

This standalone Git repository reproduces the per-file occurrence limit in
[Sourcegraph PR #16587](https://github.com/sourcegraph/sourcegraph/pull/16587).
It has one Go file, no dependencies, and exactly **100,000 occurrences in
`occurrences.go`** when indexed with the locally installed scip-go 0.2.7:

- 3 definitions: the package, `Value`, and `Values`.
- 99,997 references to `Value` in a slice literal.

The source file is about 700 KB, with 20 references per line. Hover or navigate
from any `Value` reference to exercise code navigation. The last lines are beyond
the 50,000-occurrence cap. `index.scip` is already generated and ignored by Git.

## Regenerate and index

```sh
cd /private/tmp/scip-go-100k
python3 generate.py
gofmt -w occurrences.go
go test ./...

# This machine's mise shim has no selected scip-go installation.
# Use the installed binary directly, or substitute your own scip-go path.
SCIP_GO=/Users/christoph/.local/share/mise/installs/go/1.27.0/bin/scip-go
"$SCIP_GO" index --repository-remote=example.com/scip-go-100k
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

The repository is local-only and has an initial commit on `main`. To browse it
in Sourcegraph, make this Git repository available to your development instance
and upload `index.scip` against that repository name and commit. The
`example.com/scip-go-100k` module/remote name is a fixture identifier, not a hosted
repository.
