---
title: Git Untrack previous commited files
date: 2021-02-21 11:07:15
tags: 
  - git
---

This is something that sometimes bugs me, you keep hitting the .gitinore configuration file but still the file you try to ignore always get's referenced on your changes.

If that happens most probably you've reference it in the past to be track and you need to remove it, so this are the steps.

# Update your .gitignore and untrack the file

This example to untrack `db.json`

1. Change the entry in .gitignore ex: `*.json`

2. Execute the following command to untrack

```
git update-index --assume-unchanged db.json
```

3. Commit the changes


