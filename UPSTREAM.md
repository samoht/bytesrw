# Upstream notes

## `getentropy` is hidden by musl in strict C modes

Status: fixed on the shared `wip` branch. Upstream issue
[dbuenzli/bytesrw#7](https://github.com/dbuenzli/bytesrw/issues/7) is only a
partial fix.

`bytesrw.0.3.0` fails to compile on Alpine when the C compiler is invoked in a
strict language mode:

```text
src/sysrandom/bytesrw_sysrandom_stubs.c:93:8:
error: implicit declaration of function 'getentropy'
```

Commit `89e1d04` for issue #7 added the missing `<unistd.h>` include. On musl,
however, that header declares `getentropy` only when `_GNU_SOURCE` or
`_BSD_SOURCE` is defined. With `-std=c99` neither feature-test macro is enabled,
so including the correct header is not sufficient.

The failure surfaced while bootstrapping with the pinned OCaml 5.4.1 LTO
compiler. The LTO changes are not the cause: compiling the stub directly is a
fast reproducer, without building the compiler or the package:

```sh
cc -std=c99 -Werror=implicit-function-declaration \
  -I /usr/lib/ocaml \
  -c src/sysrandom/bytesrw_sysrandom_stubs.c
```

The fix defines `_GNU_SOURCE` on Linux before any system header is included.
The command above fails against both the 0.3.0 release and the source after
issue #7, then succeeds with the patched source.
