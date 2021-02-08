# Cadmus Frontend App

- [Cadmus Frontend App](#cadmus-frontend-app)
  - [Creating the Angular Workspace](#creating-the-angular-workspace)
    - [Notes on Angular Libraries](#notes-on-angular-libraries)
      - [Consuming Library from Local NPM](#consuming-library-from-local-npm)
  - [Creating the App](#creating-the-app)
    - [Setup](#setup)
    - [Environment Variables](#environment-variables)
    - [Angular Settings](#angular-settings)
    - [Styles and Assets](#styles-and-assets)
    - [Cadmus Infrastructure](#cadmus-infrastructure)
    - [Homepage](#homepage)

## Creating the Angular Workspace

1. create a new Angular app: `ng new cadmus-<PRJ>-app --prefix cadmus`: accept strict mode, add routing and use CSS.

2. if adding new parts/fragments, you will need to add libraries for them.
   Each library is added like `ng generate library @myrmidon/cadmus-<PRJ>-<NAME> --prefix <PRJ>`. The typical structure usually includes these libraries (`@myrmidon` here is my NPM name, used as a namespace for all my Cadmus libraries):

- `@myrmidon/cadmus-<PRJ>-core` (optional): core models and eventually services, shared by several parts of the same project.
- `@myrmidon/cadmus-<PRJ>-ui` (optional): UI components shared by several parts of the same project.
- `@myrmidon/cadmus-<PRJ>-part-ui`: parts and fragments editors.
- `@myrmidon/cadmus-<PRJ>-part-pg`: parts and fragments editors wrappers with routing.

3. to speed up builds, for each added library you can add the corresponding commands to `package.json` scripts (to be run like `npm run <SCRIPTNAME>`), e.g.:

```json
{
  "build-core": "ng build @myrmidon/cadmus-tgr-core --prod && ng build @myrmidon/cadmus-tgr-ui --prod",
  "build-gr": "ng build @myrmidon/cadmus-tgr-part-gr-ui --prod && ng build @myrmidon/cadmus-tgr-part-gr-pg --prod",
  "build-ms": "ng build @myrmidon/cadmus-tgr-part-ms-ui --prod && ng build @myrmidon/cadmus-tgr-part-ms-pg --prod",
  "build-all": "npm run-script build-core && npm run-script build-gr && npm run-script build-ms"
}
```

### Notes on Angular Libraries

When we generate the library (`ng generate library`), Angular CLI adds library project to the `paths` entry in the `tsconfig.json` in the root of our project. This means that whenever in your TypeScript code you import something from `mylib`, it is actually being imported from the `dist/mylib` directory, where the build of our library has been saved.

Once you publish the library to the actual npm repository and want to start using that version, you won't have to change any imports in the source of the application: just remove that `paths` entry.

To make the library available on npm, create a production build (`ng build <LIB> --prod`), and run `npm publish` from the project’s `dist` directory:

```ps1
ng build mylib --prod
cd dist/mylib
npm publish
```

General directions for library authoring:

- remember to list in `src/public-api.ts` all what you want to be exported from your library.

- when exporting a plain class, prefix its declaration with a `// @dynamic` comment: see <https://github.com/angular/angular/issues/18867#issuecomment-357484102>. Otherwise, you will get errors about metadata generated for exported symbols.

- when using child routes, define them as follows:

```ts
// https://github.com/ng-packagr/ng-packagr/issues/778
export const RouterModuleForChild = RouterModule.forChild([
  // add your routes here...
]);
```

and then just add `RouterModuleForChild` to your library module `imports`.

- to avoid UMD bundling warnings (like "No name was provided for external module..."), in the lib's `ng-package.json` add a `umdModuleIds` section with explicit mappings for each module listed in the warnings (<https://stackoverflow.com/questions/48616267/ng-packagr-gives-no-name-was-provided-for-external-module/53521270>), e.g.:

```json
"lib": {
  "entryFile": "src/public-api.ts",
  "umdModuleIds": {
    "cadmus-core": "cadmus-core",
    "cadmus-material": "cadmus-material",
    "cadmus-api": "cadmus-api",
    "cadmus-ui": "cadmus-ui"
  }
}
```

Before consuming the library, you must always build it: `ng build LIBNAME --prod` (mind the `--prod` flag! This allows the library to be consumed by both non-Ivy and Ivy clients).

You can then import it in the host app as any other library, like `import { MyDemoLibModule } from '@myrmidon/my-lib';`.

#### Consuming Library from Local NPM

You can build a library as an npm package and then publish it to the local node package manager's registry, so that we can use it in any angular projects which are in our local machine.

1. once you have built the library, open the Terminal in your library folder from the `dist/LIBNAME` directory, and then write a command to create the Pack file: `npm pack`.

2. copy the absolute path of the library project (up to `...dist/LIBNAME/LIBNAME.TGZ`).

3. from your consumer app root folder, use `npm install PATH-TO-LIB-TGZ`.

## Creating the App

### Setup

Once you have created the workspace:

1. add Material support: `ng add @angular/material` (Indigo/Pink theme - or whatever you prefer -, setup typography styles=yes, setup animations=yes).

2. add these packages (to `package.json`) and run `npm i` (note that the version numbers will change; as for the Cadmus part libraries, add only those required by your project):

```json
"@angular/flex-layout": "^11.0.0-beta.33",
"@auth0/angular-jwt": "^5.0.1",
"@datorama/akita": "^5.2.3",
"@datorama/akita-ngdevtools": "^5.0.3",
"@types/diff-match-patch": "^1.0.32",
"diff-match-patch": "^1.0.5",
"gravatar": "^1.8.1",
"moment": "^2.28.0",
"ngx-markdown": "^10.1.1",
"ngx-moment": "^5.0.0",
"ngx-monaco-editor": "^9.0.0",
"rangy": "^1.3.0",
"@myrmidon/cadmus-admin": "0.0.6",
"@myrmidon/cadmus-api": "0.0.9",
"@myrmidon/cadmus-core": "0.0.15",
"@myrmidon/cadmus-item-editor": "0.0.10",
"@myrmidon/cadmus-item-list": "0.0.6",
"@myrmidon/cadmus-item-search": "0.0.8",
"@myrmidon/cadmus-login": "0.0.5",
"@myrmidon/cadmus-material": "0.0.5",
"@myrmidon/cadmus-part-general-pg": "0.0.9",
"@myrmidon/cadmus-part-general-ui": "0.0.18",
"@myrmidon/cadmus-part-philology-pg": "0.0.8",
"@myrmidon/cadmus-part-philology-ui": "0.0.10",
"@myrmidon/cadmus-reset-password": "0.0.5",
"@myrmidon/cadmus-state": "0.0.10",
"@myrmidon/cadmus-thesaurus-editor": "0.0.7",
"@myrmidon/cadmus-thesaurus-list": "0.0.5",
"@myrmidon/cadmus-ui": "0.0.25",
"@myrmidon/cadmus-ui-pg": "0.0.7",
"@myrmidon/cadmus-user": "0.0.5",
```

### Environment Variables

1. under `src` add an `env.js` file for project-dependent environment variables, with this content (replace the port number with your backend API port number, and the database ID with that of your project, e.g. `cadmus-pura`):

```js
// https://www.jvandemo.com/how-to-use-environment-variables-to-configure-your-angular-application-without-a-rebuild/
(function (window) {
  window.__env = window.__env || {};

  // environment-dependent settings
  window.__env.apiUrl = "http://localhost:60849/api/";
  window.__env.databaseId = "cadmus-<PRJ>";
})(this);
```

2. in `angular.json`, under `projects/APPNAME/architect/build/options/assets`:

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

3. in `src/index.html` add an import for `env.js` to your `head` element:

```html
<head>
...
<script src="env.js"></script>
</head>
```

Also ensure that the root component there is named after app.component's selector (e.g. usually `cadmus-root`; this depends on the suffix you specified when creating the app).

### Angular Settings

1. in `angular.json`: following the suggestions in <https://stackoverflow.com/questions/54891679/how-do-i-get-source-map-working-for-npm-linked-angular-library>, and the [Angular docs]<https://angular.io/guide/workspace-config#optimization-and-source-map-configuration>, explicitly opt for the source maps (under `projects/app/architect/build/options`):

```json
"sourceMap": {
  "scripts": true,
  "styles": true,
  "hidden": false,
  "vendor": true
},
"preserveSymlinks": true
```

Here, `preserveSymlinks` is added as a sample for the "real" apps which will consume the libraries, to let them preserve symlinks to `npm link`-ed libraries.

Also, the `vendorSourceMap` option suggested in the above link no more belongs to the JSON schema. I found it in many older posts about sourcemaps, but it was deprecated in 7 (<https://blog.ninja-squad.com/2019/01/09/angular-cli-7.2/>), and replaced by the `vendor` property in `sourceMap`.

2. in `angular.json`, also add `"sourceMap": true` to the shell app's `angular.json` at `projects/app/architect/build/configurations/production`, as anyway I'm not going to hide any maps, because this is an open source project; so I followed the advice from [this post](https://medium.com/angular-in-depth/debug-angular-apps-in-production-without-revealing-source-maps-ab4a235edd85).

3. in `tsconfig.json` add `"allowSyntheticDefaultImports": true`.

### Styles and Assets

1. in `styles.css` add these imports:

```css
@import "~@angular/material/prebuilt-themes/indigo-pink.css";
/* Offline material icons: used for developing.
   To remove, just comment out this import.
   The local font files are under assets/icons. If you want to totally
   remove local icons, just delete the whole folder besides removing
   this import. Their source is
   https://github.com/google/material-design-icons/tree/master/iconfont
   (when downloading from there, open each file in GitHub and click Download,
   as right clicking on the file name and selecting Download link as won't
   work as expected, producing corrupt files).
 */
/* @import "assets/icons/material-icons.css"; */
```

10. ensure you have the required icons and images in `assets` (see `cadmus-shell/assets`): usually they are `logo-white-40.png` for the top bar logo (you can use your own), and a banner for the homepage.

### Cadmus Infrastructure

1. add the required constants in files, eventually removing what is not needed and adding new entries for your new parts:

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
//  ['it.vedph.item-browser.mongo.hierarchy']: 'hierarchy'
};
```

- `src/app/part-editor-keys.ts`: this is the only file with a real content, the others being just extension points. You must specify here the connection of each part or fragment ID with its hosting library in constant `PART_EDITOR_KEYS`. This has a property named after each part/fragment type ID, with a value equal to an object with `part` equal to the library ID, and optionally `fragments` (when the part is a layer part). This property is an object with a property for each fragment type for the layer part, named after the fragment type ID, with a value equal to the library ID. Here is a sample, which draws some parts/fragments from the core libraries, and some others from the project's own libraries:

```ts
import {
  CATEGORIES_PART_TYPEID,
  HISTORICAL_DATE_PART_TYPEID,
  KEYWORDS_PART_TYPEID,
  INDEX_KEYWORDS_PART_TYPEID,
  NOTE_PART_TYPEID,
  TOKEN_TEXT_PART_TYPEID,
  COMMENT_FRAGMENT_TYPEID,
  BIBLIOGRAPHY_PART_TYPEID,
  CHRONOLOGY_FRAGMENT_TYPEID,
} from '@myrmidon/cadmus-part-general-ui';
import {
  APPARATUS_FRAGMENT_TYPEID,
  ORTHOGRAPHY_FRAGMENT_TYPEID,
  QUOTATIONS_FRAGMENT_TYPEID,
} from '@myrmidon/cadmus-part-philology-ui';
import { PartEditorKeys } from '@myrmidon/cadmus-core';
import {
  LING_TAGS_FRAGMENT_TYPEID,
  VAR_QUOTATIONS_FRAGMENT_TYPEID,
  INTERPOLATIONS_FRAGMENT_TYPEID,
  AVAILABLE_WITNESSES_PART_TYPEID,
} from '@myrmidon/cadmus-tgr-part-gr-ui';
import { MSSIGNATURES_PART_TYPEID } from '@myrmidon/cadmus-itinera-part-ms-ui';
import {
  MSCONTENTS_PART_TYPEID,
  MSFORMAL_FEATURES_PART_TYPEID,
  MSHISTORY_PART_TYPEID,
  MSORNAMENTS_PART_TYPEID,
  MSPLACES_PART_TYPEID,
  MSSCRIPTS_PART_TYPEID,
  MSUNITS_PART_TYPEID,
} from '@myrmidon/cadmus-tgr-part-ms-ui';

const GENERAL = 'general';
const PHILOLOGY = 'philology';
const ITINERA_MS = 'itinera-ms';
const TGR_GR = 'tgr-gr';
const TGR_MS = 'tgr-ms';
const TOKEN_TEXT_LAYER_PART_TYPEID = 'it.vedph.token-text-layer';

/**
 * The parts and fragments editor keys for this UI.
 * Each property is a part type ID, mapped to a value of type PartGroupKey,
 * having a part property with the part's editor key, and a fragments property
 * with the mappings between fragment type IDs and their editor keys.
 */
export const PART_EDITOR_KEYS: PartEditorKeys = {
  [BIBLIOGRAPHY_PART_TYPEID]: {
    part: GENERAL,
  },
  [CATEGORIES_PART_TYPEID]: {
    part: GENERAL,
  },
  [HISTORICAL_DATE_PART_TYPEID]: {
    part: GENERAL,
  },
  [INDEX_KEYWORDS_PART_TYPEID]: {
    part: GENERAL,
  },
  [KEYWORDS_PART_TYPEID]: {
    part: GENERAL,
  },
  [NOTE_PART_TYPEID]: {
    part: GENERAL,
  },
  [TOKEN_TEXT_PART_TYPEID]: {
    part: GENERAL,
  },
  [AVAILABLE_WITNESSES_PART_TYPEID]: {
    part: TGR_GR,
  },
  [MSSIGNATURES_PART_TYPEID]: {
    part: ITINERA_MS,
  },
  [MSPLACES_PART_TYPEID]: {
    part: TGR_MS,
  },
  [MSCONTENTS_PART_TYPEID]: {
    part: TGR_MS,
  },
  [MSUNITS_PART_TYPEID]: {
    part: TGR_MS,
  },
  [MSSCRIPTS_PART_TYPEID]: {
    part: TGR_MS,
  },
  [MSFORMAL_FEATURES_PART_TYPEID]: {
    part: TGR_MS,
  },
  [MSHISTORY_PART_TYPEID]: {
    part: TGR_MS,
  },
  [MSORNAMENTS_PART_TYPEID]: {
    part: TGR_MS,
  },
  // layer parts
  [TOKEN_TEXT_LAYER_PART_TYPEID]: {
    part: GENERAL,
    fragments: {
      [CHRONOLOGY_FRAGMENT_TYPEID]: GENERAL,
      [COMMENT_FRAGMENT_TYPEID]: GENERAL,
      [APPARATUS_FRAGMENT_TYPEID]: PHILOLOGY,
      [ORTHOGRAPHY_FRAGMENT_TYPEID]: PHILOLOGY,
      [QUOTATIONS_FRAGMENT_TYPEID]: PHILOLOGY,
      [VAR_QUOTATIONS_FRAGMENT_TYPEID]: TGR_GR,
      [INTERPOLATIONS_FRAGMENT_TYPEID]: TGR_GR,
      [LING_TAGS_FRAGMENT_TYPEID]: TGR_GR,
    },
  },
};
```

2. delete `app-routing.module.ts`, and replace the content of `app.module.ts` with something like this, customizing the paths to your part/fragment editor libraries as required (see `cadmus-shell`):

```ts
import { BrowserModule } from '@angular/platform-browser';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { RouterModule } from '@angular/router';
import { NgModule } from '@angular/core';
import { ReactiveFormsModule, FormsModule } from '@angular/forms';
import { HttpClientModule, HTTP_INTERCEPTORS } from '@angular/common/http';

import { AppComponent } from './app.component';

// flex
import { FlexLayoutModule } from '@angular/flex-layout';
// ngx monaco
import { MonacoEditorModule } from 'ngx-monaco-editor';
// ngx markdown
import { MarkdownModule } from 'ngx-markdown';
// Akita
import { AkitaNgDevtools } from '@datorama/akita-ngdevtools';

import {
  CadmusCoreModule,
  PendingChangesGuard,
  EnvServiceProvider,
} from '@myrmidon/cadmus-core';
import { CadmusUiModule } from '@myrmidon/cadmus-ui';
import { CadmusPartGeneralUiModule } from '@myrmidon/cadmus-part-general-ui';
import { CadmusPartPhilologyUiModule } from '@myrmidon/cadmus-part-philology-ui';
import { HomeComponent } from './home/home.component';
import { CadmusMaterialModule } from '@myrmidon/cadmus-material';
import {
  AuthInterceptor,
  AdminGuardService,
  AuthGuardService,
  EditorGuardService,
} from '@myrmidon/cadmus-api';
import { PART_EDITOR_KEYS } from './part-editor-keys';
import { ITEM_BROWSER_KEYS } from './item-browser-keys';
import { INDEX_LOOKUP_DEFINITIONS } from './index-lookup-definitions';

@NgModule({
  declarations: [
    AppComponent,
    HomeComponent,
  ],
  imports: [
    BrowserModule,
    BrowserAnimationsModule,
    FormsModule,
    HttpClientModule,
    ReactiveFormsModule,
    RouterModule.forRoot(
      [
        { path: '', redirectTo: 'home', pathMatch: 'full' },
        { path: 'home', component: HomeComponent },
        {
          path: 'login',
          loadChildren: () =>
            import('@myrmidon/cadmus-login').then((module) => module.CadmusLoginModule),
        },
        {
          path: 'items',
          loadChildren: () =>
            import('@myrmidon/cadmus-item-list').then(
              (module) => module.CadmusItemListModule
            ),
          canActivate: [AuthGuardService],
        },
        {
          path: 'items/:id',
          loadChildren: () =>
            import('@myrmidon/cadmus-item-editor').then(
              (module) => module.CadmusItemEditorModule
            ),
          canActivate: [AuthGuardService],
          canDeactivate: [PendingChangesGuard],
        },
        {
          path: 'items/:iid/general',
          loadChildren: () =>
            import('@myrmidon/cadmus-part-general-pg').then(
              (module) => module.CadmusPartGeneralPgModule
            ),
          canActivate: [AuthGuardService],
        },
        {
          path: 'items/:iid/philology',
          loadChildren: () =>
            import('@myrmidon/cadmus-part-philology-pg').then(
              (module) => module.CadmusPartPhilologyPgModule
            ),
          canActivate: [AuthGuardService],
        },
        {
          path: 'items/:iid/itinera-ms',
          loadChildren: () =>
            import('@myrmidon/cadmus-itinera-part-ms-pg').then(
              (module) => module.CadmusItineraPartMsPgModule
            ),
          canActivate: [AuthGuardService],
        },
        {
          path: 'items/:iid/tgr-gr',
          loadChildren: () =>
            import('@myrmidon/cadmus-tgr-part-gr-pg').then(
              (module) => module.CadmusTgrPartGrPgModule
            ),
          canActivate: [AuthGuardService],
        },
        {
          path: 'items/:iid/tgr-ms',
          loadChildren: () =>
            import('@myrmidon/cadmus-tgr-part-ms-pg').then(
              (module) => module.CadmusTgrPartMsPgModule
            ),
          canActivate: [AuthGuardService],
        },
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
        {
          path: 'admin',
          loadChildren: () =>
            import('@myrmidon/cadmus-admin').then((module) => module.CadmusAdminModule),
          canActivate: [AdminGuardService],
        },
        {
          path: 'user',
          loadChildren: () =>
            import('@myrmidon/cadmus-user').then((module) => module.CadmusUserModule),
          canActivate: [AuthGuardService],
        },
        {
          path: 'reset-password',
          loadChildren: () =>
            import('@myrmidon/cadmus-reset-password').then(
              (module) => module.CadmusResetPasswordModule
            ),
        },
        {
          path: 'search',
          loadChildren: () =>
            import('@myrmidon/cadmus-item-search').then(
              (module) => module.CadmusItemSearchModule
            ),
          canActivate: [AuthGuardService],
        },
        { path: '**', component: HomeComponent },
      ],
      {
    initialNavigation: 'enabled',
    useHash: true,
    relativeLinkResolution: 'legacy'
}
    ),
    // flex
    FlexLayoutModule,
    // Monaco
    MonacoEditorModule.forRoot(),
    // markdown
    MarkdownModule.forRoot(),
    // Akita
    AkitaNgDevtools.forRoot(),
    // Cadmus
    CadmusCoreModule,
    CadmusMaterialModule,
    CadmusPartGeneralUiModule,
    CadmusPartPhilologyUiModule,
    CadmusUiModule
  ],
  providers: [
    EnvServiceProvider,
    // parts and fragments type IDs to editor group keys mappings
    // https://github.com/nrwl/nx/issues/208#issuecomment-384102058
    // inject like: @Inject('partEditorKeys') partEditorKeys: PartEditorKeys
    {
      provide: 'partEditorKeys',
      useValue: PART_EDITOR_KEYS,
    },
    // index lookup definitions
    {
      provide: 'indexLookupDefinitions',
      useValue: INDEX_LOOKUP_DEFINITIONS,
    },
    // item browsers IDs to editor keys mappings
    // inject like: @Inject('itemBrowserKeys') itemBrowserKeys: { [key: string]: string }
    {
      provide: 'itemBrowserKeys',
      useValue: ITEM_BROWSER_KEYS,
    },
    // HTTP interceptor
    // https://medium.com/@ryanchenkie_40935/angular-authentication-using-the-http-client-and-http-interceptors-2f9d1540eb8
    {
      provide: HTTP_INTERCEPTORS,
      useClass: AuthInterceptor,
      multi: true,
    },
  ],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

3. copy the `app` component from `cadmus-shell` and eventually customize it:

```css
footer {
  background-color: #f9f9f9;
  color: #808080;
  padding: 4px;
  text-align: center;
  margin-top: 8px;
  padding-top: 8px;
  font-size: 95%;
}

footer a {
  color: #808080;
}

.active {
  font-size: 100% !important;
}

mat-icon.small-icon {
  font-size: 85% !important;
  margin-left: 8px;
}

.tb-fill-remaining-space {
  flex: 1 1 auto;
}
```

```html
<header>
  <mat-toolbar color="primary" fxLayout="row" fxLayoutAlign="start center">
    <span style="flex: 0 0 60px"
      ><img src="./assets/img/logo-white-40.png" alt="Fusisoft"
    /></span>
    <a mat-button routerLink="/home">Cadmus __PRJ__</a>

    <button
      mat-button
      [matMenuTriggerFor]="itemMenu"
      *ngIf="logged && itemBrowsers"
    >
      Items
    </button>
    <mat-menu #itemMenu>
      <a mat-menu-item routerLink="/items">Items</a>
      <a
        mat-menu-item
        *ngFor="let entry of itemBrowsers"
        [routerLink]="'item-browser/' + getItemBrowserRoute(entry.id)"
        >{{ entry.value }}</a
      >
    </mat-menu>
    <ng-container *ngIf="logged && !itemBrowsers">
      <a mat-button routerLink="/items">Items</a>
    </ng-container>

    <a mat-button routerLink="/search" *ngIf="logged">Search</a>

    <a
      mat-button
      routerLink="/thesauri"
      *ngIf="
        user && (user?.roles?.includes('admin') || user?.roles?.includes('editor'))
      "
      >Thesauri</a
    >

    <span class="tb-fill-remaining-space"></span>

    <div *ngIf="logged" fxLayout="row" fxLayoutAlign="start center">
      <img [src]="getGravatarUrl(user?.email, 32)" [alt]="user?.userName" />
      <mat-icon
        class="small-icon"
        *ngIf="user && user?.roles?.includes('admin')"
        title="admin"
        >build</mat-icon
      >
      <mat-icon
        class="small-icon"
        *ngIf="user && !user?.emailConfirmed"
        title="You must verify your email address! Please check your mailbox {{
          user?.email
        }}"
        >feedback</mat-icon
      >
      <button mat-icon-button [mat-menu-trigger-for]="menu">
        <mat-icon>more_vert</mat-icon>
      </button>
      <mat-menu x-position="before" #menu="matMenu">
        <a
          mat-menu-item
          *ngIf="user && user?.roles?.includes('admin')"
          routerLink="/admin"
          >Admin</a
        >
        <a mat-menu-item (click)="logout()">Logout</a>
      </mat-menu>
    </div>

    <div *ngIf="!logged">
      <a mat-button routerLink="/login">Login</a>
    </div>
  </mat-toolbar>
</header>

<main>
  <router-outlet></router-outlet>
</main>

<footer>
  <div layout="row" layout-align="center center">
    <p>
      TODO: institutions here...
      - Cadmus by
      <a href="https://www.fusisoft.net" target="_blank">Daniele Fusi</a> at
      <a href="https://www.unive.it/pag/39287" target="_blank">VeDPH</a>
    </p>
  </div>
</footer>
```

```ts
import { Component, OnInit, Inject } from '@angular/core';
import { User, GravatarService, Thesaurus, ThesaurusEntry } from '@myrmidon/cadmus-core';
import { AuthService } from '@myrmidon/cadmus-api';
import { AppService, AppQuery } from '@myrmidon/cadmus-state';

@Component({
  selector: 'cadmus-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  public user?: User;
  public logged?: boolean;
  public itemBrowsers?: ThesaurusEntry[] | null;

  constructor(
    @Inject('itemBrowserKeys')
    private _itemBrowserKeys: { [key: string]: string },
    private _authService: AuthService,
    private _gravatarService: GravatarService,
    private _appService: AppService,
    private _appQuery: AppQuery
  ) {
    this.user = undefined;
    this.logged = false;
    this.itemBrowsers = [];
  }

  ngOnInit(): void {
    this.user = this._authService.currentUserValue;
    this.logged = this.user !== null;

    this._authService.currentUser$.subscribe((user: User) => {
      this.logged = this._authService.isAuthenticated(true);
      this.user = user;
      // load the general app state just once
      if (user) {
        this._appService.load();
      }
    });

    this._appQuery
      .selectItemBrowserThesaurus()
      .subscribe((thesaurus: Thesaurus) => {
        this.itemBrowsers = thesaurus ? thesaurus.entries : null;
      });
  }

  public getItemBrowserRoute(id: string): string {
    return this._itemBrowserKeys[id] || id;
  }

  public getGravatarUrl(email?: string, size = 80): string {
    if (!email) {
      return '';
    }
    return this._gravatarService.buildGravatarUrl(email, size);
  }

  public logout(): void {
    if (!this.logged) {
      return;
    }
    this._authService.logout();
  }
}
```

### Homepage

Copy the `home` component from `cadmus-shell` and eventually customize it. The typical code for it is like this:

```css
#banner {
  height: 258px;
  background-image: url("/assets/img/banner.png");
  background-size: cover;
  margin-bottom: 16px;
}
#banner h1 {
  position: relative;
  padding-top: 60px;
  width: auto;
  text-align: center;
  font-size: 100px;
  font-weight: normal;
  letter-spacing: -5px;
  color: #ac7666;
}
article {
  column-width: 600px;
}
```

```html
<div style="margin: 0 16px">
  <div id="banner">
    <h1>TODO: Project title</h1>
  </div>
  <article>
    <h2>Welcome</h2>
    <h3>
      TODO: Project full title
    </h3>
    <p>TODO: info about the project...</p>
    <p *ngIf="!logged" style="margin-bottom: 8px">
      <a mat-mini-fab color="primary" routerLink="/login"
        ><mat-icon>login</mat-icon></a
      ><span style="margin-left: 8px">please login</span>
    </p>
  </article>
</div>
```

```ts
import { Component } from '@angular/core';
import { AuthService } from '@myrmidon/cadmus-api';

@Component({
  selector: 'cadmus-home',
  templateUrl: './home.component.html',
  styleUrls: ['./home.component.css'],
})
export class HomeComponent {
  public logged: boolean;

  constructor(authService: AuthService) {
    this.logged = authService.currentUserValue !== null;
  }
}
```
