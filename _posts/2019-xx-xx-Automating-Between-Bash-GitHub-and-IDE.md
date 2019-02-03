---
title:  "Automating Between Bash, GitHub, and IDE"
---

As a follow-up to my last scripts [post](https://josephtannenbaum.github.io/Automating-Parts-of-My-Work/), I want to share some recent Bash tools I've cobbled together. All of the scripts are meant to reduce repetitive file-navigating steps with the mouse and usual Bash utilities. The commands below really just glue together powerful tools like `grep`, `find`, and `hub`, pre-set with my own defaults. When GitHub offers more integration with the terminal and/or IDEs like VS Code, these should become redundant.

I often know a HTML file exists in my codebase with a string like "Edit default card" in it, the only file with that string. If I want to open that file in GitHub to link it to a friend, what are the series of steps I need to take? I wanted to need just one step: running `open_in_github $(find_this_file "Edit default card")` should just... find that file in the CWD, and open a new browser tab with its GitHub page.


```
# capture argument 1 or clipboard
# I use this in the below tools when I know I want to be passing a text selection in a website/Slack/VS Code into the command
INPUTSTRING=$([[ $# -eq 0 ]] && pbpaste || echo $1)
```

```bash
# pro
# Open a PR by its number, URL or branch in VS Code. Examples:
#   pro 1480
#   pro https://github.com/josephtannenbaum/josephtannenbaum.github.io/pull/2
#   pro jt-hello-world
PRURL=$([[ $# -eq 0 ]] && pbpaste || echo $1)
PRNUM=$(echo $PRURL | tr '/' '\n' | tail -n 1)
re='^[0-9]+$'
if ! [[ $PRNUM =~ $re ]] ; then
    # treat input as branch name
    hub checkout $PRNUM
else
    # treat input as PR number
    hub pr checkout $PRNUM
fi
git diff --name-only origin/master... | head -n 12 | xargs code
```

```bash
# Open a new tab in your browser with the GitHub page of the input file (current branch)
function opengh() {
    GHURL=$(hub browse -u)
    BRANCH=$(git rev-parse --abbrev-ref HEAD)
    if [[ "$BRANCH" = "master" ]]; then
        GHURL="$GHURL/blob/master"
    fi
    open "$GHURL/$1"
}
```

```bash
# findy
# find a file by its filename and output the most recently git-touched file
find . -name "$1" ${@:2} | sed "s|^\./||" | xargs -L1 -I{} git log -1 --format="%ai {}" -- {} | sort | tail -n 1 | rev | cut -f 1 -d ' ' | rev
```

```bash
# grind
# grep a pattern recursively and output the most recently git-touched file
FOUNDFILES=$(grep --exclude-dir=node_modules --exclude-dir=client/vendor -niRIHl "$1" *)
echo "$FOUNDFILES" | xargs -L1 -I{} git log -1 --format="%ai {}" -- {} | sort | tail -n 1 | rev | cut -f 1 -d ' ' | rev
```

```bash
# of:
# open findy result in vim

P=$([[ $# -eq 0 ]] && pbpaste || echo $1)

F=$(findy "$P")
if [ -z "$F" ]; then
    echo "No file found for input"
else
    vim $F
fi
```

I also have `og` which opens the result of `grind` in Vim.

```bash
# cg
# VS Code from Grind
# code [[ file found by grind ]]

P=$([[ $# -eq 0 ]] && pbpaste || echo $1)

F=$(grind "$P")
if [ -z "$F" ]; then
    echo "No file found for input"
else
    code $F
fi
```

