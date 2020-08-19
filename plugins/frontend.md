# Plugins - Frontend

The frontend web application is based on an Angular multiple-projects repository. In this [architecture](../web/architecture.md), using multiple projects allows for easy extension, as all what you have to do is adding new library projects for your [parts](../web/adding-parts.md)/[fragments](../web/adding-fragments.md) editors and connecting them to the main Cadmus application via a route.

Also notice that these projects are not loaded upfront at startup, but rather they get lazily loaded only when their components are first requested. Thus, having several "plugin" projects does not add any relevant contribution to the weight of the application in terms of resources consumption. Of course, including them in the application package even if you are not going to use them would anyway add to the package size, thus affecting the first-time load of the application.

Currently, our parts/fragments projects are limited, so that from a purely practical standpoint it's more convenient keeping all of them in the Cadmus web application repository. This way, we will just have a single repository for all the projects using Cadmus.

In the future this scenario might change, but at any rate what is relevant here is that each set of new models with their editors is totally self-contained in separate Angular library projects. Thus, the general organization of the web application code already takes into account its modular distribution.
