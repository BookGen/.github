# BookGen

## What Is This?

BookGen is a Makefile‐based system for turning Markdown files into one
  or more different formats which can then be uploaded online, printed,
  distributed, et cetera.
It serves a similar purpose for chaptered media as static site
  generators do for websites.

To reiterate, **BookGen is not an application, program, library, or
  framework.**
It is a series of configurable instructions which tell your computer
  (via [GNU Make][GNU Make]) to use some number of *existing* programs
  and libraries (which also must be installed on your computer) in
  order to generate a desired result.
Consequently, the setup for BookGen is a bit more complicated than many
  other workflows, and using it requires a willingness to live “in the
  operating system” of your computer rather than isolated to one or
  another application or platform.

If you can get on board with those premises, BookGen is a powerful and
  configurable workflow for cross-format book authorship, and one which
  can be easily extended and built upon to meet your precise needs.

## Caveats

Because BookGen is a Makefile‐based system, it suffers the same flaws
  that Makefiles have generally.
It does not play well with filenames which contain spaces or unusual
  ascii punctuation.
Its “A·P·I” consists of the filesystem and a few environment variables.
It cannot operate in directories which do not conform to its
  expectations.

In my experience, working around these limitations is usually an easier
  task than building a software which does not have them.

## Before You Begin

In order to use BookGen, you will need a basic understanding of the
  following concepts :—

+ [The file system](https://marrus-sh.github.io/fcpedia/Entry/file_system/)
+ [Text files & editors](https://marrus-sh.github.io/fcpedia/Entry/text/)
+ [Makefiles](https://marrus-sh.github.io/fcpedia/Entry/Makefile/)
+ [The UNIX command line](https://marrus-sh.github.io/fcpedia/Entry/command_line/)
+ [Yaml](https://marrus-sh.github.io/fcpedia/Entry/YAML/)
+ [Markdown](https://marrus-sh.github.io/fcpedia/Entry/Markdown/)

The following advanced topics may also prove useful, but shouldn’t be
  necessary to get started :—

+ [Git](https://marrus-sh.github.io/fcpedia/Entry/Git/)
+ [The Bourne shell](https://marrus-sh.github.io/fcpedia/Entry/Bourne_shell/)
+ [Python](https://marrus-sh.github.io/fcpedia/Entry/Python/)
+ [Pandoc](https://marrus-sh.github.io/fcpedia/Entry/Pandoc/)
+ [LaTeX](https://marrus-sh.github.io/fcpedia/Entry/LaTeX/)
+ [HTML](https://marrus-sh.github.io/fcpedia/Entry/HTML/)
+ [CSS](https://marrus-sh.github.io/fcpedia/Entry/CSS/)

## Components

A typical BookGen workflow consists of the following components :—

 +  The BookGen source repository, available at
    <https://github.com/BookGen/BookGen>.

 +  One or more BookGen style repositories.

 +  A collection of chaptered Markdown files, comprising the source for
      your book.

 +  A Makefile you wrote (see below) which ties these elements
      together.

## Dependencies

BookGen requires you to have all of the following installed on your
  computer :—

 +  [GNU Make][GNU Make], version 3.81 or later.
    BookGen will *not* work properly with earlier versions of GNU Make,
      or with Make versions which are not GNU.
    You can test which version of Make you have by running `make -v`.

 +  [Pandoc][Pandoc], version 2.8 or later.
    You can test which version of Pandoc you have by running
      `pandoc -v`.

 +  [Python 3][Python], as well as the [Panflute][Panflute] package,
      which can be installed by running `pip3 install panflute`.
    You can test whether Python is installed on your computer by
      running `python3 -V`.

There are additional dependencies depending on what file types you are
  hoping to generate.
For PDF generation :—

 +  A TeX distribution, including LaTeX and XeTeX.
    I use [TeXLive][TeXLive], or more properly [MacTeX][MacTeX].
    Use `xelatex -v` to see if you already have it installed.

 +  The following TeX packages (as well as all their dependencies) :—

    + `background`
    + `biblatex`, including `biber` (for works with bibliographies)
    + `everypage`
    + `hyperref`
    + `ifetex`
    + `logreq`
    + `mfirstuc`
    + `ncctools`
    + `newunicodechar`
    + `memoir`
    + `ulem`
    + `url`
    + `xcolor`

    These will likely come packaged with your TeX distribution unless
      you purposefully installed a limited distribution like BasicTeX.

For PNG generation :—

 +  Everything required for PDF generation, as PNGs are built from
      PDFs.

 +  [GhostScript][GhostScript].
    If you are on macOS, MacTeX offers their own GhostScript package at
      <http://www.tug.org/mactex/morepackages.html>, if you didn’t
      install it as part of MacTeX.
    You can use `gs -v` to see if you already have it installed.

 +  [ImageMagick][ImageMagick].
    Use `magick -version` to see if you already have it installed.

 +  The [Xpdf command line tools][Xpdf], namely `pdftotext`.
    Use `pdftotext -v` to see if you already have it installed.

 +  [Zip][Zip].
    This comes preinstalled on many platforms.
    Use `zip -v` to see if you already have it installed.

It is entirely up to you how you choose to install these dependencies,
  but my recommendation is to follow the official instructions for
  installing your TeX distribution, and to use a package manager (like
  [Homebrew][Homebrew]) for everything else.

## A Basic Setup

After you have downloaded the BookGen source, installed all
  dependencies, and chosen and downloaded a style repository, the next
  step is to create a `Makefile` in your working directory which ties
  everything together.
A simple example follows :—

```Makefile
SHELL = /bin/sh
BOOKGEN := ~/BookGen # replace this with the path to the BookGen source
MYSTYLE := ~/MyStyle # replace this with the path to your chosen style

# Uncomment these lines if you want to use multiple drafts (see below):
# DRAFTS := Drafts
# export DRAFTS

# The first command is what will be run if you just run a simple
#   `make`.
# Change this if you only want to generate certain kinds of files (for
#   example, `html`.)
default: all

# Runs BookGen.
bookgen: ; @$(MAKE) -ef "$(BOOKGEN)/GNUmakefile"

# Runs the `make` script for your chosen style.
mystyle: ; @$(MAKE) -f "$(MYSTYLE)/Makefile"

# Any unhandled targets will be passed through to the BookGen script.
# Explicitly do nothing for `make Makefile`.
Makefile: ;

# Pass through all remaining targets to BookGen.
#
# Note that `mystyle` is a dependency here.
%: mystyle; @$(MAKE) -ef "$(BOOKGEN)/GNUmakefile" $@

# Remove generated files.
clobber distclean gone:
	@$(MAKE) -f "$(ARCHIVESTYLE)/Makefile" gone
	@$(MAKE) -ef "$(BOOKGEN)/GNUmakefile" gone

# Identify phony targets.
.PHONY: default bookgen archivestyle clobber distclean gone
```

Next, create a file titled `info.yml` in your source directory.
This is a [Yaml][Yaml] file which will store all of the
  metadata for your work.
Some values supported by BookGen are :—

```yaml
title: Work title
series: Work series
author: Your name
publisher: Your publisher/website
description: |
  Work summary.
homepage: https://example.com/The-homepage-for-your-work # A URL
year: The copyright year(s) of your work
rights: A short rights statement about your work
draft: 1 # The draft number of the work (added automatically if you are in drafts mode)
lang: en # Work language as an IETF (i·e HTML) langauge tag
final: true # If this is a final, published draft; adds copyright declaration
```

For more on the allowed properties of this file, see [BookGen’s
  `CONFIGURING.md`][CONFIGURING.md] as well as the documentation for
  your chosen style.
You can also set these values on a per‐chapter basis using Yaml
  frontmatter.

## Writing Your Book

Your book can consist of any number of frontmatter chapters, main
  chapters, and appendices.
These must all be placed in a folder titled `Markdown` in your source
  directory, and must have an extension of `.md`.
**Do not use colons, semicolons, or spaces in filenames** as Makefiles
  are not well‐equipped to handle these characters and everything might
  break.

The type of a chapter is determined by its location :—

 +  All files directly in the `Markdown` directory are frontmatter,
      ordered alphabetically.
    You can use leading digits to manually determine their order.

 +  All files in the `Markdown/Chapters` directory with a filename of
      two digits are chapters.
    The chapter number is given by the filename.

 +  All files in the `Markdown/Chapters` directory with a filename of
      `A`, followed by two digits (for example, `A01`) are appendices.
    The appendix number is given by the filename.

Alternatively, if you set `DRAFTS := Drafts` in your Makefile above,
  you may instead give your chapters as *folders* of Markdown files in
  the `Drafts` directory, and the alphabetically last file in each will
  be automatically linked into the `Markdown` directory for you.
(For example, the file at `Drafts/Chapter/01/Draft_02.md` will be
  linked from `Markdown/Chapter/01.md`.)
This allows you to store multiple drafts of a chapter alongside one
  another, with the last draft used as the final result.

If you are using drafts mode, it is best not to ever touch the
  `Markdown` folder directly, and work entirely in `Drafts` (even for
  chapters with only one draft).

## Markdown Syntax

BookGen uses Pandoc under the hood, so the Markdown syntax is [Pandoc
  Markdown][Pandoc Markdown].

There are a few added features you can take advantage of in your
  Markdown for special formatting and display :—

 +  A Div with a class of `chapterprecis` can be used at the beginning
    of a chapter to insert a chapter precis :—

    ```markdown
    # My Chapter

    ::: chapterprecis :::
    | A very good chapter,
    | Yes, very good indeed.
    :::::::::::::::::::::
    ```

 +  A Div with a class of `verse` can be used for verse.

    If you also set the class `alternating`, then every other line will
      be indented :—

    ```markdown
    ::: {.verse .alternating}
    | A couplet writ in very little time,
    | With indentation on this second line.
    :::
    ```

    Alternatively, you can use an empty Span with a class of `indent`
      to manually indent verse lines :—

    ```markdown
    ::: verse :::
    | There once was a limerick, quite good,
    | And people assumed that it would
    | []{.indent}End with some joke
    | []{.indent}About some poor bloke,
    | But I don't see whyfor it should.
    :::::::::::::
    ```

 +  A Div with a `role` of `note` can be used for notes :—

    ```markdown
    ::: {role=note}
    This is a note.
    :::
    ```

 +  A Div with a class of `continuation` can be used to create a
      paragraph which continues from the previous, useful if a
      blockquote or line of verse comes between them :—

    ```markdown
    To quote Light Yagami from <cite>Death Note</cite>,

    > This useless Pride, I suppose I'll have to… Get Rid of It!

    ::: continuation :::
    (as translated by the English dub).
    ::::::::::::::::::::
    ```

 +  A Div with a class of `plain`, which contains only one paragraph,
      can be used to "unwrap" the paragraph (so that no `<p>` tag is
      used) :—

    ```markdown
    ::: plain :::
    This will not use a `<p>` tag.
    :::::::::::::
    ```

 +  A Div or Span with `data-from-metadata` set will have its contents
      replaced by the corresponding metadata value, if set.
    This is especially useful for localization :—

    ```markdown
    See [Chapter]{data-from-metadata="localization-type-chapter"} 02.
    ```

 +  A Span with a class of `lettrine` can be used for leading text.
    An initial span will produce a drop cap :—

    ```markdown
    [[T]{}his is the beginning]{.lettrine} of a section of text.
    ```

 +  A Span with a `data-colour` (or `data-color`) attribute can be used
      to set the text colour.
    This can be either a 6-digit HTML hex value or an SVG colour name.
    In the latter case, the name must be properly capitalized :—

    ```markdown
    Some [red]{data-colour=#FF0000} and
      [blue]{data-colour=MidnightBlue} text.
    ```

 +  A Span with a `data-font` attribute may be used to manually set the
      text font.
    Which values are supported depends on your current style.

    ```markdown
    This is [fantastic]{data-font=fantasyfont}.
    ```

 +  An empty Span with a class of `at` can be used to generate a `\@`
      for sentence-spacing adjustment in LaTeX.
    This is only necessary if you are generating PDFs with a style
      which does not use `\frenchspacing`:

    ```markdown
    Reading Rainbow, Mr.[]{.at} Rogers, etc.[]{.at} are all fond
      memories for I[]{.at}.
    ```

 +  A raw HTML block of the form `<hr class="plain"/>` represents a
      plain (unfancy) break.

Although these are standard BookGen features, full support may depend
  on your selected style.
Furthermore, your style may have additional features besides these.

## Further Reading

For a comprehensive account of all of the features and customization
  options of BookGen, see [the Bookgen README][README.md].

[CONFIGURING.md]: <https://github.com/BookGen/BookGen/blob/current/CONFIGURING.md>
[GhostScript]: <https://ghostscript.com/>
[GNU Make]: <https://www.gnu.org/software/make/>
[Homebrew]: <https://brew.sh/>
[ImageMagick]: <https://imagemagick.org/>
[MacTeX]: <https://www.tug.org/mactex/>
[Pandoc]: <https://pandoc.org/>
[Pandoc Markdown]: <https://pandoc.org/MANUAL.html#pandocs-markdown>
[Panflute]: <http://scorreia.com/software/panflute/>
[Python]: <https://www.python.org/>
[README.md]: <https://github.com/BookGen/BookGen/blob/current/README.md>
[TeXLive]: <https://www.tug.org/texlive/>
[Xpdf]: <https://www.xpdfreader.com/>
[Yaml]: <https://yaml.org/>
[Zip]: <http://infozip.sourceforge.net/Zip.html>
