# Developer's Guide - Creating a Cadmus Project

The [developer environment](./environment.md) includes MongoDB, MySql, Visual Studio CE, NodeJS, and Angular.

Quick links:

- [backend: adding parts](./backend/adding-parts.md)
- [backend: adding fragments](./backend/adding-fragments.md)

In what follows, the project's name (e.g. `Itinera`, `Pura`, `Tgr`, `Bdm`...) is abbreviated as `<PRJ>`.

## Backend

1. [create the backend models](./backend/core.md)
2. [create the backend services](./backend/services.md)
3. [add CLI plugins](./backend/cli.md): this is required only if you want to use the [Cadmus CLI tool](https://github.com/vedph/cadmus_tool) with your parts.
4. [create the backend API](./backend/api.md)

Step 1 is required only if you are going to add new models (parts/fragments). If you are just reusing existing models, you can proceed directly to step 2.

## Frontend

1. [create the frontend app](./frontend/creating.md) the frontend with its part/fragment editors libraries
2. [adding parts](./frontend/adding-parts.md) and [fragments](./frontend/adding-fragments.md)

## Other Features

1. [adding preview](adding-preview.md)
