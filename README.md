Bash Boilerplate
================

A collection of example bash scripts that can be used as starting points.

I also use these script to record and document various common approaches and
conventions that I've learning and encountered while working with bash. To this
end, each script contains a lot of comments attempting to describe the
functionality and syntax as much as possible.

In many cases there are debug statements or example code that demonstrate
functionality or reveal the state of the program at a certain point. These
should generally be removed when customizing the scripts, while retaining the
parts that still apply. Especially in the case of the `bash-with-commands`
script, it's probably a good idea to play around with the different features
before diving into customization.

## Scripts

### [bash-simple](https://github.com/alphabetum/bash-boilerplate/blob/master/bash-simple)

A simple bash script with some basic strictness checks and help features.

### [bash-simple-plus](https://github.com/alphabetum/bash-boilerplate/blob/master/bash-simple-plus)

A simple bash script with some basic strictness checks, option parsing,
help features, easy debug printing.

### [bash-with-commands](https://github.com/alphabetum/bash-boilerplate/blob/master/bash-with-commands)

An example of a bash program with commands. This contains lots of features.

## Notes

Most of these tips are included in the boilerplate scripts, but I'm also
adding them here for easy reference.

### ShellCheck

Use it. It's super useful.

> ShellCheck is a static analysis and linting tool for sh/bash scripts.
It's mainly focused on handling typical beginner and intermediate level
syntax errors and pitfalls where the shell just gives a cryptic error
message or strange behavior, but it also reports on a few more advanced
issues where corner cases can cause delayed failures.

#### Links

- http://www.shellcheck.net/
- http://www.shellcheck.net/about.html
- https://github.com/koalaman/shellcheck

It can be used with vim via
[syntastic](https://github.com/scrooloose/syntastic)

---

### Bash "Strict Mode"

These boilerplate scripts use a some common settings for enforcing strictness
in Bash scripts, thereby preventing some errors.

For some additional background, see Aaron Maxwell's ["Unofficial Bash Strict
Mode"](http://redsymbol.net/articles/unofficial-bash-strict-mode/) post.

#### `set -o nounset` / `set -u`

Treat unset variables and parameters other than the special parameters `@` or
`*` as an error when performing parameter expansion. An 'unbound variable'
error message will be written to the standard error, and a non-interactive
shell will exit.

##### Parameter Expansion

This requires using parameter expansion to test for unset variables.

http://www.gnu.org/software/bash/manual/bashref.html#Shell-Parameter-Expansion

The two approaches that are probably the most appropriate are:

    ${parameter:-word}
      If parameter is unset or null, the expansion of word is substituted.
      Otherwise, the value of parameter is substituted. In other words, "word"
      acts as a default value when the value of "$parameter" is blank. If "word"
      is not present, then the default is blank (essentially an empty string).

    ${parameter:?word}
      If parameter is null or unset, the expansion of word (or a message to that
      effect if word is not present) is written to the standard error and the
      shell, if it is not interactive, exits. Otherwise, the value of parameter
      is substituted.

###### Parameter Expansion Examples

Arrays:

    ${some_array[@]:-}              # blank default value
    ${some_array[*]:-}              # blank default value
    ${some_array[0]:-}              # blank default value
    ${some_array[0]:-default_value} # default value: the string 'default_value'

Postitional variables:

    ${1:-alternative} # default value: the string 'alternative'
    ${2:-}            # blank default value

With an error message:

    ${1:?'error message'}  # exit with 'error message' if variable is unbound

##### Usage

Short form:

    set -u

Long form:

    set -o nounset

---

#### `set -o errexit` / `set -e`

Exit immediately if a pipeline returns non-zero.

NOTE: this has issues. When using `read -rd ''` with a heredoc, the exit
status is non-zero, even though there isn't an error, and this setting
then causes the script to exit. `read -rd ''` is synonymous to `read -d $'\0'`,
which means read until it finds a NUL byte, but it reaches the EOF (end of
heredoc) without finding one and exits with a `1` status. Therefore, when
reading from heredocs with `set -e`, there are three potential solutions:

Solution 1. `set +e` / `set -e` again:

    set +e
    read -rd '' variable <<EOF
    EOF
    set -e

Solution 2. `<<EOF || true`:

    read -rd '' variable <<EOF || true
    EOF

Solution 3. Don't use `set -e` or `set -o errexit` at all.

More information:

https://www.mail-archive.com/bug-bash@gnu.org/msg12170.html

##### Usage

Short form:

    set -e

Long form:

    set -o errexit

---

#### `set -o pipefail`

Return value of a pipeline is the value of the last (rightmost) command to
exit with a non-zero status, or zero if all commands in the pipeline exit
successfully.

##### Usage

Long form (no short form available):

    set -o pipefail

---

#### `$IFS`

Set IFS to just newline and tab at the start

http://www.dwheeler.com/essays/filenames-in-shell.html

##### `$DEFAULT_IFS` and `$SAFER_IFS`

`$DEFAULT_IFS` contains the default `$IFS` value in case it's needed, such as
when expanding an array and you want to separate elements by spaces.
`$SAFER_IFS` contains the preferred settings for the program, and setting it
separately makes it easier to switch between the two if needed.

##### Usage

NOTE: also printing `$DEFAULT_IFS` to `/dev/null` to avoid
[ShellCheck](http://www.shellcheck.net/) warnings
about the variable being unused.

    DEFAULT_IFS="$IFS"; printf "%s" "$DEFAULT_IFS" > /dev/null
    SAFER_IFS="$(printf '\n\t')"
    # Then set $IFS
    IFS="$SAFER_IFS"

---

#### 'Strict Mode' TL;DR

Add this to the top of every script:

    # Bash 'Strict Mode'
    # https://github.com/alphabetum/bash-boilerplate#bash-strict-mode
    set -o nounset
    set -o errexit
    set -o pipefail
    DEFAULT_IFS="$IFS"; printf "%s" "$DEFAULT_IFS" > /dev/null
    SAFER_IFS="$(printf '\n\t')"
    # Then set $IFS
    IFS="$SAFER_IFS"

---

### Misc Notes

Explicitness and clarity are generally preferable, especially since bash can
be difficult to read. This leads to noisier, longer code, but should be
easier to maintain. As a result, some general design preferences:

- Group related code into sections with large, easily scannable headers.
- Prefer `printf` over `echo`. For more information, see:
  http://unix.stackexchange.com/a/65819
- Prefer `$explicit_variable_name` over names like `$var`
- Prefer splitting statements across multiple lines rather than writing
  one-liners.
- Describe behavior in comments as much as possible, assuming the reader is
  a programmer familiar with the shell, but not experienced writing shell
  scripts.

---

Copyright (c) 2015 William Melody • hi@williammelody.com
