# Contribution guidelines

Contributing to this project should be as easy and transparent as possible, whether it's:

- Reporting a bug
- Discussing the current state of the code
- Submitting a fix
- Proposing new features

## GitHub is used for everything

GitHub is used to host code, to track issues and feature requests, and to accept pull requests.

1. Fork the repo and create your branch from `master`.
2. If you've changed behavior, update the documentation (README and, if relevant, `translations/`).
3. Run `scripts/lint` and make sure `ruff check .` passes — CI enforces it.
4. Test your contribution against a running Home Assistant instance (see below).
5. Open a pull request!

## Development environment

You need Python 3.14.2 or newer (required by current Home Assistant).

- `scripts/setup` — install dependencies from `requirements.txt`.
- `scripts/develop` — start a standalone Home Assistant instance with this integration loaded (config lives in `./config`, created on first run). It listens for a remote debugger on `localhost:5678` (debugpy), and HA is served on port 8123.
- `scripts/lint` — format and autofix with ruff.

A VS Code [devcontainer](.devcontainer.json) is included that runs `scripts/setup` for you and forwards port 8123.

## Coding style

The project lints with [ruff](https://docs.astral.sh/ruff/) using `select = ["ALL"]` (see `.ruff.toml`) and is formatted with `ruff format`. Prefer a targeted `# noqa: RULE reason` over widening the global ignore list.

CI also runs [hassfest](https://developers.home-assistant.io/blog/2020/04/16/hassfest/), [HACS validation](https://github.com/hacs/action), and CodeQL on pushes and pull requests targeting `master`.

## Report bugs using GitHub's [issues](../../issues)

Report a bug by [opening a new issue](../../issues/new/choose). Great bug reports include:

- A quick summary and/or background
- Steps to reproduce — be specific
- What you expected to happen, and what actually happened
- Relevant Home Assistant log lines (enable debug logging for `custom_components.victorsmartkill`)
- A diagnostics download from the integration page if possible (sensitive fields are redacted automatically)

## License

By contributing, you agree that your contributions will be licensed under the project's [MIT License](LICENSE).
