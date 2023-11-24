The asterisk `*` pattern matches any sequence of zero or more characters (except for a leading dot) in file
or directory paths:

For e.g. `ls *.txt` will list all files with the extension `.txt`.

## Things to note

- Every time a command runs, some steps are the responsibility of the invoked program, such as `ls`, and
  some are the responsibility of the _shell_.
- Expert users understand which is which
- In the case of pattern matching (like in `ls *.txt`), **the _shell_, not the invoked program `ls`, performs
  the pattern matching**
- The _shell_ pattern matching applies only to file and directory paths. It _doesn't_ work for usernames,
  hostnames, and other types of arguments that certain commands accept.
