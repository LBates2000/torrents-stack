# Contributing

Thanks for contributing to this project.

## Workflow
- Create a branch from `main`.
- Keep changes focused and small.
- Run local checks before opening a PR:
  - `docker compose config`
  - `docker compose ps`
- Open a pull request and complete the template.

## Configuration changes
- Do not commit runtime data from `configs/` or `downloads/`.
- Do not commit real credentials, API keys, or VPN private keys.
- Keep `.env` local; update `.env.example` when adding new variables.

## Documentation updates
- Update `README.md` for any behavior, healthcheck, or command changes.
- Keep operational instructions concise and copy/paste ready.

## Commit guidance
- Use clear commit messages in imperative form, for example:
  - `Update jackett healthcheck redirects`
  - `Add compose smoke test workflow`
