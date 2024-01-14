# Nix

Nix only computes values when necessary. The `:p` prefix before an
expression in the `nix repl` and the `--strict` flag in
`nix-instantiate` force evaluation, even when it would be otherwise
skipped.

```sh
# Evaluate a nix expression in the repl.
nix-repl> :p { a.b.c = 1; }

# Evaluate nix expression in a file. Will use ./default.nix if no file
# path is provided.
nix-instantiate --eval --strict ./file.nix
```

Attribute sets declared with a `rec` prefix are recursive, allowing
access to attributes from within the set. Elements can be declared in
any order.

```nix
rec {
  one = 1;
  two = one * 1;
}
```

`with ...; ...` can be used to access attributes without repeatedly
referencing their set.

```nix
let a = { x = 1; y = 2; };
in with a; [ x y ] # Equivalent to [ a.x a.y ]
```

`inherit` allows avoiding repetition of value assignment.

```nix
let a = { x = 1; y = 2; };
in {
  inherit x y; # Equivalent to { x = x; y = y; }
}

# `inherit` from a specific attribute set.
let a = { x = 1; y = 2; };
in {
  inherit (a) x y; # Equivalent to { x = a.x; y = a.y; }
}

# `inherit` inside `let` expressions.
let inherit ({ x = 1; y = 2; }) x y;
in [ x y ]
# Equivalent to
# let
#   x = { x = 1; y = 2; }.x;
#   y = { x = 1; y = 2; }.y;
# in
```

Functions in Nix are anonymous, i.e. lambda's.

```nix
# Single argument
x: x + 1

# Multiple arguments (curried function)
x: y: x + y

# Attribute set argument (passed attribute set must be exact)
{ x, y }: x + y

# With default attributes
{ x, y ? 0 }: x + y

# With additional attributes allowed
{ x, y, ... }: x + y

# With a named attribute set argument
args@{ x, y, ... }: x + y + args.z

# Named function
let f = x: x + 1;
in f

# Calling a function with an argument
let f = x: x.a + 1;
in f { a = 1; }

# Calling a function with an argument referenced by name
let
  f = x: x.a + 1;
  v = { a = 1; };
in f v # 2

# Calling a function with an argument via parentheses
(x: x + 1) 1 # 2
```

[builtins](https://nix.dev/manual/nix/2.18/language/builtins) are
functions built into the Nix language.

`import` takes a path to a Nix file, evaluates the expression and
returns the value. If the expression in the file is a function,
arguments can be passed directly to the `import` statement.

```sh
$ echo "x: x + 1" > file.nix
$ echo "import ./file.nix 1" > default.nix
$ nix-instantiate --eval --strict # 2
```

[pkgs.lib](https://nixos.org/manual/nixpkgs/stable/#sec-functions-library)
is a library of functions provided by the `nixpkgs` repository, whose
attribute set is named `pkgs` by convention.

String interpolated paths whether they be files or directories (and all
their contents) get stored in the Nix store.

[Fetchers](https://nixos.org/manual/nixpkgs/stable/#chap-pkgs-fetchers)
allow us to take build inputs and process their contents further before
storing them in the Nix store from remote locations.

```nix
builtins.fetchurl
builtins.fetchTarball
builtins.fetchGit
builtins.fetchClosure
```

## References

- [Nix Language](https://nix.dev/tutorials/nix-language)
