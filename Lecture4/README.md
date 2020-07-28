# Exercises 4: Data Wrangling

1.  Take this [short interactive regex tutorial](https://regexone.com/).
2.  Find the number of words (in `/usr/share/dict/words`) that contain at
    least three `a`s and don't have a `'s` ending. What are the three
    most common last two letters of those words? `sed`'s `y` command, or
    the `tr` program, may help you with case insensitivity. How many
    of those two-letter combinations are there? And for a challenge:
    which combinations do not occur?

    - Find the number of words (in `/usr/share/dict/words`) that contain at
      least three `a`s and don't have a `'s` ending. What are the three
      most common last two letters of those words?

           ```
           cat /usr/share/dict/words | tr "[:upper:]" "[:lower:]" | grep -E "^([^a]*a){3}.*$" | grep -v "\'s$" | wc -l
           ```

           `cat /usr/share/dict/words` prints the contents of the file `words` to the standard output (terminal).
           `tr "[:upper:]" "[:lower:]"` translates the contents of the file to lower-case using the `tr` program.
           `grep -E "^([^a]*a){3}.*$"` searches the file using extended regular expression. The regular expression matches strings for sequences of non-`a` characters (`[^a]*`) that ends in `a`, for three times. (`([^a]*a){3}`) The rest of the string could contain whatever chracter however many times. (`.*`)
           `grep -v "\'s$"` invert matches for excluding specific string `\'s`. Note `-v` tag allows for inverse matching.
           `wc -l` counts lines in file. This is equivalent to the total number of words as each words is separated by a new line.

    - What are the three most common last two letters of those words?

      ```
      cat /usr/share/dict/words | tr "[:upper:]" "[:lower:]" | grep -E "^([^a]*a){3}.*$" | grep -v "\'s$" | sed -E "s/.*([a-z]{2})$/\1/" | sort | uniq -c | sort -n | tail -n3
      ```

      `sed -E "s/.*([a-z]{2})$/\1/"` replaces all occurrences of the regular expression to the first capture group. The regular expression captures the last two lower-case alphabet characters.  
       `sort | uniq -c` displays number of occurrences of each line along with that line.
      `sort -n` perfroms numeric sort, in an ascending order.
      `tail -n3` returns the last three items, which should be the three most common last two letters.

    - How many of those two-letter combinations are there?

      ```
      cat /usr/share/dict/words | tr "[:upper:]" "[:lower:]" | grep -E "^([^a]*a){3}.*$" | grep -v "\'s$" | sed -E "s/.*([a-z]{2})$/\1/" | sort | uniq | wc -l
      ```

    - And for a challenge: which combinations do not occur?

      ```
      cat /usr/share/dict/words | tr "[:upper:]" "[:lower:]" | grep -E "^([^a]*a){3}.*$" | grep -v "\'s$" | sed -E "s/.*([a-z]{2})$/\1/" | sort | uniq > last_letters

      source letters.sh > all_letters

      diff --changed-group-format="%<" --unchanged-group-format="" all_letters last_letters
      ```

      Output the two-letter combinations in file `words` to another file `last_letters`.  
       Use a shell script to perform nested loops on all possible lower-case two-letter combinations as in `letters.sh` and output the result to another file `all_letters`.  
       Use `diff` program to print the difference between the two files `all_letters` and `last_letters`. You can use `--changed-group-format` and `--unchanged-group-format` options to filter required data. Following three options can use to select the relevant group for each option:
      `%<` get lines from FILE1
      `%>` get lines from FILE2
      '' (empty string) for removing lines from both files. [(Source)](https://stackoverflow.com/questions/14500787/comparing-two-files-in-linux-terminal)

3.  To do in-place substitution it is quite tempting to do something like
    `sed s/REGEX/SUBSTITUTION/ input.txt > input.txt`. However this is a
    bad idea, why? Is this particular to `sed`? Use `man sed` to find out
    how to accomplish this.

    `sed s/REGEX/SUBSTITUTION/ input.txt > input.txt` will return a blank file, since `input.txt` on the right hand side of the output operator `>` will be made blank before the left hand side of the output operator could be applied. Because the file names conicide, an empty `input.txt` will be output to an empty `input.txt`.

    Use `man sed` to find out how to accomplish in-place substitution.

    ```
    -i extension
             Edit files in-place, saving backups with the specified extension.
             If a zero-length extension is given, no backup will be saved.  It
             is not recommended to give a zero-length extension when in-place
             editing files, as you risk corruption or partial content in situ-
             ations where disk space is exhausted, etc.
    ```

    It is generally a bad idea to in-place edit files as mistakes can be made when using regular expressions and without backups on the original file, it can be very difficult to recover. This is not particular to `sed` but in general to all forms of in-place editing files. To accomplish in-plface substitution add the `-i` tag.

4.  Find your average, median, and max system boot time over the last ten
    boots. Use `journalctl` on Linux and `log show` on macOS, and look
    for log timestamps near the beginning and end of each boot. On Linux,
    they may look something like:

    ```
    Logs begin at ...
    ```

    and

    ```
    systemd[577]: Startup finished in ...
    ```

    On macOS, [look
    for](https://eclecticlight.co/2018/03/21/macos-unified-log-3-finding-your-way/):

    ```
    === system boot:
    ```

    and

    ```
    Previous shutdown cause: 5
    ```

    Output the matching logs to a separate file `bootlog` since it takes a while to run through a month worth of log history everytime.

    ```
    log show | grep -E "=== system boot:|Previous shutdown cause: 5" > bootlog
    ```

    A line of the matched log looks like this on macOS:

    ```
    2020-04-29 09:46:04.022430+0900 0x0        Timesync    0x0                  0      0    === system boot: C4C57548-7964-483A-A0E9-8FD49A6088C6
    2020-04-29 09:46:04.683152+0900 0xa9       Default     0x0                  0      0    kernel: (AppleSMC) Previous shutdown cause: 5
    ```

    The line of interest the second column of the logs, of which their difference denotes the time how long the system took to boot.

    ```
    cat bootlog | awk '{print $2}' | sed -E "s/^.*:(.*)/\1/" | xargs -n2 | awk '{print $2"-"$1}' | bc | R --slave -e 'x <- scan(file="stdin", quiet=TRUE); summary(x)'
    ```

    `awk '{print $2}'` only prints the second column of the `bootlog`.  
     `sed -E "s/^.*:(.*)/\1/"` dismisses the hour and minute of the time log, and replaces the logs with the simplified version of seconds (as boot time is often around a few seconds).  
     `xargs -n2` rearranges the logs to be shown in two columns.  
     `awk '{print $2"-"$1}'` concatenates a subtraction character "-" between the second column and the first column. Note that this print is interpreted as a string type, not an operation between integer type values.  
     `bc` turns the previous strings to an integer operation.  
     `R --slave -e 'x <- scan(file="stdin", quiet=TRUE); summary(x)'` can be used as in the lectures to compute the average, median, and max boot time. Programming language `R` must be installed in order to run this command. Or install [st](https://github.com/nferraz/st) and replace the command with `st --mean --median --max`. There does not seem to be a 'vanilla' method to perform this step.

5.  Look for boot messages that are _not_ shared between your past three
    reboots (see `journalctl`'s `-b` flag). Break this task down into
    multiple steps. First, find a way to get just the logs from the past
    three boots. There may be an applicable flag on the tool you use to
    extract the boot logs, or you can use `sed '0,/STRING/d'` to remove
    all lines previous to one that matches `STRING`. Next, remove any
    parts of the line that _always_ varies (like the timestamp). Then,
    de-duplicate the input lines and keep a count of each one (`uniq` is
    your friend). And finally, eliminate any line whose count is 3 (since
    it _was_ shared among all the boots).

    Skipped (system not Linux)

6.  Find an online data set like [this
    one](https://stats.wikimedia.org/EN/TablesWikipediaZZ.htm), [this
    one](https://ucr.fbi.gov/crime-in-the-u.s/2016/crime-in-the-u.s.-2016/topic-pages/tables/table-1).
    or maybe one [from
    here](https://www.springboard.com/blog/free-public-data-sets-data-science-project/).
    Fetch it using `curl` and extract out just two columns of numerical
    data. If you're fetching HTML data,
    [`pup`](https://github.com/EricChiang/pup) might be helpful. For JSON
    data, try [`jq`](https://stedolan.github.io/jq/). Find the min and
    max of one column in a single command, and the sum of the difference
    between the two columns in another.
