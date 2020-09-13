# Version Maestro v1
Orchestrates versioning new commits given a version id and maintains the CHANGELOG
## Features
* Maintains a CHANGELOG.md file of your commits
  * Formats the changes as "{date} {commit message} | {author name} <{author email}>"
  * The {author email} is a mailto link that will connect to your mail client
  * Segregates these changes by version, with each version as its own section in the log
* Push new annotated tags to your repo with each build
  * Tag new commits in the format "v{your version}"
  * Annotate the tags with the revision type, version, and a newline separated list of changes
# Usage
See [action.yml](https://github.com/juliansangillo/version-maestro/blob/master/action.yml)
## Maintaining a CHANGELOG
Version Maestro will automatically update the CHANGELOG.md file in your repository with the latest changes/commits since the last revision. New versions are always pushed to the top of the log. This process is completely transparent in your CI/CD workflow. Note that if there are no new commits in your git history then nothing is pushed to the log.
### Format of the log
```
# CHANGELOG\n{major revision}  
## {full revision}  
+ **{date}** {commit message} | ***{author name}*** *<[{author email}](mailto:{author email})>*  
...
```
### How it gets your latest changes?
It pulls all of the latest commits by first finding the latest version in your repo and then looks up all the commits between HEAD and the latest version. It can then get the metadata for each commit, format the commit messages, and compile a list that can be appended to the log. It finds your latest version by searching down the git history across all branches until it finds the most recent tag. If this is your first push and there is no latest version yet, it will pull all changes from HEAD down to the initial commit.
### Log Retention
It is good to note that CHANGELOGs don't last forever. A log is only retained for one major revision at most. In other words, if you bump the major revision, then the log will roll over and the first changes of that major revision are pushed to the new log. The old log before the major revision is not saved. If you still wish to see the CHANGELOG at the time of any build, you can still switch to the desired tag in GitHub and that version's log should still be there.
### Example
![Example CHANGELOG](https://github.com/juliansangillo/version-maestro/blob/master/resources/changelog-ex.png)
### Change Types
When pulling the latest changes to put into the CHANGELOG, the commit messages are expected to follow a commit convention. Jira keys removed from the message, so they shouldn't impact anything. However, minus the Jira key, the rest of the message must follow the [Angular Commit Message Conventions](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#-git-commit-guidelines). If a message doesn't follow the convention, it won't be included in the log.
### Supported Types
* feat
* fix
* docs
* style
* refactor
* perf
* test
* chore
* build
* patch
* BREAKING CHANGE
## Tagging new versions
Once the CHANGELOG is updated, Version Maestro will annotate and tag the new version before pushing both to your repository. Tags are always in the format "v{version}".
### Annotations
Below is an example of a tag annotation. Annotations are always in the format {revision type} {version}\n{list of changes}.
### 
```
v1.0.0          MAJOR 1.0.0
    + **2020-09-13** feat: Added changelog-ex.png | ***Julian Sangillo*** *<[juliansangillo@gmail.com](mailto:juliansangillo@gmail.com)>*
    + **2020-09-13** feat: Added action.yml metadata | ***Julian Sangillo*** *<[juliansangillo@gmail.com](mailto:juliansangillo@gmail.com)>*
    + **2020-09-13** feat: Added ASCII title | ***Julian Sangillo*** *<[juliansangillo@gmail.com](mailto:juliansangillo@gmail.com)>*
    + **2020-09-13** feat: Added version-maestro script | ***Julian Sangillo*** *<[juliansangillo@gmail.com](mailto:juliansangillo@gmail.com)>*
    + **2020-09-10** feat: Finished utility scripts | ***Julian Sangillo*** *<[juliansangillo@gmail.com](mailto:juliansangillo@gmail.com)>*
    + **2020-09-10** fix: expansion with spaces as delimeter instead of newlines | ***Julian Sangillo*** *<[juliansangillo@gmail.com](mailto:juliansangillo@gmail.com)>*
    + **2020-09-09** feat: Added initial scripts | ***Julian Sangillo*** *<[juliansangillo@gmail.com](mailto:juliansangillo@gmail.com)>*
```
## Inputs
### version
Version ID to use. Must be a period separated string of numbers and a release tag. This field is required
### revision-pattern
Custom pattern showing version structure. Default is `M.m.b`
### preferred-type
The preferred revision type to fallback to if no other type can be determined. Possible values are 'build' or 'patch'. Default is `build`
## Setup
Below are example workflows using either the defaults or entering custom values. Note the this will push a new commit for the CHANGELOG updates as well as a new tag to your repository.
### Workflow 1
```yml
name: Workflow1
on: push
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Version Composition
        id: composer
        uses: juliansangillo/version-composer@v1

      - name: Version Orchestration
        uses: juliansangillo/version-maestro@v1
        with:
          version: ${{ steps.composer.outputs.version }}
```
### Workflow 2
```yml
name: Workflow2
on: push
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Version Composition
        id: composer
        uses: juliansangillo/version-composer@v1
        with:
          revision-pattern: M.m.b.p
          preferred-type: patch

      - name: Version Orchestration
        uses: juliansangillo/version-maestro@v1
        with:
          version: ${{ steps.composer.outputs.version }}
          revision-pattern: M.m.b.p
          preferred-type: patch
```
## See Also
[juliansangillo/version-composer](https://github.com/juliansangillo/version-composer)
