# Exercises 1: Course overview + the shell

1.  Create a new directory called `missing` under `/tmp`.
    ```
    mkdir /tmp/missing
    ```

1)  Look up the `touch` program. The `man` program is your friend.
    ```
    man touch
    ```

1.  Use `touch` to create a new file called `semester` in `missing`.
    ```
    touch /tmp/missing/semester
    ```

1)  Write the following into that file, one line at a time:
    ```
    #!/bin/sh
    curl --head --silent https://missing.csail.mit.edu
    ```
    The first line might be tricky to get working. It's helpful to know that
    `#` starts a comment in Bash, and `!` has a special meaning even within
    double-quoted (`"`) strings. Bash treats single-quoted strings (`'`)
    differently: they will do the trick in this case. See the Bash
    [quoting](https://www.gnu.org/software/bash/manual/html_node/Quoting.html)
    manual page for more information.

1.  Try to execute the file, i.e. type the path to the script (`./semester`)
    into your shell and press enter. Understand why it doesn't work by
    consulting the output of `ls` (hint: look at the permission bits of the
    file).

    It prints `-bash: ./semester: Permission denied` Type `ls -l` to look at the permission bits of the file. The first part of the print should look like `-rw-r--r--`, which means execution is not permitted but only reading (as denoted in _r_) and writing (as denoted in _w_).

1)  Run the command by explicitly starting the `sh` interpreter, and giving it
    the file `semester` as the first argument, i.e. `sh semester`. Why does
    this work, while `./semester` didn't?

    Type `man sh`. It prints `sh` is a POSIX-compliant command interpreter. `sh` specifies to the shell that the file `semester` is supposed to be interpreted i.e. executed using `sh`.

1)  Look up the `chmod` program (e.g. use `man chmod`).

    Type `man chmod`. It prints `chmod` changes file modes or Access Control Lists. Adding the tag `+x` before the file name argument would change the file mode to be executable.

1.  Use `chmod` to make it possible to run the command `./semester` rather than
    having to type `sh semester`. How does your shell know that the file is
    supposed to be interpreted using `sh`? See this page on the
    [shebang](<https://en.wikipedia.org/wiki/Shebang_(Unix)>) line for more
    information.

    ```
    chmod +x ./semester
    ./semester
    ```

1)  Use `|` and `>` to write the "last modified" date output by
    `semester` into a file called `last-modified.txt` in your home
    directory.

    ```
    date -r ./semester | cat > ~/last-modified.txt
    ```

1.  Write a command that reads out your laptop battery's power level or your
    desktop machine's CPU temperature from `/sys`. Note: if you're a macOS
    user, your OS doesn't have sysfs, so you can skip this exercise.

    Skip
