git-hooks-code-autoformat
=========================

A collection of git `pre-commit` hooks to help with code auto-formatting.

Licensed under [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0.html).

Please, share your own formatters back. PRs are welcome!

How to use
----------
You could just copy contents of this repo to `$PROJECT/.git/hooks/`. However, a slightly better way might be:

1. `$ cd $PROJECT`
1. `$ git remote add git-hooks-code-autoformat https://github.com/michalrus/git-hooks-code-autoformat.git`
1. `$ git subtree add --prefix=git-hooks/ git-hooks-code-autoformat master`

Now, you have `$PROJECT/git-hooks/` directory with contents of this repo. This subtree is updateable with `$ git subtree pull --prefix=git-hooks/ git-hooks-code-autoformat master`. Any person can just clone your project and they'll have `$PROJECT/git-hooks` in place at the very moment.

What's left is to symlink `$PROJECT/.git/hooks` to `$PROJECT/git-hooks`:

1. `$ cd .git && mv hooks hooks.old && ln -s ../git-hooks hooks`

**IMPORTANT**: this linking step **has to** be done **manually** by **every developer**.

**IMPORTANT**: for **Windows** users, due to limitations on Windows shell emulators, links seem to be implemented more often than not as 'copy', so any time the git-hooks directory is updated, the linking step needs to be re-done.

The usual workflow then is:

1. Modify some files.
1. `$ git add` them.
1. `$ git commit` them.

Now, for each staged ("git-added") file, an autoformatter will be called, and then the file will be re-added, and only then the `git commit` will continue. This might create a problem, see [Known problems](#known-problems) below.

If an error occurs (e.g. trying to format a not so well-formed XML), an error message is printed and the commit is aborted.

[Existing formatters](/autoformat)
-------------------

* [Java](/autoformat/java) ← will format [`*.java`](/autoformat/java.patterns) files using a small wrapper—[eclipse-code-formatter](/tools/eclipse-java-formatter)—around [Eclipse’s `CodeFormatter`](http://help.eclipse.org/luna/topic/org.eclipse.jdt.doc.isv/reference/api/org/eclipse/jdt/core/formatter/CodeFormatter.html) with [Google’s Java Style definition](https://github.com/google/styleguide/blob/gh-pages/eclipse-java-google-style.xml),
* [XML](/autoformat/xml) ← will format [`*.xml`, `*.bmml` and `*.owl`](/autoformat/xml.patterns) (Balsamiq Mockups, Protégé) files and any [file that is recognized as XML](/autoformat/xml.magic) by [libmagic](http://en.wikipedia.org/wiki/File_%28command%29),
* [XMind](/autoformat/xmind) ← will format [`*.xmind/**.xml`](/autoformat/xmind.patterns) files (an `*.xmind` file is really a ZIP archive and **should be extracted**—and autoformatted—before being added to VCS; you can then open a mindmap using `$ XMind some/directory.xmind` and it will then be added to the recently opened maps list),
* [Flat OpenDocument Spreadsheet](/autoformat/flat-ods) ← will format [`*.fods`](/autoformat/flat-ods.patterns) files, a format used by LibreOffice Calc for storing spreadsheets.

Adding your own formatters
--------------------------

1. Take a look at the existing formatters above.
2. Create `autoformat/NAME.patterns` file containing a regular expressions—1 per line—matching all file names to be treated with your `NAME` formatter.
2. AND/OR create `autoformat/NAME.magic` file containing a regular expressions—1 per line—matching outputs of [libmagic](http://en.wikipedia.org/wiki/File_%28command%29)’s `/usr/bin/file "$FILE"` for all files to be treated with your `NAME` formatter.
3. Create `autoformat/NAME` executable script. It might be called several times, but always with one argument, an absolute path to a file to format. When formatting, you've got to **modify this file in place**.
4. If you want to abort the commit, print an error message and exit from the formatter with a non-zero exit code.

Known problems
--------------

1. The [`pre-commit`](/pre-commit) hook autoformats every staged ("git-added") file (if an applicable formatter exists) and then **re-adds the file to the stage**. Now, if you modify some file, `$ git add` it, then modify it again and then `$ git commit`—and there exists a formatter for this type of a file—second modification will also be commited.
