# Surfline SQLFluff GitHub Workflow

Developed by Greg Clunies @ [Surfline](https://www.surfline.com/).

To add this to your repo, copy the contents of [`sqlfluff_lint_dbt_models.yml`](./sqlfluff_lint_dbt_models.yml) in this folder into a file named `.github/workflows/sqlfluff_lint_dbt_models.yml`.

## This GitHub Workflow
- Lints any added or modified models in `/models`
- Uses [`conda`](https://docs.conda.io/en/latest/miniconda.html) to setup a virtual environment and manage `python`, `dbt`, and `sqlfluff` dependencies. An example [`environment.yml`](./environment.yml) can be found in this folder. You can modify this workflow to handle your dependencies as you wish (e.g., `pip`, `virtualenv`, `poetry`, etc.).
- Uses `templater = dbt` - this requires a dummy `profiles.yml` and a connection to your data warehouse from the workflow.
- Handles connecting to warehouse via VPN if your data warehouse requires it (optional). If your warehouse doesn't require connection via VPN, you can delete the `Install OpenVPN` and `Connect to VPN` steps from the workflow.
- Annotates failures on the PR, on the line where they occur.



__NOTE:__ This workflow has been tested on Redshift. Config for Snowflake is included in this repo, but has not been tested.

## Setup
### `.sqlfluff`
We use the `.sqlfluff` found [here](./.sqlfluff).

### `templater = dbt` & dummy `profiles.yml`

When `sqlfluff` uses `templater = dbt`, it is *actually using the dbt compiler* to compile your SQL before `sqlfluff` lints it. When the dbt compiler, uh ... compiles... the SQL in your models containing macros like `dbt_utils.star()`, it needs to *connect and query the warehouse* to get information about the table and columns referenced in the marco.

But for `dbt` to connect to a data warehouse, it needs a `profiles.yml`. It's a bad idea to commit your `profiles.yml` to your repo since it has sensitive DB credentials, so instead create a *dummy `profiles.yml`* in a `./ci_cd/` folder in your dbt repo, for [Redshift](./profiles_redshift.yml) ([Snowflake version](./profiles_snowflake.yml)) this will look like:
```yaml
# -- Redshift dummy - save this at `./ci_cd/profiles.yml` --
config:
    send_anonymous_usage_stats: False
    use_colors: True

<profile-from-dbt-project.yml>:  # <--- Replace with profile value in your dbt_project.yml!
  target: sqlfluff
  outputs:
    sqlfluff:
      type: redshift
      host: "{{ env_var('PROFILES_YML_HOST') }}"       # <-- Set these environment variables in GitHub Secrets
      port: "{{ env_var('PROFILES_YML_PORT') | int }}"
      user: "{{ env_var('PROFILES_YML_USER') }}"
      pass: "{{ env_var('PROFILES_YML_PASS') }}"
      dbname: "{{ env_var('PROFILES_YML_DBNAME') }}"
      schema: "{{ env_var('PROFILES_YML_SCHEMA') }}"
      threads: 1
```
The credentials set in the dummy `profiles.yml` use dbt's `env_var` function to read in environmment variables that you need to set in your repo using [GitHub Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets). These evironment variables are then [injected into the virtual machine running the GitHub workflow](https://github.com/sqlfluff/sqlfluff-github-actions/blob/66556e8a954fe19c055ab73bccb55a4677f1b2ef/menu_of_workflows/surfline/sqlfluff_lint_dbt_models.yml#L17-L39) like:
```yaml
    # Set environment variables used throughout GitHub workflow
    env:
      DBT_PROFILES_DIR: /home/runner/work/${{ github.event.repository.name }}/${{ github.event.repository.name }}/ci_cd

      # SPECIFY database connection credentials as env vars below.
      # Env var values to be fetched from as GitHub Secrets.
      # HIGHLY recommended you use a unique set of connection credentials for this worklfow alone.

      # IF USING REDSHIFT, workflow will use these in dummy profiles.yml (else, ignored)
      PROFILES_YML_HOST: ${{ secrets.PROFILES_YML_HOST }}
      PROFILES_YML_PORT: ${{ secrets.PROFILES_YML_PORT }}
      PROFILES_YML_USER: ${{ secrets.PROFILES_YML_USER }}
      PROFILES_YML_PASS: ${{ secrets.PROFILES_YML_PASS }}
      PROFILES_YML_DBNAME: ${{ secrets.PROFILES_YML_DBNAME }}
      PROFILES_YML_SCHEMA: ${{ secrets.PROFILES_YML_SCHEMA }}
```
The exact secrets you need to configure in your Github repo are listed below.

**Redshift:**
- `PROFILES_YML_HOST`
- `PROFILES_YML_PORT`
- `PROFILES_YML_USER`
- `PROFILES_YML_PASS`
- `PROFILES_YML_DBNAME`
- `PROFILES_YML_SCHEMA`

**Snowflake:**
- `PROFILES_YML_SNOWFLAKE_ACCOUNT`
- `PROFILES_YML_SNOWFLAKE_USER`
- `PROFILES_YML_SNOWFLAKE_PASSWORD`
- `PROFILES_YML_SNOWFLAKE_ROLE`
- `PROFILES_YML_SNOWFLAKE_DATABASE`
- `PROFILES_YML_SNOWFLAKE_WAREHOUSE`

Now that the DB credentials are set in the dummy `profiles.yml`, the when `sqlfluff` uses `templater = dbt, it can connect to the warehouse!

### Connecting to warehouse via VPN
If your warehouse requires connections to be made via a VPN, you will need the following credentials (ask your IT team about creating these):
- username *(strongly recommend creating user just for this GitHub workflow)*
- password
- `vpn_config.ovpn`

You will set these as credentials as [GitHub Secrets](https://docs.github.com/en/actions/reference/encrypted-secrets), which are used in the workflow.
- `OVPN_USERNAME`
- `OVPN_PASSWORD`
- `VPN_FILE` (the contents of your `.ovpn` file).

Now the workflow can connect to your warehouse, again, required when `sqlfluff` uses `templater = dbt`.

### Support for other database types:
- A similar approach for creating dummy `profiles.yml` for other databases (e.g., BigQuery, Presto, etc.) should be possible.
- You will have to modify your dummy `profiles.yml` to match the format found in your local `profiles.yml`, typically found at `~/.dbt/profiles.yml`. Or see the dbt [docs](https://docs.getdbt.com/reference/profiles.yml#!) for connection instructions for other database types.
- Then set the connection credentials as environment variables in your `sqlfluff_lint_dbt_models.yml`.
- Store the environment variables values as GitHub Secrets.
