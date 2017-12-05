<!-------------------------------------------------------------><a id="top"></a>
# git :: Getting Information
<!----------------------------------------------------------------------------->

<!-- introduction -->
[Official documentation](https://git-scm.com/docs)

##### TOC #####
- [Commands](#commands)  
- [status](#status)  
- [log](#log)  

<!--------------------------------------------------------><a id="commands"></a>
## Commands
<!----------------------------------------------------------------------------->
Listing favourite commands as a cheatsheet.
Note that some of these are custom commands added to the git configuration.
These are marked with `(!)`
``````
(!) git glog
    gitk --all
    git log -L :"regex name":<file> [-L :"regex name":<file>]
``````


<!--
Commands:
- `git log`
- `gitk --all` show commit and branching history in gitk
- `git diff`
- `git diff --staged`
- `git branch` list branches
- `git branch -r` list remote tracking branches
- `git branch -a` list all branches
- `git tag`
- `git tag -l` list available tags
- `git remote -v` basic info
- `git remote show origin` advanced info
- `git show`
- `git ls-files -u` list files that need merging;
  gives multiple versions if available, numberd `<stage#>`
  show different version with:
  - `git show :<stage#>:<filename>`
  - `git show <branch>:<filepath>`
- `git describe` describes from HEAD
  output is:
````
<tag>_<numCommits>_g<hash>
<tag> closest ancestor tag
<numCommits> distance to tag
<hash> hash of described commit
````
- `git describe <rev>` describes a location within the repository
-->

<!----------------------------------------------------------><a id="status"></a>
## status
<!----------------------------------------------------------------------------->
`git status` The most used command.
Lists files in the working directory that differe from the current HEAD
(unless ignored by .gitignore).

[git status](https://git-scm.com/docs/git-status)

### Useful options
`git status -s` compact overview  
`git status --ignored` also shows the ignored files and directories

<!-------------------------------------------------------------><a id="log"></a>
## log
<!----------------------------------------------------------------------------->
`git log` Explorer the commit history.

[git log](https://git-scm.com/docs/git-log)

### Custom command
`git glog` Pretty output, similar to what gitk displays.
Represents: `git log --graph --oneline --decorate --all`  
Create with: `git config --global alias.glog "log --graph --decorate --all"`

### Useful options
- `git log -- <path>` Show commits applying to the file(s) in that location.  
- `git log --follow <file>` Track file past renaming (only for single files).  

Limiters
<!-- there is also a revision range notation -->
- `git log -<number>` Limit log length to `<number>` displayed commits.  
  Short for `--max-count=<number>`.
- `git log --skip=<number>` skip commits befor outputting.
- `git log --reverse` Reverse output order. Default is newest on top.  
  Always runs last, won't influence other limiters.
- `git log --since=<data>` = `git log --after=<data>` <!-- really or one day difference? -->
- `git log --until=<data>` = `git log --before=<data>`
- `git log --author=<regex>` and `git log --commiter=<regex>`  
  Can be used multiple times to select multiple autors/commiters.
  <!-- difference author versus commitor? -->
- `git log --grep=<regex> [--all-match] [--invert-grep]`
- `git log --grep-reflog=<regex>`  
  Regex settings for Limiters:
  - `-i` == `--regexp-ignore-case`
  - `-E` == `--extended-regexp` <!-- what is in basic, what in extenden? -->
  - `-F` == `--fixed-string` not a regex, compare as string
  - `-P` == `--perl-regexp` <!-- compile-time dependency, In git for Windows? -->
- `-remove-empty` ??
- `--merges` == `--min-parents=2` ?? output only merge commits
- `--no-merges` == `--max-parents=1`
- `--min-parents=<number>`
- `--no-min-parents` reset limit
- `--max-parents=<number>`
- `--no-max-parents` reset limit
- `--first-parent`
<!-- Way more possible! -->

#### Within files
Displays the within the given section of the file.  
**Note**: Not compatible with any other flags. (`--reverse` works)

`git log -L :<funcname>:<file>`
Where `<funcname>` is a regex (no / enclosing here).  
Displays the section from line with the first occourence up to the next (non
brace) on the item with the same indentation. <!-- needs more experimenting -->
Usable for displaying a function of a class definition.
Note: it only displays the first match.
To display multiple matches, repeat the `-L :<funcname>:<file>` statement.
Subsequent statements are start matching after the first match.
To force starting from the begin of the file use: `-L ^:<regex>:<file>`.
At least the last version needs to contain all the matches.<!-- check this -->


`git log -L <start>,<end>:<file>`
  `<start>` And `<end>` can be:
  - line numbers (inclusive range, files start at line 1)
  - regex (POSIX-format) `"/regex/"`  
    Searches for the first line containing a match.  
    For `<start>`: searches from the begin of the file OR the end of a previous
    `-L` range!
    This allows for tracking multiple sections within the file.
    Using `"^/regex/"` will always search from the begin of the file.
    **Can't get it to work ...**
    For `<end>`: searches from `<start>`
    **Not working in git version 2.15.0.windows.1**
  - only for `<end>`: `+<length>` or `-<length>` relative to `<start>`.  
    **Note**: `+1` or `-1` displays just `<start>` (equal to `<start>,<start>`).

`git log -L '/int main/',/^}/:main.cpp`
Shows how the function `main()` in `main.cpp` evolved over time.

-----------
[top](#top)
