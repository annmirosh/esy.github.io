---
id: configuration
title: Configuration
---

* [Configuring Your `package.json`](#configuring-your-packagejson)
  * [Specify Build & Install Commands](#specify-build--install-commands)
    * [`esy.build`](#esybuild)
    * [`esy.install`](#esyinstall)
  * [Enforcing Out Of Source Builds](#enforcing-out-of-source-builds)
  * [Exported Environment](#exported-environment)

### Configuring Your `package.json`

`esy` knows how to build your package and its dependencies by looking at the
`esy` config section in your `package.json`.

This is how it looks for a [jbuilder](https://jbuilder.readthedocs.io/) based project:

```json
{
  "name": "example-package",
  "version": "1.0.0",

  "esy": {
    "build": [
      "jbuilder build"
    ],
    "install": [
      "esy-installer"
    ],
    "buildsInSource": "_build"
  },

  "dependencies": {
    "anotherpackage": "1.0.0",
    "@esy-ocaml/esy-installer"
  }
}
```

#### Specify Build & Install Commands

The crucial pieces of configuration are `esy.build` and `esy.install` keys, they
specify how to build and install built artifacts.

##### `esy.build`

Describe how your project's default targets should be built by specifying
a list of commands with `esy.build` config key.

For example, for a [jbuilder](https://jbuilder.readthedocs.io/) based project you'd want to call `jbuilder build`
command.

```
{
  "esy": {
    "build": [
      "jbuilder build",
    ]
  }
}
```

Commands specified in `esy.build` are always executed for the root's project
when user calls `esy build` command.

[Esy variable substitution syntax](environment.md#variable-substitution-syntax) can be used to
declare build commands.

##### `esy.install`

Describe how you project's built artifacts should be installed by specifying a
list of commands with `esy.install` config key.

```
{
  "esy": {
    "build": [...],
    "install": [
      "esy-installer"
    ]
  }
}
```

For `jbuilder` based projects (and other projects which maintain `.install` file
in opam format) that could be just a single `esy-installer` invokation. The
command is a thin wrapper over `opam-installer` which configures it with Esy
defaults.

[Esy variable substitution syntax](environment.md#variable-substitution-syntax) can be used to
declare install commands.

#### Enforcing Out Of Source Builds

Esy requires packages to be built "out of source".

It allows Esy to separate source code from built artifacts and thus reuse the
same source code location with several projects/sandboxes.

There are three modes which are controlled by `esy.buildsInSource` config key:

```
{
  "esy": {
    "build": [...],
    "install": [...],
    "buildsInSource": "_build" | false | true,
  }
}
```

Each mode changes how Esy executes [build commands](#esybuild). This is how
those modes work:

* `"_build"`

  Build commands can place artifacts inside the `_build` directory of the
  project's root (`$cur__root/_build` in terms of Esy [build
  environment](environment.md#build-environment)).

  This is what [jbuilder](https://jbuilder.readthedocs.io/) or [ocamlbuild](https://github.com/ocaml/ocamlbuild/blob/master/manual/manual.adoc) (in its default configuration)
  users should be using as this matches those build systems' conventions.

* `false` (default if key is omitted)

  Build commands should use `$cur__target_dir` as the build directory.

* `true`

  Projects are allowed to place build artifacts anywhere in their source tree, but not outside of their source tree. Otherwise, Esy will defensively copy project's root into `$cur__target_dir` and run build commands from there.

  This is the mode which should be used as the last resort as it degrades
  performance of the builds greatly by placing correctness as a priority.
