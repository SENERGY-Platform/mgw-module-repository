# mgw-module-repository

Main SENERGY module repository for the multi-gateway.

## Channels

The repository is split into three channels:

| Channel   | Purpose                                                                          |
|-----------|----------------------------------------------------------------------------------|
| `main`    | Stable, production ready modules.                                                |
| `legacy`  | Older module versions kept for compatibility reasons. Might break in the future. |
| `testing` | Latest features. May contain bugs and breaking changes.                          |

Rules:

- Each channel can hold a different version of the same module.
- Each channel can only hold **one** version of a module.
- A module does not have to be present in every channel — it may exist in a single channel only.

## Structure

Each channel is a top level directory marked by a `.channel` file. Within a channel, every module lives
in its own directory containing a `Modfile` (`Modfile.yml` / `Modfile.yaml`) and any additional files the
module references (configuration files, templates, …).

## Validation

Modfiles are validated on every push to `main` by
[mgw-module-validator](https://github.com/SENERGY-Platform/mgw-module-validator); module IDs must be
prefixed with `github.com/SENERGY-Platform/`. Modules in the `main` channel additionally require a
description. A successful validation run moves the `main-validated` tag to the validated commit.

## Adding this repository to a running multi-gateway

Use the following JSON to register this repository on a running multi-gateway:

```json
{
  "owner": "SENERGY-Platform",
  "repository": "mgw-module-repository",
  "reference": "main-validated",
  "priority": 1000,
  "channels": [
    {
      "name": "main",
      "priority": 2,
      "blacklist": null
    },
    {
      "name": "testing",
      "priority": 1,
      "blacklist": null
    },
    {
      "name": "legacy",
      "priority": 0,
      "blacklist": null
    }
  ]
}
```

Fields:

| Field                | Description                                                                                                            |
|----------------------|------------------------------------------------------------------------------------------------------------------------|
| `owner`              | GitHub owner (organisation or user) of the repository.                                                                  |
| `repository`         | Name of the GitHub repository.                                                                                          |
| `reference`          | Git reference to read the modules from. `main-validated` always points at the latest commit that passed validation.     |
| `priority`           | Priority of this repository relative to other repositories configured on the multi-gateway. Must be unique across all repositories. |
| `channels`           | Channels of this repository to use. A channel that is not listed is ignored.                                            |
| `channels[].name`    | Name of the channel — must match the channel directory in the repository.                                               |
| `channels[].priority`| Priority of the channel within this repository.                                                                          |
| `channels[].blacklist`| Names of directories within the channel directory to ignore. `null` ignores nothing. `.git` and `.github` are always ignored. |

A higher priority takes precedence, on both levels: the repository priority decides between repositories
offering the same module, the channel priority between channels of the same repository offering it. With
the configuration above, a module present in both `main` and `testing` is therefore installed from `main`,
and `legacy` is only used for modules that exist in no other channel.

The same ranking is used to resolve module dependencies: they are taken from the highest priority
repository and its highest priority channel whenever possible, and only fall back to the repository and
channel the depending module itself came from.