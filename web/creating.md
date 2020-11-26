# Creating a Cadmus App

## Creating the Angular Workspace

1. create a new Angular app: `ng new cadmus-YOURPRJNAME-app --prefix cadmus`: accept strict mode, add routing and use CSS.

2. if adding new parts/fragments, you will need to add libraries for them.
   Each library is added like `ng generate library @myrmidon/cadmus-tgr-core --prefix cadmus`. The typical structure usually includes these libraries:

- `@myrmidon/cadmus-PRJNAME-core`: core models.
- `@myrmidon/cadmus-PRJNAME-ui`: shared UI components.
- `@myrmidon/cadmus-PRJNAME-part-ui`: parts and fragments editors.
- `@myrmidon/cadmus-PRJNAME-part-pg`: parts and fragments editors wrappers with routing.

When we generate the library (`ng generate library`), Angular CLI adds library project to the `paths` entry in the `tsconfig.json` in the root of our project. This means that whenever in your TypeScript code you import something from `mylib`, it is actually being imported from the `dist/mylib` directory, where the build of our library has been saved.

Once you publish the library to the actual npm repository and want to start using that version, you won’t have to change any imports in the source of the application: just remove that `paths` entry.

To make the library available on npm, create a production build (`ng build LIB --prod`), and run `npm publish` from the project’s `dist` directory:

```ps1
ng build mylib --prod
cd dist/mylib
npm publish
```

General directions for library authoring:

- remember to list in `src/public-api.ts` all what you want to be exported from your library.

- when exporting a plain class, prefix its declaration with a `// @dynamic` comment: see <https://github.com/angular/angular/issues/18867#issuecomment-357484102>. Otherwise, you will get strange errors about metadata generated for exported symbols.

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

3. to speed up the builds, for each added library you can add the corresponding commands to `package.json` scripts, e.g.:

```json
[
  "build-lib-core": "ng build cadmus-core --prod && ng build cadmus-material --prod && ng build cadmus-api --prod && ng build cadmus-ui --prod && ng build cadmus-state --prod && ng build cadmus-ui-pg --prod",
  "build-lib-p-general": "ng build cadmus-part-general-ui --prod && ng build cadmus-part-general-pg --prod",
  "build-lib-p-philology": "ng build cadmus-part-philology-ui --prod && ng build cadmus-part-philology-pg --prod",
  "build-app-pg": "ng build cadmus-admin --prod && ng build cadmus-browser-hierarchy --prod && ng build cadmus-item-editor --prod && ng build cadmus-item-list --prod && ng build cadmus-item-search --prod && ng build cadmus-layer-demo --prod && ng build cadmus-login --prod && ng build cadmus-reset-password --prod && ng build cadmus-thesaurus-editor --prod && ng build cadmus-thesaurus-list --prod && ng build cadmus-user --prod",
  "build-all": "npm run-script build-lib-core && npm run-script build-lib-p-general && npm run-script build-lib-p-philology && npm run-script build-app-pg"
]
```

### Consuming Library from Local NPM

You can build a library as an npm package and then publish it to the local node package manager's registry, so that we can use it in any angular projects which are in our local machine.

1. once you have built the library, open the Terminal in your library folder from the `dist/LIBNAME` directory, and then write a command to create the Pack file: `npm pack`.

2. copy the absolute path of the library project (up to `...dist/LIBNAME/LIBNAME.TGZ`).

3. from your consumer app root folder, use `npm install PATH-TO-LIB-TGZ`.

## Creating the App

Once you have created the workspace as explained above:

1. add Material support: `ng add @angular/material`.

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
"@myrmidon/cadmus-admin": "^0.0.5",
"@myrmidon/cadmus-api": "^0.0.6",
"@myrmidon/cadmus-core": "^0.0.6",
"@myrmidon/cadmus-item-editor": "^0.0.5",
"@myrmidon/cadmus-item-list": "^0.0.6",
"@myrmidon/cadmus-item-search": "^0.0.8",
"@myrmidon/cadmus-login": "^0.0.5",
"@myrmidon/cadmus-material": "^0.0.4",
"@myrmidon/cadmus-part-general-pg": "^0.0.5",
"@myrmidon/cadmus-part-general-ui": "^0.0.10",
"@myrmidon/cadmus-part-philology-pg": "^0.0.5",
"@myrmidon/cadmus-part-philology-ui": "^0.0.5",
"@myrmidon/cadmus-reset-password": "^0.0.5",
"@myrmidon/cadmus-state": "^0.0.6",
"@myrmidon/cadmus-thesaurus-editor": "^0.0.7",
"@myrmidon/cadmus-thesaurus-list": "^0.0.5",
"@myrmidon/cadmus-ui": "^0.0.14",
"@myrmidon/cadmus-ui-pg": "^0.0.5",
"@myrmidon/cadmus-user": "^0.0.5"
```

3. in `angular.json`, under `projects/APPNAME/architect/build/options/assets`:

- add `src/env.js`. You must change all the data there, to fit your project: name, database name, and API port:

```js
// https://www.jvandemo.com/how-to-use-environment-variables-to-configure-your-angular-application-without-a-rebuild/
(function (window) {
  window.__env = window.__env || {};

  // environment-dependent settings
  window.__env.apiUrl = "http://localhost:54183/api/";
  window.__env.databaseId = "cadmus-itinera";
  window.__env.name = "Itinera";
})(this);
```

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

4. in `angular.json`: following the suggestions in <https://stackoverflow.com/questions/54891679/how-do-i-get-source-map-working-for-npm-linked-angular-library>, and the [Angular docs]<https://angular.io/guide/workspace-config#optimization-and-source-map-configuration>, explicitly opt for the source maps (under `projects/app/architect/build/options`):

```json
"sourceMap": {
  "scripts": true,
  "styles": true,
  "hidden": false,
  "vendor": true
},
"preserveSymlinks": true
```

Here, `preserveSymlinks` is added as a sample for the "real" apps which will consume the libraries, to let them preserve symlinks to `npm link`-ed libraries).

Also, the `vendorSourceMap` option suggested in the above link no more belongs to the JSON schema. I found it in many older posts about sourcemaps, but it was deprecated in 7 (<https://blog.ninja-squad.com/2019/01/09/angular-cli-7.2/>), and replaced by the `vendor` property in `sourceMap`.

5. in `angular.json`, also add `"sourceMap": true` to the shell app's `angular.json` at `projects/app/architect/build/configurations/production`, as anyway I'm not going to hide any maps, because this is an open source project; so I followed the advice from [this post](https://medium.com/angular-in-depth/debug-angular-apps-in-production-without-revealing-source-maps-ab4a235edd85).

6. in `tsconfig.json`:

- add `"allowSyntheticDefaultImports": true`.

7. in `index.html`:

- add this script before `</head>` (remember to check that you have included it in `angular.json`!):

```html
<!-- Load environment variables -->
<script src="env.js"></script>
```

Ensure that the root component there is named after app.component's selector (e.g. `cadmus-root`).

8. `styles.css`: add these imports:

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

9. ensure you have the required icons and images in `assets` (see `cadmus-shell/assets`).

10. add the required constants in files, eventually removing what is not needed and adding new entries for your new parts:

- `src/app/index-lookup-definitions.ts`
- `src/app/item-browser-keys.ts`
- `src/app/part-editor-keys.ts`

11. add the required metadata in `app.module.ts` (see `cadmus-shell`).

12. copy `app` component from `cadmus-shell` and eventually customize it.

13. copy `home` component from `cadmus-shell` and eventually customize it.

14. if you want to include demos, copy the `demo` folder.
