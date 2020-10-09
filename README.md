# x9incexc-bash

A Bash script-based and reasonably simple program that generates a list of files based on advanced pattern matching.

The resulting list of files can be consumed by other programs, e.g. `rsync`, via some variationof `--files-from` argument.

This is the most complete and almost-done part of [a larger effort](https://github.com/x9-testlab/x9incexc_language-selection) to select the optimal language to rewrite this in binary form.

## Feautes

- Scan one or more specified directories.
- Filter results based on regex (and/or regex macros).
- See separate lists of "included" files, and "excluded" files! (This is huge, and helps debug problems before consuming programs have to deal with it.)
    - For example, for consumption by backup programs, this helps you identify exactly what will not be backed up.
    - Optionally, or additionally, see the same info in "diff" format.
- Unlike many glob-based "include/exclude" features of other programs (e.g. `rsync`, `rclone`), includes and excludes are processed linearly, top-to-bottom. (Not backwards, right-to-left or bottom-to-top.)
    - Includes and excludes don't depend on each other. There is no concept of "nesting", which greatly complicates both programming logic, and the human brainpower to rationalize what's going on. (I.e. you don't have to hold in mind a complex "mental stack" of what is in and what's out, as you're struggling to write the rules.)
    - Anything included, can be later excluded.
    - Anything excluded, can be later included.
    - And so on, infinitely. This makes it "infinitely" easier to think through and hold in the mind all at once, as there are no dependencies other than the top-down order of processing.


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
