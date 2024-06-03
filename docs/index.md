# Welcome to hostr

This documentation is generated with MkDocs. For full documentation visit [mkdocs.org](https://www.mkdocs.org).

## Commands

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs -h` - Print help message and exit.

If you installed with pip, you can find mkdocs at `/home/user_name/.local/bin/`

## Installation

Install `MKDocs`:

```bash
pip install mkdocs
```

`MkDocs` does not allow re-using your `README.md` in your root folder, install the following plugin to allow `mkdocs.yml` in the same folder as your documentation (the `docs` folder which normally contains your documentation):

```bash
pip install mkdocs-same-dir
```

`Material for MkDocs` Started as a stylesheet for `MkDocs` and is now a powerful documentation framework on top of `MkDocs`, install with:

```bash
pip install mkdocs-material
```

and add the following line to `mkdocs.yml`:

```yaml
theme:
  name: material

markdown_extensions:
  - pymdownx.superfences:
      custom_fences:
        - name: mermaid
          class: mermaid
          format: !!python/name:pymdownx.superfences.fence_code_format
```

Typically, you can find the configuration settings in a [configuration section](https://squidfunk.github.io/mkdocs-material/reference/diagrams/) on the [Material for MkDocs documentation site](https://squidfunk.github.io/mkdocs-material/).  

Check the [mkdocs](./mkdocs.yml) file for more plugin/extension examples.

## Project layout

```text
    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Other markdown pages, images and other files.
```
