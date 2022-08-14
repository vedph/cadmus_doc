# Adding Preview

- [Adding Preview](#adding-preview)
  - [Backend API](#backend-api)
  - [Frontend App](#frontend-app)

The preview feature is a new Cadmus subsystem developed in the context of its [migration](https://github.com/vedph/cadmus-migration) capabilities. It can provide a quick "preview" of any Cadmus part or fragment in a more human-friendly way, totally customizable via a dedicated configuration.

To add preview capabilities to an existing API you can follow the steps described here, for both backend and frontend.

## Backend API

(1) in `appsettings.json`, add this entry:

```json
"Preview": {
  "IsEnabled": true
}
```

(2) optionally, add a different preview factory provider. The standard one is already found in `Cadmus.Api.Services` (`StandardCadmusPreviewFactoryProvider`), and usually this is enough.

(3) in `Startup.cs`:

- add `GetPreviewerAsync` (see [Startup.cs](https://github.com/vedph/cadmus_api/blob/master/CadmusApi/Startup.cs)).
- in `ConfigureServices`, add this line:

```cs
// previewer
services.AddSingleton(p => GetPreviewer(p));
```

This configures the previewer service. When preview is not enabled, this will just return a do-nothing service.

(4) in `wwwroot` add `preview-profile.json` with your profile. For more information about this profile, see the [migration](https://github.com/vedph/cadmus-migration) project. Essentially, this is a JSON file used to define the preview of each part or fragment you use in your project. You are not required to add a preview for each of these objects, though; you can just limit yourself to the objects you want. At a minimum, you can add a preview using the builtin modules by just knowing some XSLT (even if Cadmus data is no XML).

## Frontend App

(1) ensure that your Cadmus libraries are up to date.

(2) install the preview libraries: `npm i @myrmidon/cadmus-preview-ui @myrmidon/cadmus-preview-pg`.

(3) add styles for the preview in a new CSS file `preview-styles.css` under your app's `src` folder (next to your `styles.css`). You can find an example [here](https://github.com/vedph/cadmus-shell/blob/master/src/preview-styles.css).

(4) import this file in `styles.css`:

```css
@import "preview-styles.css";
```
