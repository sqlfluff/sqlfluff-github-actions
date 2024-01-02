# honeycomb.io SQLFluff GitHub Workflow

This workflow is a massively simplified and updated version of the [surfline workflow](menu_of_workflows/surfline).

To add this to your repo, copy the contents of [`sqlfluff_lint_dbt_models.yml`](./sqlfluff_lint_dbt_models.yml)
in this folder into a file named `.github/workflows/sqlfluff_lint_dbt_models.yml`.

## This GitHub Workflow
- Lints any added or modified models in `/models`, with one exclusion example
- Uses a bare `requirements.txt` file to manage python dependencies. The contents of it are simply `sqlfluff==2.3.5`
- Identifies only the changed files on Pull Requests so that only they are linted.
- Annotates failures on the PR, on the line where they occur, using SQLFluff's native annotation mode.

__NOTE:__ This workflow has been tested on Snowflake, but it should be possible to modify the flow to work with other data warehouses.

## Setup
### `.sqlfluff`

The `.sqlfluff` found [here](./.sqlfluff) skips the dbt templater in favor of the much simpler and faster jinja templater. A few jinja contexts are supplied as an example, and these are typically enough to keep the templater happy.
