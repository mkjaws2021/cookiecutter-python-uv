# Notes

Supplement to the [README](README.md): configuration and optional tools.

## Cookiecutter

The [README](README.md) runs cookiecutter with `uvx`, which needs no install.
To install it globally instead:

```sh
uv tool install cookiecutter
```

## Configuring cookiecutter-python-uv

Make a [config file](https://cookiecutter.readthedocs.io/en/stable/advanced/user_config.html)
with default settings (see [template_config.yml](template_config.yml)). Save it as
`.cookiecutterrc`, or pass it with `--config-file template_config.yml`.

## Adding project dependencies

Dependencies can be specified in a space-separated list of PyPI requirements,
which are written as given: `dep1 dep2>=1.2 dep3==4.*`

## Project Tools

### Act

[act](https://github.com/nektos/act) runs GitHub Actions locally in Docker:

```sh
act push
act pull_request
act schedule
```

Install with `brew install act`, or as a GitHub CLI extension:

```sh
gh extension install nektos/gh-act
```

### Direnv

[direnv](https://direnv.net) can automagically load environment variables.

#### Installing direnv

```sh
curl -sfL https://direnv.net/install.sh | bash
```

Warning: if installed simultaneously from multiple sources, bad things can happen.

Make sure that direnv is available in your shell by adding the following to your
.zshrc/.bashrc/.profile (swap zsh for the name of your shell).

```sh
eval "$(direnv hook zsh)"
```

### GitHub CLI

[GitHub CLI](https://cli.github.com/) creates the repository on GitHub. Useful extensions:

- [copilot](https://github.com/github/gh-copilot) - chat interface for questions about the command line
- [dash](https://github.com/dlvhdr/gh-dash) - displays a dashboard with pull requests and issues
- [gh-f](https://github.com/gennaro-tedesco/gh-f) - fuzzy finder for gh-cli
- [gh-notify](https://github.com/meiji163/gh-notify) - shows your GitHub notifications
- [markdown-preview](https://github.com/yusukebe/gh-markdown-preview) - renders markdown documents in your browser
- [poi](https://github.com/seachicken/gh-poi) - safely cleans up old local branches

The README Setup section installs it with webi. Package-manager alternatives are under
[Alternative installation methods](#alternative-installation-methods).

### Prek

[prek](https://prek.j178.dev) runs formatting, linting, and other hooks on `git commit` and
`git push`, and the test suite on `git push` alone. `default_install_hook_types` in the generated
`prek.toml` makes `prek install` write the pre-push shim as well as the pre-commit one.

It ships in the generated project's dev dependency group, so it needs no global install. If you want one anyway:

```sh
uv tool install prek
```

### Alternative installation methods

Use these instead of the README's `curl` commands.

#### Cookiecutter

```sh
# Apt
apt install cookiecutter
# Brew
brew install cookiecutter
```

#### Direnv

```sh
# Apt
apt install direnv
# Brew
brew install direnv
```

#### GitHub CLI

```sh
# Apt
apt install gh
# Brew
brew install gh
```

#### Prek

```sh
# Brew
brew install prek
```
