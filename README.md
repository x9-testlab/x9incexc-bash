# x9incexc-bash

A Bash script-based and reasonably simple program that generates a list of files based on advanced pattern matching.

The resulting list of files can be consumed by other programs, e.g. `rsync`, via some variationof `--files-from` argument.

This is the most complete and almost-done part of [a larger effort](https://github.com/x9-testlab/x9incexc_language-selection) to select the optimal language to rewrite this in binary form.

## Feautes

- Scans one or more specified directories, to generate a combined, sorted file list.
- Filter those results based on a series of regexes (and/or regex macros).
- View separate lists of "included" files, and "excluded" files! (This is huge, and helps debug problems before consuming programs have to deal with it.)
    - For example: for consumption by a backup program, this helps you identify exactly what will and won't not be backed up.
    - Optionally, or additionally, see the same info in "diff" format.
- Unlike many glob-based "include/exclude" features of other programs (e.g. `rsync`, `rclone`), includes and excludes are processed linearly, top-to-bottom. (Not backwards, right-to-left or bottom-to-top.)
    - Includes and excludes don't depend on each other. There is no concept of "nesting", which greatly complicates both programming logic, and the human brainpower to rationalize what's going on. (I.e. you don't have to hold in mind a complex "mental stack" of what is in and what's out, as you're struggling to write the rules.)
    - Anything included, can be later excluded.
    - Anything excluded, can be later included.

## How it works

It makes it easy to reason about the input and output, if you think of it this way:

- The directory-scanning process produces a main list of files. This list is considered the _immutible source list_.
- The include/exclude process generates a _second_ list of filtered files - what you're going to consume - intended to be (but not necessarily) smaller than the first. (But never larger.)
- The filtered list actually starts out as an exact copy of the immutible source list.
- Any exclude rule, _always removes matches from the filtered list_.
- Any include rule, _always adds back in, matches from the immutible source list_.
- At any point in the list of include/exclude rules, you could (for some reason) remove _everything_ from the filtered list, by specifying `.*` as an exclude rule. No one is going to judge you (except your peers who might have to maintain it).
- Axiomatically, at any point you could override all your laborious reductions, by specifying `.*` as an _include_ rule. (I mean, at this point it would just be obvious that your list of rules is capricious and punishing.)

That's it, it's that simple. That's why a simple Bash script can do it. And as you might have figured out:

- Anything included at any point, can be later excluded.
- Anything excluded at any point, can be later included.
    - And so on, infinitely. This makes it easier to think through and hold in the mind all at once, as there are no dependencies and nesting rules - just a simple top-down order of processing.
- It's fairly trivial to exclude a broad-scoped pattern, only to add a more nuanced subset back in later.
- It's fairly trivial to add or remove from an already nuanced set, a _different_ pattern that cuts across the existing results set a different way.
- It would be easy to repeatedly remove and add back in the same file, folder, or patterns, repeatedly. (Indefinitely, really.)
    - That's OK, the cost in performance is reasonably low (barring obviously absurd extremes), but doing so is also a fairly obvious mistake to notice and avoid.
    - Adding the same files back in over and over, doesn't result in them being listed multiple times in the final output. No matter what, each unique filename exists in the final output only once.

## Logical example

- Start with a list of all normal files in these folders:
    - `/home`
    - `/var`
- From that resulting _immutible source list_ of files, and initial identical copy as _filtered list_:
    - Remove any lines with one or more match of `.*\/\.ecryptfs($|[^\w].*)` from _filtered list_.
    - Remove any lines with one or more match of `.*\/Downloads` from _filtered list_.
    - Remove any lines with one or more match of `.*\/\.cache` from _filtered list_.
    - Add back to _filtered list_, anything from _immutible source list_ matching `.*\.(jpe?g|dng|arw)$`.
    - Remove from _filtered list_ (even from photos just added back in), `(^|.*[^\w])cancun($|[^\w].*)`.
    - Add back to _filtered list_, just in case, anything from _immutible source list_ matching `.*\.(xls|ppt|doc)x$`.
    - But really, we don't want _anything at all_ in trash, so finally remove from _filtered list_, `.*\trash($|[^\w].*)`.

With simplifying regex macros, the previous include/exclude list could look like this:


## To do

- [ ] Finish basic logic. (Almost there!)
- [ ] Move source dir array, and regex definitions, to a separate file, for modularity and multiple definitions.
    - At it's simplest, a simple procedurally-defined wrapper script that:
        - Follows a predefined "interface" - that defines an expected set of variables, at minimum:
            - Source dir array
            - Include and exclude regexes
            - Output files
        - Then `source`s this script, which consumes those variables, and errors if they don't exist or are invalid.
    - Alternatively (and more standard but very slightly harder), this could be a simpler .cfg file that provides the same data, and is specified as an argument when directly invoking this script.
- [ ] Implement regex macros to take some pain out of what is inevitably high redundancy (and chance for errors) in this kind of regex-based filtering.
- [ ] Allow alternate filtering syntax, e.g. * and ** globbing; probably by treating them as macros and converting to regex.
    - Would also need a prefix to tell the preprocessor that "this is a glob specification to convert to regex".
- [ ] Allow ability to specify different output formatters. E.g. `plain` (for most programs e.g. `rsync --files-from`), `csv` (why?), `restic --files-from`, `borg --patterns-from`, etc.
- [ ] Allow date and filesize formatting. This is easy to do with `find`. Drawbacks and limitations (which will be solved when based on a database):
    - Generating an "excluded" file and/or diff file for fine-tuning and troubleshooting, becomes more complex. (It would require two runs of `find` behind the scenes: One with specified size and date filters, one without.)
    - Unlike regexes, `find`-based size and date arguments would be exclude-only, with no possibility of being infinitely stackable include/exclude. (Until the underlying engine is a database rather than `find`.)
- [ ] [Reimpliment more robustly in Go](https://github.com/x9-testlab/x9incexc-go), once the basic syntax and logic are nailed down. (Arguably already there, but this script is so close to being useful _now_!)
- [ ] Scan directly (rather than via `find`), and store results in an intermediate database, along with attributes such as mtime and filesize.
    - This would make it significantly easier to fine-tune include/exclude rules, for users who have the knowledge and/or GUI to browse a sqlite3 database.
        - But file-based outputs would still be an option, if not the default.
    - Cross-platform is then possible.
    - Bash has unacceptably poor performance for highly-iterative and/or recursive operations such as scanning a large hierarchical filesystem.
    - Bash isn't ideal for creating and managing sqlite3 databases. (It's actually incredibly easy via the sqlite3 CLI; just very slow. But the result - a browsable database - is the same.)
    - Implies a two-step process. So maybe generating an intermediate, browsable database would need to be an optional step to aid in fine-tuning and troubleshooting.
