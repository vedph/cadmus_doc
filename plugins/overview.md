# Plugins - Overview

Given the web-targeted nature of Cadmus and its Docker-oriented distribution, the strategy for extending the editor with plugins is based on a design-time approach.

The Cadmus architecture is open and modular, but not in the sense that you are discovering plugins at runtime. Such a scenario would not fit well a robust web layer, ready to be packed in a Docker image and deployed in a cloud-like environment.

Thus, we are not going to fire the same Cadmus Docker image for all the projects based on it, and then bind "plugins" at runtime, which would adversely affect a binaries-based distribution; rather, we are going to provide a full Docker stack for each single project, with all its extensions in place. This favors a robust and versionable deployment, where each publication of every project can be based on a well-defined Docker stack, self-contained and including all its dependencies.

Currently, this implies that you have to do some very limited manual work in order to create your custom-extended version of the system. In the future, it will be possible to provide automated procedures for all the steps illustrated in this section; but currently this is not the top priority. As I'm still in the early stage of applying Cadmus to several different projects, I'm learning from the patterns emerging in recurring procedures, which will help to provide automation at a later, more mature stage.

At any rate, what should be emphasized here is that the whole architecture of the system has been designed with expansion in mind, and this explains why the procedure illustrated here should appear very mechanic and short (which provides the basis for a future automation of such a procedure).

Extending the system must be done both in the [backend](backend.md) and in the [frontend](frontend.md).
