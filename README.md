# Cadmus Conceptual Documentation

Just like the Cadmus project itself, this documentation is work in progress. Its main purpose is highlighting some conceptual points standing behind the general architecture of the system.

For a more theoric and non-technical introduction to the Cadmus system, please see:

- [my seminar presentation at VeDPH](https://www.youtube.com/watch?v=lYykjz26TCg&feature=youtu.be)
- D. Fusi, _Sailing for a Second Navigation: Paradigms in Producing Digital Content_, «SemRom» n.s. 7 (2018) 213-276.

Note: all the diagrams found in this documentation are made using [PlantUml](https://plantuml.com/). Some editors like [VSCode](https://code.visualstudio.com/) have extensions which directly support viewing them in a Markdown preview. See the [PlantUml web site](https://plantuml.com/) for more visualization options.

## Stack

Cadmus essentially includes this stack of layers:

1. *database*: MongoDB for data, MySql for index.
2. *data layer*: storage repositories and their types.
3. *business layer*: core models and logic.
4. *web API layer*: REST API. A future GraphQL API is planned.
5. *web frontend*: Angular 10+ web application.

These layers are distributed across 3 projects: core (2-3), api (4), web (5).

## Core

Core data and business layers.

- [core](core/core.md)
  - [core layers](core/core.layers.md)
  - [layer reconciliation](core/layer-reconciliation.md)
  - [core storage](core/core.storage.md)
  - [core config](core/core.config.md)
  - [dynamic lookup](core/dynamic-lookup.md)
- [backend packages](core/packages.md)
- [adding parts and fragments](core/adding-parts.md)
- [profiles](core/profiles.md)
- [seeding](core/seeding.md)

## API

Web REST API.

- [settings](api/settings.md)

## Web

Web frontend.

- [architecture](architecture.md): general monorepo architecture.
- [routes](routes.md): most relevant route templates.
- [editor components](editor-components.md): editor components.
- [edit state](edit-state.md): app's local edit state.
- [roles](roles.md): user roles for authorizing operations.
- [adding parts](adding-parts.md): how to add new parts.
- [adding fragments](adding-fragments.md): how to add new fragments.
- [demo presets](demo-presets.md): preset JSON samples for editors demo pages.

## Deploying

- [setting up Docker](deploy/docker-setup.md)
- [building Docker images](deploy/docker-build.md)
- [using Docker images](deploy/docker-usage.md)
