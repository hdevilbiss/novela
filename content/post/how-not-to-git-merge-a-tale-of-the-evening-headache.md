+++
authors = []
date = 2021-02-07T05:00:00Z
excerpt = "Build test site, ignore test site, git merge, find white screen of death on production, git revert, missings commits, merge hotfix, git revert revert, what just happened"
hero = ""
timeToRead = 0
title = "How Not to Git Merge: A Tale of the Evening Headache"

+++
I don't know who else has done it: a hasty merge commit into your production branch. This may build assets that cause a white screen of death on your live site. Perhaps you used a spread operator (`...`) in JavaScript without trans-piling it?

Perhaps, to rectify your horrifying mistake, you performed a `git revert` of your merge commit... In which case, I would ask you: _Why have you done this to yourself?_

(...)

## Fixing a revert of a merge commit

### Get your commits back