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
    },
    {
      "Keys": "iso639-3",
      "Id": "it.vedph.renderer-filter.iso639"
    }
  ],
  "JsonRenderers": [
    {
      "Keys": "it.vedph.note",
      "Id": "it.vedph.json-renderer.xslt",
      "Options": {
        "QuoteStripping ": true,
        "Xslt": "<?xml version=\"1.0\" encoding=\"UTF-8\"?><xsl:stylesheet xmlns:xsl=\"http://www.w3.org/1999/XSL/Transform\" xmlns:tei=\"http://www.tei-c.org/ns/1.0\" version=\"1.0\"><xsl:output media-type=\"text/html\" method=\"html\" omit-xml-declaration=\"yes\" encoding=\"UTF-8\"/><xsl:template match=\"tag[normalize-space(.)]\"><div class=\"pv-muted\"><xsl:value-of select=\".\"/></div></xsl:template><xsl:template match=\"text\"><div class=\"note-text\"><_md><xsl:value-of select=\".\"/></_md></div></xsl:template><xsl:template match=\"root\"><xsl:apply-templates/></xsl:template><xsl:template match=\"*\"/></xsl:stylesheet>",
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
        "Xslt": "<?xml version=\"1.0\" encoding=\"UTF-8\"?><xsl:stylesheet xmlns:xsl=\"http://www.w3.org/1999/XSL/Transform\" xmlns:xs=\"http://www.w3.org/2001/XMLSchema\" exclude-result-prefixes=\"xs\" version=\"1.0\"><xsl:output media-type=\"text/html\" method=\"html\" omit-xml-declaration=\"yes\" encoding=\"UTF-8\"/><xsl:template match=\"lemma\"><span class=\"apparatus-lemma\"><xsl:value-of select=\".\"/></span></xsl:template><xsl:template match=\"location\"><location><xsl:value-of select=\".\"/></location></xsl:template><xsl:template match=\"witnesses\"><span class=\"apparatus-w-value\"><xsl:value-of select=\"value\"/></span><xsl:if test=\"note\"><span class=\"apparatus-w-note\"><xsl:text> </xsl:text><xsl:value-of select=\"note\"/><xsl:text> </xsl:text></span></xsl:if></xsl:template><xsl:template match=\"authors\"><xsl:text> </xsl:text><span class=\"apparatus-a-value\"><xsl:value-of select=\"value\"/></span><xsl:if test=\"note\"><xsl:text> </xsl:text><span class=\"apparatus-a-note\"><xsl:value-of select=\"note\"/></span></xsl:if><xsl:text> </xsl:text></xsl:template><xsl:template match=\"entries\"><xsl:variable name=\"nr\"><xsl:number/></xsl:variable><xsl:if test=\"$nr &gt; 1\"><span class=\"apparatus-sep\">| </span></xsl:if><xsl:if test=\"tag\"><span class=\"apparatus-tag\"><xsl:value-of select=\"tag\"/></span><xsl:text> </xsl:text></xsl:if><xsl:if test=\"subrange\"><span class=\"apparatus-subrange\"><xsl:value-of select=\"subrange\"/></span><xsl:text> </xsl:text></xsl:if><xsl:if test=\"string-length(value) &gt; 0\"><span class=\"apparatus-value\"><xsl:value-of select=\"value\"/></span><xsl:text> </xsl:text></xsl:if><xsl:choose><xsl:when test=\"type = 0\"><xsl:if test=\"string-length(value) = 0\"><span class=\"apparatus-type\">del. </span></xsl:if></xsl:when><xsl:when test=\"type = 1\"><span class=\"apparatus-type\">ante lemma </span></xsl:when><xsl:when test=\"type = 2\"><span class=\"apparatus-type\">post lemma </span></xsl:when></xsl:choose><xsl:if test=\"note\"><span class=\"apparatus-note\"><xsl:value-of select=\"note\"/></span><xsl:text> </xsl:text></xsl:if><xsl:apply-templates/></xsl:template><xsl:template match=\"root\"><xsl:apply-templates/></xsl:template><xsl:template match=\"*\"/></xsl:stylesheet>"
      }
    },
    {
      "Keys": "it.vedph.token-text-layer|fr.it.vedph.comment",
      "Id": "it.vedph.json-renderer.xslt",
      "Options": {
        "WrappedEntryNames": {
          "categories": "category",
          "references": "reference",
          "keywords": "keyword",
          "externalIds": "externalId"
        },
        "Xslt": "<?xml version=\"1.0\" encoding=\"UTF-8\"?><xsl:stylesheet xmlns:xsl=\"http://www.w3.org/1999/XSL/Transform\" xmlns:xs=\"http://www.w3.org/2001/XMLSchema\" exclude-result-prefixes=\"xs\" version=\"1.0\"><xsl:output media-type=\"text/html\" method=\"html\" omit-xml-declaration=\"yes\" encoding=\"UTF-8\"/><xsl:template match=\"*[not(*) and not(normalize-space())]\"></xsl:template><xsl:template name=\"build-link\"><xsl:param name=\"val\"/><xsl:choose><xsl:when test=\"starts-with($val, 'http')\"><xsl:element name=\"a\"><xsl:attribute name=\"href\"><xsl:value-of select=\"$val\"/></xsl:attribute><xsl:attribute name=\"target\">_blank</xsl:attribute><xsl:value-of select=\"$val\"/></xsl:element></xsl:when><xsl:otherwise><xsl:value-of select=\"$val\"/></xsl:otherwise></xsl:choose></xsl:template><xsl:template match=\"reference\"><li><xsl:if test=\"type[normalize-space(.)]\"><span class=\"comment-ref-y\"><xsl:value-of select=\"type\"/></span></xsl:if><xsl:if test=\"tag[normalize-space(.)]\"><span class=\"comment-ref-t\"><xsl:value-of select=\"tag\"/></span></xsl:if><xsl:if test=\"citation\"><span class=\"comment-ref-c\"><xsl:call-template name=\"build-link\"><xsl:with-param name=\"val\" select=\"citation\"></xsl:with-param></xsl:call-template></span></xsl:if><xsl:if test=\"note[normalize-space(.)]\"><xsl:text></xsl:text><span class=\"comment-ref-n\"><xsl:value-of select=\"note\"/></span></xsl:if></li></xsl:template><xsl:template match=\"root\"><div class=\"comment\"><xsl:if test=\"tag[normalize-space(.)]\"><div class=\"comment-tag\"><xsl:value-of select=\"tag\"/></div></xsl:if><xsl:if test=\"categories/category\"><div class=\"pv-flex-row comment-categories\"><xsl:for-each select=\"categories/category\"><div class=\"comment-category\"><xsl:value-of select=\".\"/></div></xsl:for-each></div></xsl:if><xsl:if test=\"text\"><div class=\"comment-text\"><_md><xsl:value-of select=\"text\"/></_md></div></xsl:if><xsl:if test=\"keywords/keyword\"><ul class=\"comment-keywords\"><xsl:for-each select=\"keywords/keyword\"><xsl:sort select=\"indexId\"/><xsl:sort select=\"language\"/><xsl:sort select=\"value\"/><li><xsl:if test=\"indexId[normalize-space(.)]\"><span class=\"comment-kw-x\"><xsl:value-of select=\"indexId\"/></span></xsl:if><span class=\"comment-kw-l\">^^<xsl:value-of select=\"language\"/></span><span class=\"comment-kw-v\"><xsl:value-of select=\"value\"/></span></li></xsl:for-each></ul></xsl:if><xsl:if test=\"references/*\"><div class=\"comment-hdr\">references</div><ol class=\"comment-references\"><xsl:apply-templates select=\"references/reference\"/></ol></xsl:if><xsl:if test=\"externalIds/*\"><div class=\"comment-hdr\">identifiers</div><ul class=\"comment-ids\"><xsl:for-each select=\"externalIds/externalId\"><li><xsl:if test=\"tag[normalize-space(.)]\"><span class=\"comment-id-t\"><xsl:value-of select=\"tag\"/></span></xsl:if><xsl:if test=\"scope[normalize-space(.)]\"><span class=\"comment-id-s\"><xsl:value-of select=\"scope\"/></span></xsl:if><span class=\"comment-id-v\"><xsl:call-template name=\"build-link\"><xsl:with-param name=\"val\" select=\"value\"/></xsl:call-template></span><xsl:if test=\"assertion/*\"><div class=\"comment-assertion\"><xsl:if test=\"assertion/tag\"><span class=\"comment-id-t\"><xsl:value-of select=\"assertion/tag\"/></span></xsl:if><xsl:if test=\"assertion/rank\"><xsl:text></xsl:text><span class=\"comment-id-r\">R<xsl:value-of select=\"assertion/rank\"/></span></xsl:if><xsl:if test=\"assertion/note\"><xsl:text></xsl:text><div class=\"comment-id-n\"><xsl:value-of select=\"assertion/note\"/></div></xsl:if><xsl:if test=\"assertion/references\"><ol class=\"comment-assertion-refs\"><xsl:apply-templates select=\"assertion/references/reference\"/></ol></xsl:if></div></xsl:if></li></xsl:for-each></ul></xsl:if></div></xsl:template><xsl:template match=\"*\"/></xsl:stylesheet>",
        "FilterKeys": [
          "markdown",
          "iso639-3"
        ]
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
.pv-flex-row {
  display: flex;
  gap: 8px;
  align-items: center;
  flex-wrap: wrap;
}
.pv-flex-row * {
  flex: 0 0 auto;
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

/* note */

.note-text {
  margin: 8px;
  column-count: 4;
  column-width: 400px;
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

/* comment layer */

.comment a {
  text-decoration: none;
}
.comment a:hover {
  text-decoration: underline;
}
.comment-tag {
  color: silver;
  font-weight: bold;
  padding: 6px;
  border: 1px solid silver;
  border-radius: 6px;
}
.comment-text {
  margin: 8px;
  column-count: 4;
  column-width: 400px;
}
.comment-categories {
    margin: 6px 0;
}
.comment-category {
    background-color: #afd3ff;
    border: 1px solid #afd3ff;
    border-radius: 4px;
    padding: 4px;
}
.comment-keywords {
  line-height: 200%;
}
.comment-kw-x {
  background-color: #34eb98;
  color: white;
  border-radius: 4px;
  padding: 4px;
  margin: 0 4px;
}
.comment-kw-l {
  background-color: #bdb03e;
  color: white;
  border-radius: 4px;
  padding: 4px;
  margin: 0 4px;
}
.comment-kw-v {
  color: #827609;
}
.comment-hdr {
  color: royalblue;
  border-bottom: 1px solid royalblue;
  margin: 8px 0;
  font-variant: small-caps;
}
.comment-references {
  line-height: 200%;
}
.comment-ref-y {
  background-color: #35c6ea;
  color: white;
  border-radius: 4px;
  padding: 4px;
  margin: 0 4px;
}
.comment-ref-t {
  background-color: #34eb98;
  color: white;
  border-radius: 4px;
  padding: 4px;
  margin: 0 4px;
}
.comment-ref-c {
}
.comment-ref-n {
  font-style: italic;
}
.comment-ids {
  line-height: 200%;
}
.comment-id-t {
  background-color: #34eb98;
  color: white;
  border-radius: 4px;
  padding: 4px;
  margin: 0 4px;
}
.comment-id-r {
  font-weight: bold;
  color: orange;
}
.comment-id-n {
  font-style: italic;
}
.comment-id-s {
  border: 1px solid orange;
  border-radius: 4px;
  padding: 4px;
  margin: 0 4px;
}
.comment-assertion {
  border: 1px solid orange;
  border-radius: 6px;
  padding: 6px;
  margin: 4px;
  background-color: #fefefe;
}
.comment-assertion-refs {
}
```
