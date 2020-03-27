---
title: "LaTeX Tooling Guide"
layout: post
---

I'll let you in on a small secret: I hate LaTeX with a passion. It's a bloated
mess with crazy syntax. It generates reams of warnings you can never totally get
rid of, and some of the most confusing and/or unhelpeful errors I have ever
seen. It needs to be run N times in a row. It even has a stupid name.

Don Knuth wrote TeX, and it may have been art. But, to use a fashionable word,
TeX was never meant to scale to LaTeX. Pervasive use of non-hygienic macros and
constant manipulations of global values is a recipe for disaster.

I wish I could avoid LaTeX completely, unfortuately conferences have templates
and the only other format available is MS Word, which has its own flaws (as I
recall, in how it handles numbering and floating figures).

Anyhow, the current article is about how I setup my Latex to alleviate some of
these pains. I'll start with a tour of what I use, then give you [a makefile]
and an [example git repository][a repository] to bring all these tools together.

---

## Installing and Updating

See [this article][latex-compilers] about available Latex distributions. I
recommend sticking to the tried-and-true, so that would be MiKTeX for Windows,
MacTeX for OSX and TeX Live for Linux.

[latex-compilers]: https://www.sharelatex.com/learn/Choosing_a_LaTeX_Compiler

If you already have a Latex distribution installed, but it is not up to date,
this fantastic [TeX Stack Exchange answer][latex-update] will tell you how to
update it.

[latex-update]: https://tex.stackexchange.com/questions/55437/how-do-i-update-my-tex-distribution

## Editors

Most people will use some kind of Latex-mode that comes with their editor of
choice. I edit text in Emacs, and so I use [Auctex].

Nevertheless, it might be nicer to have something closer to an IDE for Latex. My
option of choice here is [TeXstudio]. Even though most of my Latex editing is in
Auctex, I use it from time to time to typeset math or help with diagnostics.

[Auctex]: https://www.gnu.org/software/auctex/
[TeXstudio]: http://www.texstudio.org/

Finally, but perhaps most importantly, there are a few online Latex editors. The
best of them is [ShareLaTeX]. Its great strength is that it saves you from doing
any kind of setup, and in my own experience is super reliable.

[ShareLaTeX]: https://www.sharelatex.com/

ShareLatex also filters out unhelpful warning and errors and does its best to
correlated what is left with your source file. Later in this guide we will show
command line tools that do the same thing.

It's good enough as an editor, but also allows realtime collaboration on a
document. Best of all, it now has the ability to sync with GitHub, meaning you
can mix online and offline collaboration.

## Building

One of the famous annoyances of Latex is that it needs to be run multiple times
to get references right, interleaved with runs of your bibliography tool
(usually BibTeX or Biber).

Fortunately this whole process can be automated. Most Latex distribution come
with a tool called `latexmk`, which does automates all these invocations for
you.

Beware that `latexmk` requires Perl to run, which is not bundled with Latex.
This tends to be a caveat on Windows. If you are running Windows and want to use
`latexmk`, install [Strawberry Perl] to fix the problem.

[Strawberry Perl]: http://strawberryperl.com/

There is a better tool, though, which is called [latexrun]. `latexrun` does what
`latexmk` does, but also sanitizes Latex's output to only show helpful errors,
similar to what you would see on ShareLatex.

[latexrun]: https://github.com/aclements/latexrun

`latexrun` is a single Python3 script, which is great for portability. You can
even drop it directly in your paper's git repository.

## Errors

If you're using ShareLatex or `latexrun`, you're already pretty well covered
here.

There is one more tool that bears mentioning: [pulp]. `pulp` lets you filter out
unwanted warnings and errors, and tries to correlated errors with file
positions.

[pulp]: https://github.com/dmwit/pulp

`pulp`'s output is less pretty than that of `latexrun`, but its great strength
is that it lets define custom filtering rules, to remove those annoying warnings
that you cannot get rid of, but break nothing. Or, more frequently, comes from
the class file that you are forced to use to submit you paper (I'm looking at
you, ACM).

Here's an exemple of filtering spec to get rid of all errors that pop up when
compiling the [acmart] example file:

```
!(boring | info | message | under | over
| p "hyperref" & "Token not allowed in a PDF string"
| "Class acmart Info"
| "file:line:error style messages enabled"
| "Excluding.*comment"
| "Processing.*comment"
| "Include comment"
| "comment.cut"
| "(msharpe)"
| "]" && !".."
)
```

The first line matches `pulp`'s default specification.

[acmart]: http://www.sigplan.org/Resources/Author/

`pulp` can be integrated with `latexmk` fairly easily, by means of a
configuration file.

`pulp` is not as friendly to install as the other tools. Here is how to install
on OSX, assuming you already have [homebrew] (on other platforms, you just need
to install the Haskell platform in some other way):

[homebrew]: https://brew.sh/

```
brew cask install haskell-platform
git clone https://github.com/dmwit/pulp.git
cabal update
cabal install
PATH="$HOME/Library/Haskell/bin:$PATH"
```

## Preview

Unless you're working with TeXstudio or ShareLatex, you'll need to preview the
generated PDF files using a PDF viewer. You want to use one that picks up
changes to the PDF on the disk, and reloads it on the fly, while keeping your
position in the PDF. On OSX, the built-in viewer (confusingly named *Preview*)
does this adequately. On Windows, I'm partial to [Sumatra PDF].

`latexmk` and `latexrun` will automatically open a generated PDF file (using the
OS-determined associated tool).

[Sumatra PDF]: https://www.sumatrapdfreader.org/free-pdf-reader.html

## All Together

I've put together [a repository] that serves as template for papers using
the [ascmart sigplan][acmart] style.

[a repository]: https://github.com/norswap/template-sigplan

The tooling centers around [a makefile]. The default configuration is to use
`latexmk` with pulp integration. The following commands are available:

- `make` / `make view`: generates the PDF file and displays it
- `make pdf`: generates the PDF file but does not display it; latex is run silently
- `make verbose`: like `make pdf`, but shows error details
- `make rebuild`: forces a rebuild of the pdf, even if no changes are picked up
- `make clean`: removes all temporary files (for all tools!)
- `make mrproper`: like `make clean`, but also removes the generated PDF

[a makefile]: https://github.com/norswap/template-sigplan/blob/master/makefile

If you don't have `pulp`, just delete or rename the `.latexmkrc` file from the
repo.

If `latexmk` is not found or doesn't work (for instance if Perl is not
installed), the makefile will fall back to using plain make rules, and `make
pdf` will be verbose by default.

It is also possible to use `latexrun`, which is bundled in the repository, by
running `make run` (Python3 required).

`pulp` can be run standalone with `make pulp` (pulp is not bundled in the
repository), assuming a compilation was attempted earlier and the `.log` file is
left uncleaned.

**DISCLAIMER**: I won't be too assiduous in maintaining this makefile, and it
may even contain a few bugs. It should be taken as an example of how to do
things, rather than as a toolchain to be relied on.

With all this, you'll have no Latex excuses for turning your next paper late :)
