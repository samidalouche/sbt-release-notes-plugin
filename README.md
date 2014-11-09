# SBT Release Notes Plugin

Please see [the release notes](RELEASE_NOTES.md) for the latest version of this plugin.

[![Build Status](https://travis-ci.org/urbas/sbt-release-notes-plugin.svg?branch=master)](https://travis-ci.org/urbas/sbt-release-notes-plugin)

This SBT plugin generates release notes. It takes multiple small release note entry files and concatenates them into
a complete release notes file. The small release note entry files can be added by developers for their isolated
features. Such a separation of per-feature release notes is useful in automatic release notes generation when performing
automated releases (e.g.: releases with continuous integration).

## Usage

1. Add this to your `project/plugins.sbt`:

  ```scala
  resolvers += "Sonatype Public Releases" at "https://oss.sonatype.org/content/groups/public/"
  
  addSbtPlugin("si.urbas" % "sbt-release-notes-plugin" % "0.0.1")
  ```

2. Choose your [release notes format](#formats).

3. __Optional__: Choose your [release notes strategies](#strategies).

4. Put the selected format and strategies into your project. Add the following to your `build.sbt` file:

  ```scala
  val root = project.in(".")
    .enablePlugins(<Selected format>, <Selected strategy 1>, <Selected strategy 2>, ... , <Selected strategy N>)
  ```

An example configuration suitable for GitHub repositories:

  ```scala
  val root = project.in(".")
    .enablePlugins(MdReleaseNotesFormat, RootFolderReleaseNotesStrategy)
  ```

Now you can finally run the following SBT commands.

## SBT tasks

`releaseNotes`: Creates the release notes file. The location of the file determined by the [format](#formats)
(by default this file is `target/releasenotes/RELEASE_NOTES`).

`blessReleaseNotes`: Prepares release notes for the next version. You must commit its changes. In detail, this task does
the following:

1. optionally copies the release notes file into the folder determined by the [strategies](#strategies) (default is not
to copy the release notes anywhere).

2. prepends the current release note entries to the `src/releasenotes/releaseNotesBody.previousVersion` file.

3. __DANGER__: deletes all the release notes entry files.

### Formats

__Markdown__:

- Format name: `MdReleaseNotesFormat`
- Release note entries: `src/releasenotes/*.md`
- Default release notes location: `target/releasenotes/RELEASE_NOTES.md`

__RST__:

- Format name: `RstReleaseNotesFormat`
- Release note entries: `src/releasenotes/*.rst`
- Default release notes location: `target/releasenotes/RELEASE_NOTES.rst`

__Write your own__:

Take a look at [the RST](releaseNotesPlugin/src/main/scala/si/urbas/sbt/releasenotes/RstReleaseNotesFormat.scala) or
[Markdown](releaseNotesPlugin/src/main/scala/si/urbas/sbt/releasenotes/MdReleaseNotesFormat.scala) as examples.

### Strategies

#### Root folder

- __Strategy name__: `RootFolderReleaseNotesStrategy`

This strategy is suitable for GitHub-style repositories. See [the github example](samples/github).

Places the blessed release notes file into the project's root folder. For example, if you use the [Markdown](#markdown) format,
then the file `RELEASE_NOTES.md` will be placed in the topmost folder of your project.

#### Sphinx

- __Strategy name__: `SphinxReleaseNotesStrategy`

This strategy is suitable for use with the [sbt-site plugin](https://github.com/sbt/sbt-site) and its Sphinx support).
See [the sphinx example](samples/sphinx).

Does not produce blessed release notes. This strategy outputs the release notes file into `src/sphinx/releaseNotes.rst` (instead
of `target/releasenotes/RELEASE_NOTES.rst`).

You can add this to your `build.sbt` file, if you want the release notes to be generated before Sphinx generates the
documentation:

```scala
import com.typesafe.sbt.site.SphinxSupport._

generate.in(Sphinx) <<= generate.in(Sphinx).dependsOn(releaseNotes)
```

Or you can use `~ ; releaseNotes ; makeSite` command chain when you're updating release notes.
