# Exercises 6: Version Control (Git)

1.  If you don't have any past experience with Git, either try reading the first
    couple chapters of [Pro Git](https://git-scm.com/book/en/v2) or go through a
    tutorial like [Learn Git Branching](https://learngitbranching.js.org/). As
    you're working through it, relate Git commands to the data model.
1.  Clone the [repository for the
    class website](https://github.com/missing-semester/missing-semester).

    ```
    git clone https://github.com/missing-semester/missing-semester class_website
    ```

        1. Explore the version history by visualizing it as a graph.
             ```
             cd ./class_website
             git log --all --graph --decorate
             ```
        1. Who was the last person to modify `README.md`? (Hint: use `git log` with
           an argument)
             ```
             git log --all --graph --decorate README.md
             ```
             Note that the top of the tree graph denotes the latest modification.
             ```
             * commit 9b2d860b1fe9e6ada1b564de3438db091c4d56a4
             | Author: Anish Athalye <me@anishathalye.com>
             | Date:   Sat May 16 11:10:40 2020 -0400
             |
             |     Add build status to README
             ```

        1. What was the commit message associated with the last modification to the
           `collections:` line of `_config.yml`? (Hint: use `git blame` and `git
           show`)
           ```
           git blame _config.yml
           ```
           It prints that the corresponding hash for the commit is a88b4eac.
           ```
           git show a88b4eac
           ```
           It shows the commit message is "Redo lectures as a collection".

1) One common mistake when learning Git is to commit large files that should
   not be managed by Git or adding sensitive information. Try adding a file to
   a repository, making some commits and then deleting that file from history
   (you may want to look at
   [this](https://help.github.com/articles/removing-sensitive-data-from-a-repository/)).

   ```
   git filter-branch --force --index-filter \
   "git rm --cached --ignore-unmatch PATH-TO-YOUR-FILE-WITH-SENSITIVE-DATA" \
   --prune-empty --tag-name-filter cat -- --all
   ```

1) Clone some repository from GitHub, and modify one of its existing files.
   What happens when you do `git stash`? What do you see when running `git log --all --oneline`? Run `git stash pop` to undo what you did with `git stash`.
   In what scenario might this be useful?

   `git-stash` - Stash the changes in a dirty working directory away.

   `git log --all --oneline` prints `(refs/stash) WIP on master:` followed by the hash id and the commit message of the latest commit i.e. the `HEAD` commit before the local change. After running `git stash pop`, the stash log will disappear.

   `git-stash` would be useful when checking out to other commits but do not want to overwrite the local changes not yet committed, either because they don't seem good enough yet to commit or there are more urgent bugs to address at the moment, etc.

1) Like many command line tools, Git provides a configuration file (or dotfile)
   called `~/.gitconfig`. Create an alias in `~/.gitconfig` so that when you
   run `git graph`, you get the output of `git log --all --graph --decorate --oneline`.

   ```
   [alias]
   graph = log --all --graph --decorate --oneline
   ```

   Note the syntax of defining aliases is different from defining aliases in `.bashrc`, for example.

1) You can define global ignore patterns in `~/.gitignore_global` after running
   `git config --global core.excludesfile ~/.gitignore_global`. Do this, and
   set up your global gitignore file to ignore OS-specific or editor-specific
   temporary files, like `.DS_Store`.
1) Clone the [repository for the class
   website](https://github.com/missing-semester/missing-semester), find a typo
   or some other improvement you can make, and submit a pull request on GitHub.

   For an exercise in Lecture 9: Security, I spotted a typo that incorrectly identified a git sigining feature as `git commit -C` instead of `git commit -S`.

   I rasied an issue on the official MIT Missing Sem repo and got affirmed it was a typo indeed. [(My issue)](https://github.com/missing-semester/missing-semester/issues/56)
