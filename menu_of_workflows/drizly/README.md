# Drizly's SQLFluff GitHub Workflow

Developed by Emily Hawkins @ Drizly

To add this to your repo, copy the contents of `lint_sqlfluff.yml` in this folder into a file named `.github/workflows/lint_sqlfluff.yml`.

## Assumptions
1. Only SQL files in your `dbt/models` directory are linted
2. You are on the latest version of SQLFluff (at least 0.6.1) with the format `github-annotations` available.

## Usage notes
This action will take any changed files (modifed or added) in a PR and lint them using your SQLFluff configuration.
The output of `sqlfluff lint` is saved into an `annotations.json` file, which is then read by the [yuzutech/annotations-action](https://github.com/marketplace/actions/annotate-action), which will annotate the failures on your PR, on the line where they occur. We have `annotation-level` set to `failure` but it can also be changed to `notice` or `warning`. You can find SQLFluff documentation on the format and annotation-level argument [here](https://docs.sqlfluff.com/en/stable/cli.html#cmdoption-sqlfluff-lint-f).
