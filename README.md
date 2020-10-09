# x9incexc-bash

Bash script-based (but still advanced) program to generate a list of files based on advanced pattern matching. This is the most complete and almost-done part of [a larger effort](https://github.com/x9-testlab/x9incexc_language-selection) to select the optimal language to rewrite this in binary form.

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
- [ ] [Reimpliment more robustly in Go](https://github.com/x9-testlab/x9incexc-go), once the basic syntax and logic are nailed down. (Arguably already there, but this script is so close to being useful _now_!)
