---
title: "2. Coding environment"
subsection: "tutorial"    
tags:
    - tutorial
menu:
    learn-tutorial:
        parent: "getting-started"
        weight: 2
---

SeedStack doesn't require you to use a specific IDE, or even use one. But it is sometimes more productive to use a
full-featured IDE for developing Java applications. All generated projects are standard Maven projects, so you can import 
them in your IDE of choice.<!--more--> 

## IntelliJ

### Import project

To import a SeedStack project, follow these steps:

1. In the `File` menu, click `Open...`
2. Navigate to the project directory and directly open the pom.xml file
3. In the popup, click the `Open as project` button.

### Run/debug project

To run a SeedStack project, follow these steps:

1. Add a `Run configuration` by using the toolbar dropdown
2. In the dialog box, click the `+` button and choose `Application`
3. Name your run configuration as you wish
4. Specify `org.seedstack.seed.core.SeedMain` as main class.
5. Close the dialog with `OK` then click the run button (green play button) in the toolbar.

![IntelliJ run configuration dialog](../img/intellij-run.png)

{{< callout tips >}}
You can debug the project by clicking on the toolbar bug button instead of the run button.
{{< /callout >}}

### Plugin

A SeedStack plugin is available for IntelliJ. It provides the following features: 

* Support for SeedStack YAML configuration language
    * Completion,
    * Java and macro references,
    * Rename and safe-delete refactoring,
    * Quick documentation of options,
    * Inspections,
    * Find usages,
    * Jump to configuration classes.
* SeedStack Navigator
    * Business domain structure,
    * Configuration files,
    * REST resources,
    * Tools.
    
To get the latest version of the plugin:

1. Go to the [release tab](https://github.com/seedstack/intellij-plugin/releases) of the [GitHub repository](https://github.com/seedstack/intellij-plugin) 
2. Download the `seedstack-intellij-plugin-x.y.z` ZIP file on your computer.

To install it:
 
1. Start IntelliJ IDEA, 
2. Go to "Settings" (or "Preferences" on MacOS), choose "Plugins" then "Install from disk...",
3. Select the ZIP file you just downloaded and restart IntelliJ when asked. 

After restarting, you should be able to use the plugin features in your IDE. 

## Eclipse

### Import project

To import a SeedStack project, follow these steps:

1. In the `File` menu, click `Import...`
2. From the `Maven` section, choose `Existing Maven projects...`
3. In the dialog box, click `Browse...` and choose the project directory 
4. Click `Finish` 

### Run/debug project

To run a SeedStack project, follow these steps:

1. In the `Run` menu, choose `Run configurations...`
2. In the dialog box, select `Java application` in the left list, then click new button
3. Name your run configuration as you wish
4. In the `Project` field, specify the project you just imported
5. In the `Main class` field, specify `org.seedstack.seed.core.SeedMain`.
6. Close the dialog, with the `Run` button.

![IntelliJ run configuration dialog](../img/eclipse-run.png)

{{< callout tips >}}
You can debug the project by clicking on the toolbar bug button instead of the run button.
{{< /callout >}}