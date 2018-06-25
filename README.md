# Git pull request tools

Everyone abstractly likes pull requests, but they can be a lot of keystrokes:
making new branches, pushing them, and especially keeping track of
what can be deleted.  This is an attempt to reduce the number of
keystrokes to a bare minimum.

## PR workflows

Here is our current short PR workflow (1):

1. `git prnew $brname`: create a new branch based on inferred
   `upstream/HEAD`.  (note: we don't currently automatically
   re-fetch).

2. Do work, commit, etc.

3. `git prpush`: Infer current branch name and push to inferred
   remote.  You can give a different name to push to a different
   branch.

4. Once you are done, `git prrm $brname` to remove both local and
   remote branches (again inferring upstream remote).

There is actually an even shorter way (2):

1. `git prnew`: create a detached head, don't even name it locally.

2. Do work, commit, etc.  If you change your mind, no need to remove
   anything.

2. `git prpush $brname`: push to inferred upstream.  Note you need to
   give a name since we don't have a local branch name.

3. You don't need to remove anything - remove branch remotely (via
   Github web) or delete your fork if this was really a one-time
   thing.



Common assumptions:

- The upstream remote is `upstream`, `origin`, or the first remote in
  the `git remote` output, whichever is found first.

- You always push the `origin`, `upstream`, or first remote found,
  whichever comes first.


## Installation and usage of `git-pr`

In this repository, you find a shell script `git-pr` which, when
placed anywhere on your `$PATH`, provides the following commands.  for
each command, you can run `-h` to get help (with no arguments).  This
is the recommended way to use this, because the shell script provides
the most flexibility and power.

* `git pr new`: create a new PR based on the current
  `(inferred_upstream)/HEAD`.  See `-h` for some considerations.  Also
  at the top of the script is a configuration option to force a
  `fetch` before to make sure you are up to date.  With one argument,
  create a branch of this name, otherwise create a detached head.

* `git pr push`: Push a PR.  With no arguments, send to inferred
  origin automatically with a name the same as the current branch.
  With one argument, send to a branch of that name.  With two
  arguments, the first is the remote name to use, and the second is
  the branch name to push to.

* `git pr di`: Diff between current working dir and merge-base of
  inferred_upstream.

* `git pr rm $branch_name`: Remove this branch, both locally and
  on inferred_origin.

* `git pr fetch $pr_number`: Fetch the given upstream PR to a new
  local remote branch `inferred_upstream/pr/$pr_number`. (all fetch
  commands support github at least)

* `git pr fetchall`: Fetch all remote upstream PRs to local
  repository.  Warning: this includes all PRs, open and closed.

* `unfetchall`: Remove all `inferred_upstream/pr/$pr_number`
  branches.  Warning: *all*.


## Aliases

These are old aliases which predated and somewhat reproduce the
functionality of the `git-pr` script.  If you can use the `git-pr`
script, it is more powerful, but these old one-liners may still be
useful to someone.

```
# Create a new HEAD suitable for a pull request.  Use
# upstream/HEAD, origin/HEAD, or (the first remote)/HEAD as
# the base.  If a argument is given, create a branch of this
# name, otherwise create a detached head here.
prnew = !sh -x -c 'git checkout $(test $1 && echo "-b" $1) $(git remote | grep upstream || git remote | grep origin || git remote | head -1)/HEAD' -

# Push a PR branch.  If one argument given, push to that
# branch on guessed remote (same logic as above).  This is
# suitable for the local detached heads workflow.  If two
# arguments given, the first is a remote and the second is
# branch name.  If no arguments given, use the current branch
# name as the remote branch name (only works if you have a
# local branch).
prpush = !sh -x -c 'git push $(test $2 && echo $1 || echo origin) HEAD:refs/heads/${2:-${1:-$(git symbolic-ref HEAD | cut -d/ -f3)}}' -

# Compare current working tree to upstream merge-base.  Uses
# same logic to find upstream merge base as "prnew"
prdi = !git diff $(git merge-base $(git remote | grep upstream || git remote | grep origin || git remote | head -1)/HEAD HEAD)

# Delete a branch (argument 1) both locally and remotely.
prrm = !sh -x -c "'git branch | grep $1 && git branch -d $1 ; git branch -a | grep origin/$1 && git push origin :$1'" -


# Fetch PR refs to local.  Give one PR numbers, and they will
# be pulled as $remote/pr/$number.  (This should be modified
# to take multiple PR numbers)
prfetch = !sh -x -c 'git fetch $(git remote | grep upstream || git remote | grep origin || git remote | head -1) --refmap="+refs/pull/*/head:refs/remotes/$(git remote | grep upstream || git remote | grep origin || git remote | head -1)/pr/*" refs/pull/$1/head' -

# Fetch all PRs from a certain remote
prfetchall = !sh -x -c 'git fetch upstream "+refs/pull/*/head:refs/remotes/$(git remote | grep upstream || git remote | grep origin || git remote | head -1)/pr/*"' -

# Delete everything fetched by prfetchall.  Note, this deletes
# these refs from *all* remotes, not just the default
# upstream.  It will delete anything matching '/pr/[0-9]+'
prunfetchall = !sh -x -c 'git branch --remote -d `git branch --remote | grep -E '/pr/[0-9]+$'`' -
```

## Possible problems

We use the remote HEAD to infer what the upstream branch is.  There
are some problems with this:

- it is only set when first cloned, if default branch changes it won't
  be updated later.  However, you can manually set this with `git
  remote set-head ${inferred_upstream} $branch_name`

- If multiple branches had the same HEAD as the default branch, the
  remote default branch can't be inferred automatically.

- Setting the option `NEW_ALWAYS_FETCH=1` in the file solves this, at
  the cost of network access for `git pr new`.


## Feedback

Please send feedback.  By its very nature, this makes some choices
about how workflows work.  If these can be improved to suit other
workflows, or you have ideas, please send pull requests.


## Other resources

* This is seems nice for automatically fetching PRs: https://gist.github.com/piscisaureus/3342247
* Related, similar but slightly less features: https://gist.github.com/gnarf/5406589
* Alias to fetch a single PR by ID: https://davidwalsh.name/pull-down-pr
* Has some super complicated aliases which I haven't examined yet:
  https://gist.github.com/metlos/9368527
* Big library of aliases (not read yet, no obvious relevant ones): https://github.com/Ajedi32/git_aliases

