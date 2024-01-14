## nix branch `use-zipballs`

Use zipballs for Github Flake inputs.\
This branch is based on https://github.com/NixOS/nix/pull/6530.

### pkg snippet
The following snippet shows how to build this branch with just nixpkgs >= `nixos-23.05`.\
This is useful for inclusion in a NixOS config.\
It requires files `boehmgc-traceable_allocator-public.diff` and
`libzip-unix-time.patch` which can be found in this repo.\
Copy them to your config and adjust the paths in the code below.
```nix
{ pkgs, ... }:
let
  nix = let
    baseNix = pkgs.nixVersions.nix_2_17;
  in
    (baseNix.override {
      boehmgc = baseNix.boehmgc.overrideAttrs (old: {
        patches = old.patches ++ [
          ./FIXME/boehmgc-traceable_allocator-public.diff
        ];
      });
    }).overrideAttrs (old: rec {
      version = "2.20.0-zipballs";

      buildInputs = old.buildInputs ++ [
        (pkgs.libgit2.overrideAttrs (attrs: {
          src = pkgs.fetchFromGitHub {
            owner = "libgit2";
            repo = "libgit2";
            rev = "45fd9ed7ae1a9b74b957ef4f337bc3c8b3df01b5";
            hash = "sha256-oX4Z3S9WtJlwvj0uH9HlYcWv+x1hqp8mhXl7HsLu2f0=";
          };
          version = "1697646580";
          cmakeFlags = (attrs.cmakeFlags or []) ++ ["-DUSE_SSH=exec"];
        }))
        (pkgs.libzip.overrideDerivation (old: {
          # Temporary workaround for https://github.com/NixOS/nixpkgs/pull/178755
          cmakeFlags = old.cmakeFlags or [] ++ [ "-DBUILD_REGRESS=0" ];
          patches = [ ./FIXME/libzip-unix-time.patch ];
        }))
      ];

      src = pkgs.fetchFromGitHub {
        owner = "erikarvstedt";
        repo = "nix";
        rev = "<FIXME: Insert rev from branch `use-zipballs` here>";
        sha256 = "";
      };
    });
in
  ...
```

# Nix

[![Open Collective supporters](https://opencollective.com/nixos/tiers/supporter/badge.svg?label=Supporters&color=brightgreen)](https://opencollective.com/nixos)
[![Test](https://github.com/NixOS/nix/workflows/Test/badge.svg)](https://github.com/NixOS/nix/actions)

Nix is a powerful package manager for Linux and other Unix systems that makes package
management reliable and reproducible. Please refer to the [Nix manual](https://nixos.org/nix/manual)
for more details.

## Installation and first steps

Visit [nix.dev](https://nix.dev) for [installation instructions](https://nix.dev/tutorials/install-nix) and [beginner tutorials](https://nix.dev/tutorials/first-steps).

Full reference documentation can be found in the [Nix manual](https://nixos.org/nix/manual).

## Building And Developing

See our [Hacking guide](https://nixos.org/manual/nix/unstable/contributing/hacking.html) in our manual for instruction on how to
 set up a development environment and build Nix from source.

## Contributing

Check the [contributing guide](./CONTRIBUTING.md) if you want to get involved with developing Nix.

## Additional Resources

- [Nix manual](https://nixos.org/nix/manual)
- [Nix jobsets on hydra.nixos.org](https://hydra.nixos.org/project/nix)
- [NixOS Discourse](https://discourse.nixos.org/)
- [Matrix - #nix:nixos.org](https://matrix.to/#/#nix:nixos.org)

## License

Nix is released under the [LGPL v2.1](./COPYING).
