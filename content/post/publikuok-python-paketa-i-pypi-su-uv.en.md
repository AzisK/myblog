---
title: Publish a Python Package to PyPI with uv
slug: publish-python-package-to-pypi-with-uv
description: How to create, build and publish a Python package to PyPI using uv
date: 2026-03-09T06:00:00Z
author: Ąžuolas Krušna
tags: ["Software Engineering", "Python", "uv", "PyPI"]
ShowCodeCopyButtons: true
---

Hi there! Today we'll learn how to create and publish a Python package to PyPI from scratch using uv. uv is blazingly fast, which makes programming even more enjoyable!

### Installing uv

If you don't have uv yet, install it in your terminal:

```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Make sure it works:

```bash
uv --version
```

### Creating a new library

uv has a convenient command for initializing a project. We'll use the `--lib` flag, which creates a library — a package meant to be used in other projects.

```bash
uv init --lib my-package
cd my-package
```

This command will create the following project structure:

```
my-package/
├── .python-version
├── README.md
├── pyproject.toml
└── src/
    └── my_package/
        ├── __init__.py
        └── py.typed
```

When creating a package, uv uses the src layout. This means the package code lives in the `src/` directory, not in the project root.

Why does this matter? Without the `src/` layout, Python would import your module directly from the project root — you'd be testing source files, not what the user will actually get. With the `src/` layout, `uv run` first installs the package into a temporary environment, then imports it — so you're testing the actual installed package, just as other people would receive it. This helps catch packaging configuration errors (forgotten files, incorrect paths).

### Reviewing pyproject.toml

The generated `pyproject.toml` file will look something like this:

```toml
[project]
name = "my-package"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.11"
dependencies = []

[build-system]
requires = ["uv_build>=0.10.2,<0.11.0"]
build-backend = "uv_build"
```

The key parts here:

- `[project]` — package metadata: name, version, description, dependencies.
- `[build-system]` — the build system. uv uses its own `uv_build` by default, but you can also use `hatchling`, `flit-core`, `setuptools`, or others.

Let's fill in the metadata according to our needs: description, author, license:

```toml
[project]
name = "my-package"
version = "0.1.0"
description = "My awesome Python package"
readme = "README.md"
requires-python = ">=3.11"
license = "MIT"
authors = [
    { name = "My Name", email = "my@email.com" },
]
dependencies = []
```

### Writing code

Open `src/my_package/__init__.py` and write your package logic. For example:

```python
def hello(name: str = "World") -> str:
    return f"Hello, {name}!"
```

We can test it right away:

```bash
uv run python -c "import my_package; print(my_package.hello('PyPI'))"
```

You should see:

```
Hello, PyPI!
```

### Adding dependencies (if needed)

If your package uses other libraries, add them with `uv add`:

```bash
uv add requests
```

This will automatically update `pyproject.toml` and the `uv.lock` file.

### Building the package

When the code is ready, build it into distribution formats (source distribution and wheel):

```bash
uv build
```

The result will appear in the `dist/` directory:

```
dist/
├── my_package-0.1.0-py3-none-any.whl
└── my_package-0.1.0.tar.gz
```

- `.tar.gz` — the source distribution with all the code.
- `.whl` — the compiled wheel distribution, which installs faster.

### PyPI account and API token

Before publishing, you need to:

1. Create an account at [pypi.org](https://pypi.org/account/register/).
2. Generate an API token: go to [Account Settings → API tokens](https://pypi.org/manage/account/#api-tokens) and create a new token.

PyPI doesn't support password authentication, so an API token is required.

I recommend testing with [TestPyPI](https://test.pypi.org/) first — it's a copy of PyPI for testing, where you can try publishing without consequences. Create an account and API token there separately.

### Publishing to TestPyPI

First, define the TestPyPI index in your `pyproject.toml`:

```toml
[[tool.uv.index]]
name = "testpypi"
url = "https://test.pypi.org/simple/"
publish-url = "https://test.pypi.org/legacy/"
explicit = true
```

Then publish:

```bash
uv publish --index testpypi --token your-testpypi-api-token
```

Check if the package is available:

```bash
uv run --with my-package --index-url https://test.pypi.org/simple/ --no-project -- python -c "import my_package; print(my_package.hello())"
```

### Publishing to PyPI

When everything works with TestPyPI, publish to the real PyPI:

```bash
uv publish --token your-pypi-api-token
```

Now your package is available to everyone!

Check that the package installs correctly:

```bash
uv run --with my-package --no-project -- python -c "import my_package; print(my_package.hello())"
```

The `--no-project` flag ensures the package is installed from PyPI, not from the local project directory.

### Updating the version

To release a new version of the package, you can use the uv `version` subcommand:

```bash
# Set a specific version
uv version 1.0.0

# Bump the minor version (0.1.0 → 0.2.0)
uv version --bump minor

# Bump the patch version (0.1.0 → 0.1.1)
uv version --bump patch
```

Having bumped the version, run `uv build` and `uv publish` to publish the package again.

### Troubleshooting

If something doesn't work:

1. Is the package name available? Check `https://pypi.org/project/your-package/`.
2. Is the API token correct and starts with `pypi-`? TestPyPI and PyPI tokens are different!
3. Is the version new? PyPI doesn't allow overwriting an already published version — bump it with `uv version --bump patch`.
4. Does `uv build` succeed? Check that `pyproject.toml` has the correct `[build-system]` section and that the `src/` structure matches the package name.

---

That's it! Looking forward to trying out your package!

---

### Sources

- uv documentation: Building and publishing a package https://docs.astral.sh/uv/guides/package/
- uv documentation: Working on projects https://docs.astral.sh/uv/guides/projects/
- uv documentation: Creating projects https://docs.astral.sh/uv/concepts/projects/init/
- PyPI: Python Package Index https://pypi.org/
- uv GitHub repository https://github.com/astral-sh/uv
