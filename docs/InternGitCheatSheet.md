# Intern Git Cheat Sheet #

## Terminology ##

Key words to know:
Branch, Clone, Fetch, Fork, Head, Merge, Origin, Pull Request, Push, Rebase, Upstream

(https://acloudguru.com/blog/engineering/git-terms-explained)



## Useful Habits ##
- Commit **Early**, Commit **Often**.

- Commit **Often**, Perfect **Later**.

- Follow good git commit message rules (https://github.com/erlang/otp/- wiki/writing-good-commit-messages). In summary, first line 50 characters max in imperative language. Skip a line then more detailed explanation in imperative language.

- Never work directly on the main branch. create your own, also never push to the main branch directly.

- Reference related issues in your commits.

- After cloning your fork it is useful to add a remote to your the upstream project: `git remote add upstream https://github.com/coreos/<project>.git`. This will allow you to frequently pull the latest changes via `git pull upstream`. 

## Typical Scenarios ##


### Basic Workflow ###

After discussion on a GitHub issue, you decide to tackle it, you clone the repo, then you create a branch to work on. You switch to that branch, start working on it. You add commits and good commit messages. when you are done, you push your changes.

```
git clone git@github.com:coreos/YourRepoHere.git

git checkout -b YourBranchNameHere

git add .

git commit

git push -u origin YourBranchNameHere
```

### You have a few commits and you would like to squash them into a single commit ###

Find out how many commits you would like to squash, then use:

```
git rebase -i HEAD~[X]
```

where [X] is the number of commits you would like to see in the interactive menu.

 ```
 E.g 4 = "git rebase -i HEAD~4"
 ```
Then you will see a menu showing the 4 commits with the word pick before them:
![Squash commits](https://raw.githubusercontent.com/mohelt/InternCheatSheets/main/images/rebase.webp)

Pick means you want the commit to remain. Squash means you want the changes in the commit to be melded to the first commit above it with the word pick before it.

## Pulling PRs Locally ##

Sometimes you may need to test a change in a PR (ie. during PR review). The following alias can be added to `~/.gitconfig`:

```bash
[alias]
copr = "!f() { git fetch $1 pull/$2/head:PR${2}; git checkout PR${2};}; f"
```
Now you can use the upstream remote and the PR number to copr (checkout PR :)).

```
$ git copr upstream 1912
```

Credits go to Dusty Mabe for showing me this.
#### References ####
https://docs.google.com/document/d/156sIos7H2h0-rOn1C36SBTn-emfQN4UKqvSi8XMlQBk/edit

https://github.com/erlang/otp/wiki/writing-good-commit-messages

https://medium.com/@lorenzen.jacob/good-git-habits-73db205533d0

https://www.baeldung.com/ops/
git-squash-commits

https://thoughtbot.com/blog/git-interactive-rebase-squash-amend-rewriting-history