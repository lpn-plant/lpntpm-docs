# About

This repository contains source code for lpnTPM documentation webpage

# Local development

## Environment setup

```bash
$ virtualenv -p $(which python3) venv
$ source venv/bin/activate
$(venv) pip install mkdocs mkdocs-material
```

## Build

```bash
$(venv) mkdocs build
```

## Preview

```bash
$(venv) mkdocs serve
```

# Contribution

Please use GitHub `Pull Request` and `Issues` to collaborate.

# pre-commit hooks

- [Install pre-commit](https://pre-commit.com/index.html#install)

- Install hooks into repo:

```bash
pre-commit install --hook-type commit-msg
```

- Enjoy automatic checks on each `git commit` action!

- (Optional) Run hooks on all files (for example, when adding new hooks or
  configuring existing ones):

```bash
pre-commit run --all-files
```
