---
title: "Adding a catalog of Maven Archetypes in Intelli J"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Maven
  - Java
  - IntelliJ
---

Archetypes make it easy to scaffold a project, but remembering all the archetypes available and using the command line to activate it can be challenging to remember. Somehow, it's also not available by default even with the enterprise IntelliJ IDE. But here's the way to make it available. 

## Install the Maven Archetype Catalog plugin. 

From the IntelliJ menu, navigate to `Preferences > Plugins`, under the `Marketplace` tab, search for `Maven Archetype Catalog` and click install.

![Screenshot of Plugin installation](/assets/images/2020/04/Maven-Archetype-Catalog-Plugin.png)

> Make sure to restart the IDE after installing!

## Add the Archetype Catalog in the plugin

After the plugin is installed, a new `Maven Archetype Catalog` will be added under `Preferences > Build, Execution, Deployment > Build Tools`. 

![Screenshot of Maven-Archetype-Catalog-Plugin](/assets/images/2020/04/IntelliJ-Maven-Archetype-Catalog-Settings.png)

Click the `+` button at the bottom to add the Archetype Catalog URL. 

> https://repo.maven.apache.org/maven2/archetype-catalog.xml

**I got this url from the official maven project site - https://maven.apache.org/archetype/maven-archetype-plugin/specification/archetype-catalog.html.**


## Creating a new project with Maven Archetype

From then on, you can then create new projects easily with the huge catalog of maven archetypes available easily.

![Screenshot of new project to use maven archetype](/assets/images/2020/04/IntelliJ-new-project-with-archetypes.png)


