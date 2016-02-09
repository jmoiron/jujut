### Juju Test Bug Reproduction

This is a somewhat bizarre minimal test that reproduces behavior in the `runtime.Callers` function called by
`juju/errors/path.init` that causes it to panic.

This package has a single test file which starts an import chain:

* `bug_test` imports `github.com/jmoiron/jujut/pkg1`
* `pkg1` imports `github.com/jmoiron/jujut/pkg2`
* `pkg2` imports `github.com/jmoiron/jujut/pkg3`
* `pkg3` imports `github.com/jmoiron/jujut/pkg4`
* `pkg4` imports `github.com/juju/errors`

This causes `runtime.funcline` to fail and return 0 before copying the name of the file from the filetab, leaving its
default value of "?" [1]

When I run `go test` and add a print print statement to juju/errors/path.go's init[2], It prints:

```
5128569 ? 0 true
```

It seems clear that `runtime.Caller` doesn't mind returning garbage in 1.4.x in general [3] and a line return of "0"
would align with a file of "?".  Note that this code has been changed since 1.5 series and likely has completely
different behavior now.

[1] https://github.com/golang/go/blob/release-branch.go1.4/src/runtime/symtab.go#L212
[2] https://github.com/juju/errors/blob/master/path.go#L19
[3] https://github.com/golang/go/blob/release-branch.go1.4/src/runtime/extern.go#L115-L117
