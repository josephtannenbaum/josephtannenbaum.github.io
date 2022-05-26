---
title:  "VS Code: Switch to main branch with one keybinding"
---

<img src="{{ site.baseurl }}/images/Screen Shot 2022-05-26 at 11.28.36 AM.png" alt="VS Code" />

`gitmm` script (remember to chmod +x)

```bash
#!/bin/bash
set -e

git_main_branch () {
        command git rev-parse --git-dir &> /dev/null || return
        local ref
        for ref in refs/{heads,remotes/{origin,upstream}}/{main,trunk}
        do
                if command git show-ref -q --verify $ref
                then
                        echo ${ref:t}
                        return
                fi
        done
        echo master
}

git fetch
git checkout $(git_main_branch) -q
git pull
```

`tasks.json`
```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "gitmm",
      "type": "shell",
      "command": "/Users/joet/bin/gitmm",
      "isBackground": true,
      "presentation": {
        "reveal": "never",
        "revealProblems": "onProblem",
        "showReuseMessage": false
      },
      "problemMatcher": []
    }
  ]
}
```

`keybindings.json`
```
// ...
  {
    "key": "cmd+k m",
    "command": "workbench.action.tasks.runTask",
    "args": "gitmm"
  },
// ...
```
