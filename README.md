# erltup

`erltup` is a collection of files for using the build tool `tup` in
erlang projects.

`erltup` contains build rules for erlang modules, C programs and C
NIFs.

# Prerequisites

You need to have `tup` installed.

# Use with erlang.mk

`erltup` can be used as a plugin to `erlang.mk`.  By using `erltup`
like this, you can use `erlang.mk` to manage dependencies, run tests,
build releases etc, but use `tup` for very fast and correct builds
during development.

Add the following lines to your `Makefile`:

```makefile
# set TUP if `tup` is not available in PATH
TUP ?= tup
ifneq (, $(shell $(TUP) --version 2>/dev/null))
DEPS = erltup
DEP_PLUGINS = erltup
dep_erltup  = git https://github.com/mbj4668/erltup.git
endif
```

Then run `make deps` to set things up.

When you build the project with `make deps`, `erltup` will create the
following files:

```
tup.config
```

It will also create, if they don't already exist:
```
Tuprules.tup
ebin/Tupfile
```

Now you can use `tup` to rebuild, instead of `make`!

NOTE: While you can use both `make all` and `tup` to rebuild the
project, `tup` complains if it detects that its targets are rebuilt
outside of `tup`.

NOTE: `make clean` in `erlang.mk` removes the entire `ebin` directory,
which means that the `Tupfile` will be removed.  If you have your own
`ebin/Tupfile`, make sure to store it safely!

## Use a custom `Tuprules.tup`

The generated `Tuprules.tup` file has a single rule:

```
# Generated by erltup.mk
include @(DEPS_DIR)/erltup/Tuprules.tup
```

If you need to modify this file, remove the comment and add your own
rules.

## Use a custom `ebin/Tupfile`

The generated `Tupfile` file looks like this:
```
# Generated by erltup.mk

include_rules

: foreach ../src/*.erl |> !erlc |>
```

If you need to modify this file, remove the comment and add your own
rules.

# Use with make

Download `erltup` from git, and include `erltup.mk` in your
`Makefile`.  Or just get inspiration from these files.

`tup.config` with the variables used in `Tuprules.tup`.

# Rules for Erlang

If you are using `erlang.mk` as describes above, a trivial `Tupfile`
for erlang files is generated in `ebin`:

```
include_rules

: foreach ../src/*.erl |> !erlc |>

.gitignore
```

If needed, you can create your own file with additional rules instead.

# Rules for NIFs

Assuming you have your C source code in `$(C_SRC_DIR)`, create a
Tupfile in `$(C_SRC_DIR)`:

```
include_rules

: foreach *.c |> !cc_nif |>

.gitignore
```

And a Tupfile in `priv`:

```
include_rules

: $(C_SRC_DIR)/my_nif.o |> !ld_nif |> my_nif.so

.gitignore
```

# Use `erltup` in a large project

When you have a large project with several erlang applications and
some dependencies, normal `make` is tricky to get right, and often
quite slow.  This is where `tup` shines.

One way to structure such projects is like this:

```
Makefile
erlang.mk
Tuprules.tup
lib/my-app-1/src/*.erl
             ebin/Tupfile
    my-app-2/src/*.erl
             ebin/Tupfile
deps/
```

In this case, erlang.mk on the top is used to manage dependencies and
build releases.

# Known issues

`tup` doesn't work well with `erlc -server`.  It probably keeps track
of the server process and waits for it to stop before continuing.
