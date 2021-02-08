+++
authors = []
date = 2021-02-07T05:00:00Z
excerpt = "Build test site, ignore test site, git merge, find white screen of death on production, git revert, missings commits, merge hotfix, git revert revert, what just happened"
hero = "/images/how-not-to-run-a-pull-request.jpg"
timeToRead = 0
title = "How Not to Git Merge: A Tale of the Evening Headache"

+++
I don't know who else has done it: a hasty merge commit into your production branch. This may build assets that cause a white screen of death on your live site. Perhaps you used a spread operator (`...`) in JavaScript without trans-piling it?

Perhaps, to rectify your horrifying mistake, you performed a `git revert` of your merge commit... In which case, I would ask you: _Why have you done this to yourself?_

(...)

Terms

Remote repository = Source code available from a code hosting service, such as GitHub or an internal server.

Repository = A local copy of source code.

## Don't revert your merge

GitHub makes it very easy to press a green button and perform a messy `git revert abc`, where `abc` is the commit hash of your **merge commit:** the commit which tracks when you merged in **all commits from your feature branch.**

If you revert the merge commit, it will eliminate the document state introduced by all the commits on the incoming branch.

### Reverting merge commits makes the branch messy

Here is an example of why not to revert a merge commit.

In your development branch, you changed all the buttons from yellow to blue (commit #1), make all the headings bold (commit #2), and added a default value to a function (commit #3). This affects 2 different documents in your repository: `app.scss` and `app.js`. 

You're happy with your changes. It runs fine in development. Time to go live! You make a pull request (PR) to add all these changes to your main branch.

You are so confident in your changes, that you don't even preview the test site created by the remote build script. Why did you even bother setting it up?

However, you click on your live site, and see a white screen and a console error complaining about "exports".

### Go back to a previous commit instead

(...)

I remember reading about it but I cannot paraphrase it from memory.

## Fixing a revert of a merge commit

You will have to revert the revert to restore the document states to those introduced by the feature branch.

### Get your commits back

`git log` to get the commit hash.

`git revert abc` where `abc` is the commit hash

## Never do this again

1. Always preview test builds; otherwise, why waste the resources?
2. something something git fetch git merge git checkout