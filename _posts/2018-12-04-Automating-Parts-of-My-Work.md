Much of what modern programmers do is already automated, but I felt I could automate things further and enjoy development loops with even less repetition. The more I automate with bash, the more I understand how others use `vim` or `emacs` as their IDEs&mdash; though I use [VS Code](https://code.visualstudio.com/) for most of my editing. More valued than the small amount of efficiency gained, I enjoy the process of abstracting some specific, repetitive task to the press of a button.

- I wrote [a bash command to switch the local branch to updated master](#switch-the-local-branch-to-updated-master)
- I wrote [a bash command to update all local master branches](#update-all-local-master-branches)
- I wrote [a bash command to create and open a Pull Request](#create-and-open-a-pull-request)
- I wrote [a bash command to show the contributors on a file](#show-the-contributors-on-a-file)
- I wrote [a bash command to remove exclusive testing in my local changes](#remove-exclusive-testing-in-my-local-changes)
- I created [a status bar in my terminal showing branch and CI status](#terminal-status-bar-showing-branch-and-ci-status)

## Switch the local branch to updated master

```bash
alias gmm='git checkout master -q; git pull'
```

I use this all the time; I often start writing code in my IDE then realize I'm on an unrelated branch, or I want to work on the newest copy of master from GitHub. I call it `gmm` as I tend to name aliases using muliple of the same character, to save my fingers.

## Update all local master branches

```bash
#!/bin/bash

pull_if_master () {
  [[ -d .git ]] \
    && [[ "master" = $(git rev-parse --abbrev-ref HEAD) ]] \
    && [[ -z $(git diff) ]] \
    && echo "Pulling $1 master!" \
    && git pull
}

cd ~/my_repo_folder
for dir in */
do
  cd $dir
  pull_if_master $dir
  cd ..
done
```

I wanted to automate running the previous `gmm` script on all of my repos so I can get the most recent copies from remote all at once, but I wanted to be safe about it--I wrote the script to only pull for directories that are already on `master` and have no local changes.

## Create and open a Pull Request

```bash
#!/bin/bash
set -e

echo "Creating branch '$1'..."
git checkout -b $1

echo "Committing current changes..."
git add -A
git commit -a -m "$3"

git push -u origin $1
echo "Creating PR '$2' on GitHub..."
PR_URL="$(hub pull-request -m "$2")"
echo $PR_URL
/usr/bin/open "$PR_URL"
```

A while ago I found GitHub's [`hub`](https://github.com/github/hub#git--hub--github) program, which I installed with `brew` on Mac. I wrote this script leveraging it because I often find myself with a set of changes on `master` in a local repo, and want to turn them into a new branch with a PR. This script gets me from point A to point B with one command, which takes three arguments:

`gitpr name-of-branch "Name of the new Pull Request" "First commit message"`

## Show the contributors on a file

```bash
#!/bin/bash

git blame --line-porcelain $* |
  sed -n 's/^author //p' |
  sort | uniq -c | sort -rn
```

With some help from the internet, I put together this command to show me a sorted list of who last changed the lines of a file. This gives me a simplified picture of who has dealt with a file recently. Of course I have to take into account the possibility that someone has moved the whole file or done linting. This is how it outputs:


> 119 Annie J. Williams
>
>  72 Rachelle A. Ramos
>
>  60 Ruth Earl
>
>  43 Joe Tannenbaum
>
>  33 Tifany D. Saldivar
>
>   5 John A. Davis
>
>   5 J.B. Braswell
>
>   4 Robert Cramer
>
>   3 geoffrey123321
>
>   1 Helen Caldwell

## Remove exclusive testing in my local changes

```
#!/bin/bash

changed_specs() {
  git diff --name-status origin/master | rev | cut -f 1 | rev | grep spec
}

changed_specs | while read spec; do
  echo $spec
  sed -i '' -e 's/fdescribe/describe/g' $spec
  sed -i '' -e 's/describe.only/describe/g' $spec
  sed -i '' -e 's/fit/it/g' $spec
  sed -i '' -e 's/it\.only/it/g' $spec
done
```

I like this script, though it only saves a few seconds when I use it--it saves all the friction of finding the test I set as exclusive in my IDE, scrolling to the right line of code, and either reverting with the UI or manually deleting some text. I can easily add exclusivity patterns for more test frameworks I end up working with. I used `grep` in this script to change only files with "spec" in the name, but that won't suit everyone's codebases.

## Terminal status bar showing branch and CI status

In my iTerm2 window I've been enjoying [`byobu`](http://www.byobu.co/), which is a wrapper for [`tmux`](https://duckduckgo.com/?q=tmux&ia=images&iax=images); I've seen it hiccup a few times but I think behaves well enough. With a little bit of work I wrote some shell scripts that output to the status bar based on the current pane:

- [Current branch name](https://gist.github.com/josephtannenbaum/22aff3968cd58fbbb2c993d8ea097e58) with */+ flags for changes like in VS Code. I also wanted the text to change color based on a diff with the origin branch, but I'm not sure how well that works.
- [Current branch CI status](https://gist.github.com/josephtannenbaum/c36d96ba40fb4dc65441f451d1494b36) -- I used `hub` and GitHub CI integration to show whether the branch's build is passing, in progress, not started, or failed. This is quicker than opening the specific Pull Request or CI website.


That's all I've prepared for now. I had to edit some of these scripts for generality, so I can't guarentee all the code will work out of the box. I'm hoping to write more entries about my development experience in the future!
