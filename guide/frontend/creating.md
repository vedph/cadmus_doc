# Cadmus Frontend App

- [Cadmus Frontend App](#cadmus-frontend-app)
  - [Creating the Angular Workspace](#creating-the-angular-workspace)
  - [Adding Libraries](#adding-libraries)
  - [Setting Environment Variables](#setting-environment-variables)
  - [Tuning Angular Settings](#tuning-angular-settings)
  - [Assets](#assets)
  - [Cadmus Infrastructure](#cadmus-infrastructure)
  - [Adding Docker Support](#adding-docker-support)

For what follows please see the reference [shell app repository](https://github.com/vedph/cadmus-shell).

## Creating the Angular Workspace

To start with, just create a new Angular app with routing support and add Angular Material to it. Eventually, you will add a number of libraries with your own models and editors.

(1) create a new Angular app: `ng new cadmus-<PRJ>-app`: add routing and use CSS.

(2) enter the newly created directory and add Angular Material support: `ng add @angular/material` (Indigo/Pink theme - or whatever you prefer -, setup typography styles=yes, setup animations=yes).

(3) if adding new parts/fragments, you will need to add libraries for them. Each library is added like `ng generate library @myrmidon/cadmus-<PRJ>-<NAME> --prefix <PRJ>`. The typical structure usually includes these libraries (`@myrmidon` here is my NPM name, used as a namespace for all my Cadmus libraries):

- `@myrmidon/cadmus-<PRJ>-core` (optional): core models and eventually services, shared by several parts of the same project.
- `@myrmidon/cadmus-<PRJ>-ui` (optional): UI components shared by several parts of the same project.
- `@myrmidon/cadmus-<PRJ>-part-ui`: parts and fragments editors.
- `@myrmidon/cadmus-<PRJ>-part-pg`: parts and fragments editors wrappers with routing.

(4) to speed up builds, for each added library you can add the corresponding commands to `package.json` scripts (to be run like `npm run <SCRIPTNAME>`), e.g.:

```json
{
  "build-core": "ng build @myrmidon/cadmus-tgr-core && ng build @myrmidon/cadmus-tgr-ui",
  "build-gr": "ng build @myrmidon/cadmus-tgr-part-gr-ui && ng build @myrmidon/cadmus-tgr-part-gr-pg",
  "build-ms": "ng build @myrmidon/cadmus-tgr-part-ms-ui && ng build @myrmidon/cadmus-tgr-part-ms-pg",
  "build-all": "npm run-script build-core && npm run-script build-gr && npm run-script build-ms"
}
```

## Adding Libraries

Some essential libraries are required for the Cadmus frontend. Flex layout is for dynamic CSS flex/grid layouts in Material; auth0 is used for JWT tokens; Akita is used for non-centralized state management; and various @myrmidon libraries provide essential components, authentication and authorization services and components; gravatar is to handle users avatars, moment for human-friendly date and time, rangy for text layers.

Typically you will also need ngx-markdown if you have components displaying Markdown, and ngx-monaco if you components using Markdown or other languages.

(1) add some essential libraries:

```bash
npm i @angular/flex-layout @auth0/angular-jwt @datorama/akita @datorama/akita-ngdevtools @myrmidon/ng-tools @myrmidon/ng-mat-tools @myrmidon/auth-jwt-login @myrmidon/auth-jwt-admin diff-match-patch gravatar moment ngx-markdown ngx-moment@5.0.0 ngx-monaco-editor@9.0.0 rangy

npm i @types/diff-match-patch @types/rangy
```

Please note that currently the libraries specified with a version need to stay with that version because of issues in their latest bits.

(2) typically you will also need some [bricks](https://github.com/vedph/cadmus-bricks-shell), like `@myrmidon/cadmus-refs-doc-references`, `@myrmidon/cadmus-refs-historical-date`, `@myrmidon/cadmus-refs-external-ids`, etc. This depends on the part editors you will use or create. For example:

```bash
npm i @myrmidon/cadmus-refs-doc-references @myrmidon/cadmus-refs-historical-date @myrmidon/cadmus-refs-external-ids @myrmidon/cadmus-mat-physical-size @myrmidon/cadmus-refs-assertion @myrmidon/cadmus-refs-chronotope @myrmidon/cadmus-refs-asserted-chronotope @myrmidon/cadmus-refs-proper-name @myrmidon/cadmus-ui-flags-picker
```

(3) finally, you will need all the libraries from the shell app workspace, which there are just imported as projects in the same workspace:

```bash
npm i @myrmidon/cadmus-api @myrmidon/cadmus-core @myrmidon/cadmus-profile-core @myrmidon/cadmus-state @myrmidon/cadmus-ui @myrmidon/cadmus-ui-pg @myrmidon/cadmus-graph-pg @myrmidon/cadmus-graph-ui @myrmidon/cadmus-item-editor @myrmidon/cadmus-item-list @myrmidon/cadmus-item-search @myrmidon/cadmus-layer-demo @myrmidon/cadmus-thesaurus-editor @myrmidon/cadmus-thesaurus-list @myrmidon/cadmus-thesaurus-ui
```

(you can leave out the graph if you don't use it); and eventually the parts, like e.g.:

```bash
npm i @myrmidon/cadmus-part-general-pg @myrmidon/cadmus-part-general-ui @myrmidon/cadmus-part-philology-ui @myrmidon/cadmus-part-philology-pg
```

## Setting Environment Variables

This is essential to let the frontend find the server, while allowing us to manually edit this URI after building a distribution, and before creating a Docker image.

(1) under `src` add an `env.js` file for project-dependent environment variables, with this content (replace the port number with your backend API port number):

```js
// https://www.jvandemo.com/how-to-use-environment-variables-to-configure-your-angular-application-without-a-rebuild/
(function (window) {
  window.__env = window.__env || {};

  // environment-dependent settings
  window.__env.apiUrl = "http://localhost:60849/api/";
})(this);
```

If you are going to use the [external bibliography API](https://github.com/vedph/cadmus_biblioapi), also add its URL here, e.g.:

```js
window.__env.biblioApiUrl = 'http://localhost:61691/api/';
```

In this case typically you will also need to install `@myrmidon/cadmus-biblio-core @myrmidon/cadmus-biblio-api @myrmidon/cadmus-biblio-ui @myrmidon/cadmus-part-biblio-ui`, which provide the corresponding frontend. Later, in your app's `part-editor-keys.ts`, remember to setup the route to the bibliography part editor like:

```ts
import { EXT_BIBLIOGRAPHY_PART_TYPEID } from '@myrmidon/cadmus-part-biblio-ui';

// ...

export const PART_EDITOR_KEYS: PartEditorKeys = {
  [EXT_BIBLIOGRAPHY_PART_TYPEID]: {
    part: BIBLIO,
  },
  // ... etc.
};
```

(2) in `angular.json`, under `projects/APPNAME/architect/build/options/assets`:

- add `src/env.js`.
- add a glob for monaco. The result would be something like this (see <https://www.npmjs.com/package/ngx-monaco-editor>):

```json
"assets": [
  "src/favicon.ico",
  "src/assets",
  "src/env.js",
  {
    "glob": "**/*",
    "input": "node_modules/ngx-monaco-editor/assets/monaco",
    "output": "/assets/monaco"
  }
],
```

(3) in `src/index.html` add an import for `env.js` to your `head` element:

```html
<head>
...
<script src="env.js"></script>
</head>
```

Also, you can change the web app's title in `head`. Ensure that the root component there is named after `app.component`'s selector (e.g. usually `app-root`).

## Tuning Angular Settings

This is suggested to enable source maps in production and avoid nasty warnings after compilation.

(1) in `angular.json`: following the suggestions in <https://stackoverflow.com/questions/54891679/how-do-i-get-source-map-working-for-npm-linked-angular-library>, and the [Angular docs](https://angular.io/guide/workspace-config#optimization-and-source-map-configuration), explicitly opt for the source maps (under `projects/app/architect/build/options`):

```json
"sourceMap": {
  "scripts": true,
  "styles": true,
  "hidden": false,
  "vendor": true
},
```

(2) in the same file, you will typically have to raise the warning limits for your `budget` size if getting a warning after building.

## Assets

This is optional and depends on your visuals.

(1) ensure you have the required icons and images in `assets` (see `cadmus-shell/assets`): usually they are `logo-white-40.png` for the top bar logo (you can use your own), and a couple of banner images for the homepage.

## Cadmus Infrastructure

(1) add some extension points, eventually adding new entries for your new parts (see [dynamic lookup](https://github.com/vedph/cadmus_doc/blob/master/core/dynamic-lookup.md)):

- `src/app/index-lookup-definitions.ts` with this content:

```ts
import { IndexLookupDefinitions } from '@myrmidon/cadmus-core';

export const INDEX_LOOKUP_DEFINITIONS : IndexLookupDefinitions = {}
```

- `src/app/item-browser-keys.ts` with this content:

```ts
/**
 * Mapping between item browser keys and their routes, used to avoid
 * long and complex names in the route by replacing the ID with an alias.
 */
export const ITEM_BROWSER_KEYS = {
// e.g. ['it.vedph.item-browser.mongo.hierarchy']: 'hierarchy'
};
```

- `src/app/part-editor-keys.ts`: this is the only file with a real content, the others being just extension points. You must specify here the connection of each part or fragment ID with its hosting library in constant `PART_EDITOR_KEYS`. This has a property named after each part/fragment type ID, with a value equal to an object with `part` equal to the library ID, and optionally `fragments` (when the part is a layer part). This property is an object with a property for each fragment type for the layer part, named after the fragment type ID, with a value equal to the library ID.

(2) add routes in `app-routing.module.ts`, as sampled in this project. You should have routes for:

- static pages, like home;
- user authentication and accounts wrappers pages: login, reset password, register user, manage users;
- Cadmus-specific pages like items list and editor, thesauri list and editor, the parts you want, and eventually the graph page.

You can copy all these pages from the [shell app](https://github.com/vedph/cadmus-shell).

Also ensure that you specify options in the root router module like in the shell app. The following is a generic sample:

```ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

// myrmidon
import {
  AuthJwtAdminGuardService,
  AuthJwtGuardService,
} from '@myrmidon/auth-jwt-login';
import { PendingChangesGuard } from '@myrmidon/cadmus-core';
import { EditorGuardService } from '@myrmidon/cadmus-api';

// locals
import { HomeComponent } from './home/home.component';
import { ManageUsersPageComponent } from './manage-users-page/manage-users-page.component';
import { RegisterUserPageComponent } from './register-user-page/register-user-page.component';
import { ResetPasswordComponent } from './reset-password/reset-password.component';
import { LoginPageComponent } from './login-page/login-page.component';

const routes: Routes = [
  { path: '', redirectTo: 'home', pathMatch: 'full' },
  { path: 'home', component: HomeComponent },
  // auth
  { path: 'login', component: LoginPageComponent },
  {
    path: 'reset-password',
    component: ResetPasswordComponent,
    canActivate: [AuthJwtGuardService],
  },
  {
    path: 'register-user',
    component: RegisterUserPageComponent,
    canActivate: [AuthJwtAdminGuardService],
  },
  {
    path: 'manage-users',
    component: ManageUsersPageComponent,
    canActivate: [AuthJwtAdminGuardService],
  },
  // cadmus
  {
    path: 'demo/layers',
    loadChildren: () =>
      import('@myrmidon/cadmus-layer-demo').then(
        (module) => module.CadmusLayerDemoModule
      ),
  },
  // cadmus - items
  {
    path: 'items',
    loadChildren: () =>
      import('@myrmidon/cadmus-item-list').then(
        (module) => module.CadmusItemListModule
      ),
    canActivate: [AuthJwtGuardService],
  },
  {
    path: 'items/:id',
    loadChildren: () =>
      import('@myrmidon/cadmus-item-editor').then(
        (module) => module.CadmusItemEditorModule
      ),
    canActivate: [AuthJwtGuardService],
    canDeactivate: [PendingChangesGuard],
  },
  {
    path: 'search',
    loadChildren: () =>
      import('@myrmidon/cadmus-item-search').then(
        (module) => module.CadmusItemSearchModule
      ),
    canActivate: [AuthJwtGuardService],
  },
  // cadmus - thesauri
  {
    path: 'thesauri',
    loadChildren: () =>
      import('@myrmidon/cadmus-thesaurus-list').then(
        (module) => module.CadmusThesaurusListModule
      ),
    canActivate: [EditorGuardService],
  },
  {
    path: 'thesauri/:id',
    loadChildren: () =>
      import('@myrmidon/cadmus-thesaurus-editor').then(
        (module) => module.CadmusThesaurusEditorModule
      ),
    canActivate: [EditorGuardService],
  },
  // cadmus - parts
  {
    path: 'items/:iid/general',
    loadChildren: () =>
      import('@myrmidon/cadmus-part-general-pg').then(
        (module) => module.CadmusPartGeneralPgModule
      ),
    canActivate: [AuthJwtGuardService],
  },
  {
    path: 'items/:iid/philology',
    loadChildren: () =>
      import('@myrmidon/cadmus-part-philology-pg').then(
        (module) => module.CadmusPartPhilologyPgModule
      ),
    canActivate: [AuthJwtGuardService],
  },
  // cadmus - graph
  {
    path: 'graph',
    loadChildren: () =>
      import('@myrmidon/cadmus-graph-pg').then(
        (module) => module.CadmusGraphPgModule
      ),
    canActivate: [AuthJwtGuardService],
  },
  // fallback
  { path: '**', component: HomeComponent },
];

@NgModule({
  imports: [
    RouterModule.forRoot(routes, {
      initialNavigation: 'enabled',
      useHash: true,
    }),
  ],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

(3) add all the required module imports to your `app.module.ts` (see the shell app for a sample). Also, be sure to specify the `providers` including `EnvServiceProvider`, DI tokens for extension points, and HTTP interceptor for adding the JWT token to each request made to the backend server.

(4) code your `app.component.*` files like those in the [shell app](https://github.com/vedph/cadmus-shell), eventually customizing it as desired.

## Adding Docker Support

Finally, you can add Docker support to create an image of your frontend app. Use as templates the files in the shell app:

- `Dockerfile`
- `nginx.conf`: the NGINX configuration for serving the web app from the Docker container.
- `docker-compose.yml`
- `docker-compose_linux-vol.yml`: a variation of the preceding, using Linux-hosted volumes for persistent data storage.
- `dockerignore`
