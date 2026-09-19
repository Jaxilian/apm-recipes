# apm-recipes

The package repository for [apm](../apm), the AOS package manager. Recipes
live in git; the signed index and the artifacts are the assets of one
GitHub Release with a fixed tag, `index`, so every URL a client fetches is
stable and costs nothing to serve.

## Use it

```sh
sudo apm repo add main https://github.com/Jaxilian/apm-recipes/releases/download/index
sudo apm key trust "$(curl -sL https://raw.githubusercontent.com/Jaxilian/apm-recipes/main/keys/apm.pub | sed -n 2p)"
sudo apm update
apm find hello
sudo apm install hello-c
```

`key trust` takes a file path or the key's base64 (the second line of a
minisign `.pub`).

## Layout

```
recipes/<xx>/<org>.<name>/recipe.toml   one recipe per package; xx = first two
                                        letters of the name, so a directory never
                                        holds more than a few hundred entries
keys/apm.pub                            the publisher's minisign public key
publish.sh                              builds index/ and uploads it to the release
index/                                  (not in git) what a client fetches:
                                        stamp.toml + .minisig, index-<sha256>.zip,
                                        and every package, recipe and source tarball
```

Three kinds of package, one index:

| Kind | In the index as | The client |
| --- | --- | --- |
| prebuilt | `<org>-<name>-<version>-<release>.<arch>.apkg` | downloads, verifies, installs |
| recipe | `<org>.<name>.recipe.toml` | downloads, verifies, **compiles** with the gcc and cargo AOS ships |
| vendor binary | a recipe whose source is the vendor's own download | downloads from the vendor, verifies against the recipe's sha256 |

## Add a package

1. Write `recipes/<xx>/<org>.<name>/recipe.toml`. `apm new` scaffolds one;
   `apm build recipes/.../recipe.toml` builds it locally.
2. Every `[[source]]` needs a `sha256`. A recipe without one does not parse.
   Never point at a rolling "latest" URL — the checksum is the only integrity
   a third-party download has.
3. A source tarball with no stable home goes in `index/` before publishing
   and is addressed by the release URL.
4. `./publish.sh`, then commit the recipe.

## What is deliberately not here yet

- **GitHub Actions.** apm itself is not on GitHub yet, so nothing in CI can
  build the index. `publish.sh` runs on the publisher's machine.
- **Per-package releases.** One release holds everything; GitHub allows a
  thousand assets on it, which is plenty until the runtime bundle lands.
- **More than one signing key.** The client trusts whatever is in its
  keyring; rotation is adding a second `.pub`.
