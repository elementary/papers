# papers
Papers Document Viewer flatpak built against the elementary platform

## Building, Testing, and Installation
Run `flatpak-builder` to configure the build environment, download dependencies, build, and install

```bash
flatpak-builder build org.gnome.papers.json --user --install --force-clean --install-deps-from=appcenter --install-deps-from=flathub
```

Then execute with

```bash
flatpak run org.gnome.Papers
```

## How to generate `generated-sources.json`
Some code of Papers is written in Rust so it uses Cargo to resolve dependencies of it. However, network access during
the build process is not recommended in Flatpak.
[flatpak-cargo-generator](https://github.com/flatpak/flatpak-builder-tools/tree/master/cargo) resolves this problem
by converting all necessary dependencies listed in the Cargo.lock into a Flatpak manifest file to download them
as modules at the beginning of the build process.

You'll need:

- git
- python3-poetry

First prepare the Cargo.lock of Papers. We patch Cargo.lock to add granite, so we need to apply our patch against
the original source code of Papers:

```
# The tag name matches with the version of papers in our manifest file
git clone https://gitlab.gnome.org/GNOME/papers.git -b 48.4 --depth=1
cd papers/
git apply <path to add-granite.patch>
cd ../
```

Now, we can use flatpak-cargo-generator to generate `generated-sources.json`:

```
git clone https://github.com/flatpak/flatpak-builder-tools.git --depth=1
cd flatpak-builder-tools/cargo/
poetry install
poetry shell
python3 ./flatpak-cargo-generator.py ../../papers/Cargo.lock -o generated-sources.json
```
