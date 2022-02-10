# Jetblue's Linting On Comment Workflow
Created by @reidwil

![](https://github.com/reidwil-jb/workflows/blob/main/.github/workflows/lintworkflow.png)

-------------
  
## How To:

0. Copy the contents of the `lint.yml` into `$REPO_ROOT/.github/workflows/` directory of your github project.

1. Create a PR with sql code

2. Comment on the PR with `/lint` (the specific trigger can be adjusted [here](https://github.com/reidwil-jb/workflows/blob/main/.github/workflows/lint.yml#L21))

3. Sit back and wait for github actions to return the results!


## Info

This workflow is used in a github pull request as a comment. The lint _only_ will run if you "trigger" it as a comment inside of a pull request.

The workflow uses `diff-quality` to find **only new** files from a pull request relative to the default branch on the repository (main, master, development). It will lint those files and return a tidy comment on the pull request.

This process currently is dependent on:

   1. action/checkout@v2

   2. [SQLfluff/Diff-quality](https://docs.sqlfluff.com/en/stable/production.html#adding-diff-quality-to-your-builds) for linting

   3. [Khan's pull request trigger](https://github.com/Khan/pull-request-comment-trigger)

   4. [octions's post comments](https://github.com/maxkomarychev/octions#read-input-from-file)
