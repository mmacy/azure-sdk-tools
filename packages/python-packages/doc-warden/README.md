# Doc Warden

Every CI build owned by the Azure-SDK team also needs to verify that the documentation within the target repo meets a set of standards. `Doc-warden` is intended to ease the _implementation_ of these checks in CI builds.

Features:

* Enforces Readme Standards
    - [x] Readmes present 
    - [ ] Readmes have appropriate contents
    - [ ] Files issues for failed standards checks
        - [ ] Exit code > 0 for issues discovered
* Generates report for included observed packages

This package is under development, and as such Python version compatibility has not been finalized at this time.

## Prerequisites
This package is intended to be run as part of a pipeline within Azure DevOps. As such, [Python](https://www.python.org/downloads/) must be installed prior to attempting to install or use `Doc-Warden.` While `pip` comes pre-installed on most modern Python installs, if `pip` is an unrecognized command when attempting to install `warden`, run the following command **after** your Python installation is complete.

In addition, `warden` is distributed using `setuptools` and `wheel`, so those packages should also be present prior to install. 

```
/:> python -m ensurepip
/:> pip install setuptools wheel
```

## Usage

Right now, `warden` has a single command.  `scan`, which by default looks for a target `.docsettings.yml` file within the target repo. However, all the parameters that can be pulled from the `.docsettings` files will **override** whatever is placed within the `.docsettings` file.

Example usage:

```

<pre-step, clone target repository>
...
/:> pip install setuptools wheel
/:> sudo pip install doc-warden
/:> ward scan -d $(Build.SourcesDirectory)

```
**Notes for example above**

* Devops is a bit finicky with registering a console entry point, hence the `sudo` just on the installation. `sudo` is only required on devops machines.
* Assumption is that the `.docsettings` file is placed at the root of the repository.
    * To provide a different path (like `azure-sdk-for-java` does...), use: 
        * `ward scan -d $(Build.SourcesDirectory) -c $(Build.SourcesDirectory)/eng/.docsettings.yml`

##### Parameter Options

`command` 
Currently supports the `scan` command. Additional commands may be supported in the future. **Required.**

`--scan-directory`
The target directory `warden` should be scanning. **Required.**

`--scan-language`
`warden` checks for packages by _convention_, so it needs to understand what language it is looking at. This must be populated either in `.docsettings file` or by parameter. **Required.**

`--config-location`
By default, `warden` looks for the `.docsettings` file in the root of the repository. However, populating this location will override this behavior and instead pull the file from the location in this parameter. **Optional.**

`--verbose-output`
Enable or disable output of an html report. Defaults to false. **Optional.**

##### Notes for Devops Usage

The `-d` argument should be `$(Build.SourcesDirectory)`. This will point `warden` at the repo that has been associated with CI.

## Methodology

### Enforcing Readme Presence 

When should we expect a readme to be present?

**Always:**

* At the root of the repo
* Associated with a `package` directory

#### .Net

A package is indicated by:
* a `*.csproj` file
    * Project file does not end with `tests.csproj`

#### Python

A package is indicated by: 

* the presence of a `setup.py` file

#### Java

A package is indicated by:

* the presence of a `pom.xml` file
    * The POM `<packaging>` value within is set to `JAR`

#### Node & JS

A package is indicated by: 

* The presence of a `package.json` file

#### Control, the `.docsettings.yml` File, and You

Special cases often need to be configured. It seems logical that there needs be a central location (per repo) to override conventional settings. To that end, a new `.docsettings.yml` file will be added to each repo. 

```
<repo-root>
│   README.md
│   .docsettings.yml    
│
└───.azure-pipelines
│   │   <build def>
│   
└───<other files and folders>
```

The presence of this file allows each repository to customize how enforcement takes place within their repo.

**Example DocSettings File for Java Repo**

```
omitted_paths:
  - archive/*
language: java
root_check_enabled: True
```

The above configuration tells `warden`...

- The language within the repo is `java`
- To ensure that a `README.md` is present at the root of the repository.
- To omit any paths under `archive/` from the readme checks.

Possible values for `language` right now are `['net', 'java', 'js', 'python']`. Greater than one target language is not currently supported.

## Provide Feedback

If you encounter any bugs or have suggestions, please file an issue [here](<https://github.com/Azure/azure-sdk/issues>) and assign to `scbedd`.