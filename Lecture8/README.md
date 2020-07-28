# Exercises 8: Metaprogramming

1.  Most makefiles provide a target called `clean`. This isn't intended
    to produce a file called `clean`, but instead to clean up any files
    that can be re-built by make. Think of it as a way to "undo" all of
    the build steps. Implement a `clean` target for the `paper.pdf`
    `Makefile` above. You will have to make the target
    [phony](https://www.gnu.org/software/make/manual/html_node/Phony-Targets.html).
    You may find the [`git ls-files`](https://git-scm.com/docs/git-ls-files) subcommand useful.
    A number of other very common make targets are listed
    [here](https://www.gnu.org/software/make/manual/html_node/Standard-Targets.html#Standard-Targets).

    ```Makefile
    paper.pdf: paper.tex plot-data.png
    	pdflatex paper.tex

    plot-%.png: %.dat plot.py
    	./plot.py -i $*.dat -o $@

    .PHONY: clean
    clean:
    	git ls-files -o | xargs rm -f
    ```

    Explicitly declaring the target to be `.PHONY` of which its dependency is `clean`, it will force `clean` to be executed everytime, whereas without it as `clean` is without dependence its recipe will not be executed if there happens to be a file named `clean` in the directory.

    Run `git add -A` with the `Makefile` defined as above and `paper.tex`, `plot.py` (make sure to run `chmod +x plot.py` to execute), `data.dat` defined as in the lecture notes.

    `git ls-files -o` is a command that list all files that are untracked by git. Undoing all of the build steps is equivalent to removing all untracked files i.e. the newly created `paper.pdf` and other files created by the `make paper.pdf` command, and the `rm -f` command can be pipelined. Running `make clean` will be now our equivalent of "undo".

2)  Take a look at the various ways to specify version requirements for
    dependencies in [Rust's build
    system](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html).
    Most package repositories support similar syntax. For each one
    (caret, tilde, wildcard, comparison, and multiple), try to come up
    with a use-case in which that particular kind of requirement makes
    sense.

    - Caret requirements:

      An update is allowed if the new version number does not modify the left-most non-zero digit in the major, minor, patch grouping. Examples better illustrate this:

      ```
      ^1.2.3  :=  >=1.2.3, <2.0.0
      ^1.2    :=  >=1.2.0, <2.0.0
      ^1      :=  >=1.0.0, <2.0.0
      ^0.2.3  :=  >=0.2.3, <0.3.0
      ^0.2    :=  >=0.2.0, <0.3.0
      ^0.0.3  :=  >=0.0.3, <0.0.4
      ^0.0    :=  >=0.0.0, <0.1.0
      ^0      :=  >=0.0.0, <1.0.0
      ```

      An update is only allowed without a change in the major version if the caret requirement is specificed to be `python ^2.7.2`, for example. Any other minor or patch updates should be fine, but it should not update to anything of `python3`.

    - Tilde requirements:

      If you specify a major, minor, and patch version or only a major and minor version, only patch-level changes are allowed. If you only specify a major version, then minor- and patch-level changes are allowed.

      ```
      ~1.2.3  := >=1.2.3, <1.3.0
      ~1.2    := >=1.2.0, <1.3.0
      ~1      := >=1.0.0, <2.0.0
      ```

      In essence, this is a stricter version of caret requirements. An update is only allowed in the patch version if the tilde requirement is specificed to be `python ~2.7.2`, for example. Any other updates beyond `python 2.8`, including `python3`, are prohibited.

    * Wildcard requirements:

      Wildcard requirements allow for any version where the wildcard is positioned. Much like regex logic.

      ```
      *     := >=0.0.0
      1.*   := >=1.0.0, <2.0.0
      1.2.* := >=1.2.0, <1.3.0
      ```

      Any update is allowed as long as the major version remains if the wildcard requirement is specificed to be `python 2.*`, for example. `python3`, however, is prohibited.

    * Comparsion requirements:

      Comparison requirements allow manually specifying a version range or an exact version to depend on. Quite straightforward.

      ```
      >= 1.2.0
      > 1
      < 2
      = 1.2.3
      ```

      Any update is allowed if the comparison requirement is specificed to be `python >= 2.7`, for example. (Unless you regress instead of update) `python3` would also be allowed.

    * Multiple requirements

      Multiple version requirements can be separated with a comma, e.g., `>= 1.2, < 1.5`. This would be useful in setting an interval for version requirements for dependencies.

3.  Git can act as a simple CI system all by itself. In `.git/hooks`
    inside any git repository, you will find (currently inactive) files
    that are run as scripts when a particular action happens. Write a
    [`pre-commit`](https://git-scm.com/docs/githooks#_pre_commit) hook
    that runs `make paper.pdf` and refuses the commit if the `make`
    command fails. This should prevent any commit from having an
    unbuildable version of the paper.

    To implement a `pre-commit`, overwrite `pre-commit.sample` in the `./.git/hooks` folder and rename it `pre-commit`.

    ```
    $ cd ./.git/hooks
    $ vim pre-commit.sample
    ```

    `pre-commit`:

    ```bash
    #!/bin/sh

    if ! make paper.pdf ; then
        echo "Cannot make paper.pdf"
        exit 1
    fi
    ```

    For the purposes of testing, `chmod -x plot.py` to make it unexecutable and yield an error purposefully. It shows the `pre-commit` hook successfully prevents any commit from having an
    unbuildable version of the paper.

    ```
    $ git commit -m 'failure test'
    make: *** No rule to make target `paper.pdf'.  Stop.
    Cannot make paper.pdf
    ```

4)  Set up a simple auto-published page using [GitHub
    Pages](https://help.github.com/en/actions/automating-your-workflow-with-github-actions).
    Add a [GitHub Action](https://github.com/features/actions) to the
    repository to run `shellcheck` on any shell files in that
    repository (here is [one way to do
    it](https://github.com/marketplace/actions/shellcheck)). Check that
    it works!

    Go to the Github remote repository and click Settings. Scroll down and there is a Github Pages section. By default it will create a simple auto-published page that displays `README.md`. Tested it on my MIT missing semester repository. [(Link)](https://ivan-kim.github.io/MIT-missing-semester/)

    Go to the Github remote repository and click Actions. Create a new workflow file `main.yml`. The workflow file could be like this.

    ```
    name: Shellchecking

    on:
      push:
        branches: [ master ]

    jobs:
      build:
        runs-on: ubuntu-latest

        steps:
        - uses: actions/checkout@v2

        - name: shellcheck
          uses: ludeeus/action-shellcheck@0.1.0
    ```

    Then on the local terminal do `git pull origin master`. Try making a faulty `wrongshell.sh` file and push it to the Github repository. If there is a green check next to the commit message in the Actions tab, that means it should be working. Click on the `build` section with a green check on the left. Each step of the workflow is logged. The `shellcheck` subsection is where the `shellcheck` print messages are logged.

5.  [Build your
    own](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/building-actions)
    GitHub action to run [`proselint`](http://proselint.com/) or
    [`write-good`](https://github.com/btford/write-good) on all the
    `.md` files in the repository. Enable it in your repository, and
    check that it works by filing a pull request with a typo in it.
