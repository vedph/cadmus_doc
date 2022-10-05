# Adding Preview Infrastructure

## Backend

(1) ensure that all your Cadmus packages are up to date.

(2) in `appsettings.json`, add this entry:

```json
"Preview": {
  "IsEnabled": true
}
```

You can eventually add a different preview factory provider. The standard one is already found in `Cadmus.Api.Services` (`StandardCadmusPreviewFactoryProvider`), and usually this is enough.

(3) in `Startup.cs`, add `GetPreviewer` (see [here](https://github.com/vedph/cadmus_api/blob/master/CadmusApi/Startup.cs)).

(4) in `ConfigureServices`, add this line:

```cs
// previewer
services.AddSingleton(p => GetPreviewer(p));
```

This configures the previewer service. When preview is not enabled, this will just return a do-nothing service.

(5) in `wwwroot` add `preview-profile.json` with your profile, like e.g.:

```json
{
  "RendererFilters": [
    {
      "Keys": "markdown",
      "Id": "it.vedph.renderer-filter.markdown",
      "Options": {
        "MarkdownOpen": "<_md>",
        "MarkdownClose": "</_md>",
        "Format": "html"
      }
    },
    {
      "Keys": "token-extractor",
      "Id": "it.vedph.renderer-filter.mongo-token-extractor",
      "Options": {
        "LocationPattern": "<location>([^<]+)</location>",
        "TextTemplate": "<span class=\"apparatus-lemma\">{text}</span>",
        "TextCutting": true,
        "Mode": 3,
        "Limit": 80,
        "MinusLimit": 5,
        "PlusLimit": 5
      }
    }
  ],
  "JsonRenderers": [
    {
      "Keys": "it.vedph.note",
      "Id": "it.vedph.json-renderer.xslt",
      "Options": {
        "QuoteStripping ": true,
        "Xslt": "<?xml version=\"1.0\" encoding=\"UTF-8\"?><xsl:stylesheet xmlns:xsl=\"http://www.w3.org/1999/XSL/Transform\" xmlns:tei=\"http://www.tei-c.org/ns/1.0\" version=\"1.0\"><xsl:output method=\"html\" encoding=\"UTF-8\" omit-xml-declaration=\"yes\"/><xsl:template match=\"tag\"><p class=\"muted\"><xsl:value-of select=\".\"/></p></xsl:template><xsl:template match=\"text\"><div><_md><xsl:value-of select=\".\"/></_md></div></xsl:template><xsl:template match=\"root\"><xsl:apply-templates/></xsl:template><xsl:template match=\"*\"/></xsl:stylesheet>",
        "FilterKeys": "markdown"
      }
    },
    {
      "Keys": "it.vedph.token-text-layer|fr.it.vedph.apparatus",
      "Id": "it.vedph.json-renderer.xslt",
      "Options": {
        "FilterKeys": [
          "token-extractor"
        ],
        "Xslt": "<?xml version=\"1.0\" encoding=\"UTF-8\"?><xsl:stylesheet xmlns:xsl=\"http://www.w3.org/1999/XSL/Transform\" xmlns:xs=\"http://www.w3.org/2001/XMLSchema\" exclude-result-prefixes=\"xs\" version=\"1.0\"><xsl:output method=\"html\" /><xsl:template match=\"lemma\"><span class=\"apparatus-lemma\"><xsl:value-of select=\".\"/></span></xsl:template><xsl:template match=\"location\"><location><xsl:value-of select=\".\"/></location></xsl:template><xsl:template match=\"witnesses\"><span class=\"apparatus-w-value\"><xsl:value-of select=\"value\"/></span><xsl:if test=\"note\"><span class=\"apparatus-w-note\"><xsl:text> </xsl:text><xsl:value-of select=\"note\"/><xsl:text> </xsl:text></span></xsl:if></xsl:template><xsl:template match=\"authors\"><xsl:text> </xsl:text><span class=\"apparatus-a-value\"><xsl:value-of select=\"value\"/></span><xsl:if test=\"note\"><xsl:text> </xsl:text><span class=\"apparatus-a-note\"><xsl:value-of select=\"note\"/></span></xsl:if><xsl:text> </xsl:text></xsl:template><xsl:template match=\"entries\"><xsl:variable name=\"nr\"><xsl:number/></xsl:variable><xsl:if test=\"$nr &gt; 1\"><span class=\"apparatus-sep\">| </span></xsl:if><xsl:if test=\"tag\"><span class=\"apparatus-tag\"><xsl:value-of select=\"tag\"/></span><xsl:text> </xsl:text></xsl:if><xsl:if test=\"subrange\"><span class=\"apparatus-subrange\"><xsl:value-of select=\"subrange\"/></span><xsl:text> </xsl:text></xsl:if><xsl:if test=\"string-length(value) &gt; 0\"><span class=\"apparatus-value\"><xsl:value-of select=\"value\"/></span><xsl:text> </xsl:text></xsl:if><xsl:choose><xsl:when test=\"type = 0\"><xsl:if test=\"string-length(value) = 0\"><span class=\"apparatus-type\">del. </span></xsl:if></xsl:when><xsl:when test=\"type = 1\"><span class=\"apparatus-type\">ante lemma </span></xsl:when><xsl:when test=\"type = 2\"><span class=\"apparatus-type\">post lemma </span></xsl:when></xsl:choose><xsl:if test=\"note\"><span class=\"apparatus-note\"><xsl:value-of select=\"note\"/></span><xsl:text> </xsl:text></xsl:if><xsl:apply-templates/></xsl:template><xsl:template match=\"root\"><xsl:apply-templates/></xsl:template><xsl:template match=\"*\"/></xsl:stylesheet>"
      }
    }
  ],
  "TextPartFlatteners": [
    {
      "Keys": "it.vedph.token-text",
      "Id": "it.vedph.text-flattener.token"
    }
  ]
}
```

## Frontend

(1) add packages:

```bash
npm i @myrmidon/cadmus-text-block-view @myrmidon/cadmus-preview-ui @myrmidon/cadmus-preview-pg
```

(2) import these modules in the app module:

```ts
import { CadmusTextBlockViewModule } from '@myrmidon/cadmus-text-block-view';
import { CadmusPreviewUiModule } from '@myrmidon/cadmus-preview-ui';
import { CadmusPreviewPgModule } from '@myrmidon/cadmus-preview-pg';
```

- `CadmusTextBlockViewModule`
- `CadmusPreviewUiModule`
- `CadmusPreviewPgModule`

(3) in app routing module add the route:

```ts
  // cadmus - preview
  {
    path: 'preview',
    loadChildren: () =>
      import('@myrmidon/cadmus-preview-pg').then(
        (module) => module.CadmusPreviewPgModule
      ),
    canActivate: [AuthJwtGuardService],
  },
```

(4) under your app `styles.css` (or `.scss`) add `@import "preview-styles.css";` and create a new file `preview-styles.css` with your preview styles, like e.g.:

```css
/*
Preview styles. By convention they all start with prefix "pv-".
*/

.pv-muted {
  color: silver;
}
.cadmus-text-block-view-col {
  border: 1px solid transparent;
  padding: 1px;
}
.cadmus-text-block-view-col:hover {
  border: 1px solid #ffdd26;
  padding: 1px;
}

/* layer ID-based styles */

div[class^="it.vedph.token-text-layer|fr.it.vedph.apparatus"],
div[class*=" it.vedph.token-text-layer|fr.it.vedph.apparatus"] {
  background-color: lightsalmon;
}

div[class^="it.vedph.token-text-layer|fr.it.vedph.comment"],
div[class*=" it.vedph.token-text-layer|fr.it.vedph.comment"] {
  background-color: palegreen;
}

div[class^="it.vedph.token-text-layer|fr.it.vedph.orthography"],
div[class*=" it.vedph.token-text-layer|fr.it.vedph.orthography"] {
  background-color: palevioletred;
}

/* apparatus layer */

.apparatus-lemma {
  padding: 2px 4px;
  border: 1px solid silver;
  border-radius: 4px;
  margin-right: 4px;
  color: #065e1d;
}

.apparatus-w-value {
  font-weight: bold;
}

.apparatus-w-note {
  font-style: italic;
}

.apparatus-a-value {
  font-style: italic;
}

.apparatus-a-note {
  font-style: italic;
}

.apparatus-sep {
  margin-left: 0.75em;
}

.apparatus-tag {
  font-style: italic;
}

.apparatus-subrange {
  color: silver;
}

.apparatus-value {
  color: #b8690f;
}

.apparatus-type {
  font-style: italic;
}

.apparatus-note {
  font-style: italic;
}
```
