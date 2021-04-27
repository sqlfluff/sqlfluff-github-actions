# Surfline SQLFluff GitHub Workflow

Developed by Greg Clunies @ [Surfline](https://www.surfline.com/).

To add this to your repo, copy the contents of [`sqlfluff_lint_dbt_models.yml`](./sqlfluff_lint_dbt_models.yml) in this folder into a file named `.github/workflows/sqlfluff_lint_dbt_models.yml`.

## Assumptions
1. You are using `templater = dbt` when running `sqlfluff`.
1. You use [`conda`](https://docs.conda.io/projects/conda/en/latest/index.html) to manage your virtual environments, your versions of `dbt` and `sqlfluff`, and any other dependencies. An example [`environment.yml`](./environment.yml) can be found in this folder.
1. You need to connect to your database via VPN. If not, you can remove the VPN steps from the workflow. Otherwise:
    - We assume the VPN uses the OpenVPN protocol.
    - You will need credentials (username, password, and `.ovpn` file) for the VPN. It's a good idea to create new credentials to be used by this workflow only.

## Implementation
__NOTE:__ This workflow has been tested  on Redshift. Config for Snowflake is included, but not tested.
#### Create a dummy `profiles.yml`
For `sqlfluff` to use `templater = dbt`, dbt needs to connect to your database. _Why?_
- When SQLFLuff runs, it compiles your SQL _using the dbt compiler_.
- In order to compile SQL with macros like `dbt_utils.star()`, dbt has to query the database in order to write the corresponding SQL.

As a result of this compilcation:
- Depending on your database type, copy the contents of [`profiles_redshift.yml`](./profiles.yml) or [`profiles_snowflake.yml`](./profiles_snowlfake.yml) in this folder into a file named `./ci_cd/profiles.yml` in your repo. This is your 'dummy' `profile.yml`.
- Replace `<profile-from-dbt-project-dot-yml>` in your dummy `profiles.yml` with the target name specified in your dbt project's `dbt_project.yml` file.

__NOTE:__ For connecting to other database types, see notes at end of the __Secret Management__ section below.

#### Secret Management
You will need to set the following credentials as [GitHub Secrets](https://docs.github.com/en/actions/reference/encrypted-secrets) to be used in this workflow. We _strongly recommend_ creating credentials to be used by this workflow alone.

- `OVPN_USERNAME`
- `OVPN_PASSWORD`
- `VPN_FILE` (the contents of your `.ovpn` file).

If using Redshift:
- `PROFILES_YML_HOST`
- `PROFILES_YML_PORT`
- `PROFILES_YML_USER`
- `PROFILES_YML_PASS`
- `PROFILES_YML_DBNAME`
- `PROFILES_YML_SCHEMA`

If using Snowflake:
- `PROFILES_YML_SNOWFLAKE_ACCOUNT`
- `PROFILES_YML_SNOWFLAKE_USER`
- `PROFILES_YML_SNOWFLAKE_PASSWORD`
- `PROFILES_YML_SNOWFLAKE_ROLE`
- `PROFILES_YML_SNOWFLAKE_DATABASE`
- `PROFILES_YML_SNOWFLAKE_WAREHOUSE`

If using other database type:
- A similar approach for creating dummy `profiles.yml` for other databases (e.g., BigQuery, Presto, etc.) should be possible.
- You will have to modify your dummy `profiles.yml` to match the format found in your local `profiles.yml`, typically found at `~/.dbt/profiles.yml`. Or see the dbt [docs](https://docs.getdbt.com/reference/profiles.yml#!) for connection instructions for other database types.
- Then set the connection credentials as environment variables in your `sqlfluff_lint_dbt_models.yml`.
- Store the environment variables values as GitHub Secrets.
