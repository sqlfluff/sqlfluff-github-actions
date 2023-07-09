# tails.com SQLFluff GitHub Workflow

Developed by Alan Cruickshank @ [Tails.com](https://tails.com/gb/careers/), with much of
the base design heavily borrowed from the [surfline workflow](menu_of_workflows/surfline).
Much of the design and documentation is taken directly from that flow and so Redshift users
may find it helpful to combine both approaches.

To add this to your repo, copy the contents of [`sqlfluff_lint_dbt_models.yml`](./sqlfluff_lint_dbt_models.yml)
in this folder into a file named `.github/workflows/sqlfluff_lint_dbt_models.yml`.

## This GitHub Workflow
- Lints any added or modified models in `/data_warehouse`, this assumes
  that your `dbt_project.yml` and all other files for linting are within a this
  subdirectory. To set this up for your project, run a find a replace on `data_warehouse`
  in the `sqlfluff_lint.yml` file.
- Uses `templater = dbt` - this requires a dummy `profiles.yml` and a connection to your
  data warehouse from the workflow. It is assumed that this will be found in a `ci_cd`
  root folder within your project.
- Uses a bare `requirements.txt` file to manage python dependencies. This specifies
  the version of dbt and sqlfluff to use. This likewise found within the `ci_cd` folder
  within the root of your project. Note, that the flow uses a feature in sqlfluff
  which is only present in version `0.10.1` or later (the `--write-output` feature).
  This makes the flow robust to some of the logging issues found while using the dbt
  templater.
- Identifies only the changed files on Pull Requests so that only they are linted.
- Handles connecting to warehouse via VPN if your data warehouse requires it (optional).
  If your warehouse doesn't require connection via VPN, you can delete the `Install OpenVPN`
  and `Connect to VPN` steps from the workflow.
- Annotates failures on the PR, on the line where they occur.

__NOTE:__ This workflow has been tested on Snowflake, but it should be possible to modify
the flow to work with other data warehouses.

## Setup
### `.sqlfluff`

We use the `.sqlfluff` found [here](./.sqlfluff), the flow should also work other compatible
config files (such as `tox.ini`).

### `templater = dbt` & dummy `profiles.yml`

When `sqlfluff` uses `templater = dbt`, it is *actually using the dbt compiler* to compile your
SQL before `sqlfluff` lints it. When the dbt compiler, compiles the SQL in your models
containing macros like `dbt_utils.star()`, it needs to *connect and query the warehouse* to
get information about the table and columns referenced in the marco.

But for `dbt` to connect to a data warehouse, it needs a `profiles.yml`. It's a bad idea to commit
your `profiles.yml` to your repo since it has sensitive DB credentials, so instead create a
*dummy `profiles.yml`* in a `./ci_cd/` folder in your dbt repo, for snowflake this will look like:
```yaml
# -- Snowflake dummy - save this at `./ci_cd/profiles.yml` --
config:
    send_anonymous_usage_stats: False
    use_colors: True

<profile-from-dbt-project.yml>:  # <--- Replace with profile value in your dbt_project.yml!
  target: sqlfluff
  outputs:
    sqlfluff:
      type: snowflake
      account: "{{ env_var('PROFILES_YML_SNOWFLAKE_ACCOUNT') }}"  # <-- Set these environment variables in GitHub Secrets
      user: "{{ env_var('PROFILES_YML_SNOWFLAKE_USER') }}"
      password: "{{ env_var('PROFILES_YML_SNOWFLAKE_PASSWORD') }}"
      role: "{{ env_var('PROFILES_YML_SNOWFLAKE_ROLE') }}"
      database: "{{ env_var('PROFILES_YML_SNOWFLAKE_DATABASE') }}"
      warehouse: "{{ env_var('PROFILES_YML_SNOWFLAKE_WAREHOUSE') }}"
      threads: 1
```
The credentials set in the dummy `profiles.yml` use dbt's `env_var` function to read in environment variables that you need to set in your repo using [GitHub Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets). These environment variables are then [injected into the virtual machine running the GitHub workflow](https://github.com/sqlfluff/sqlfluff-github-actions/blob/66556e8a954fe19c055ab73bccb55a4677f1b2ef/menu_of_workflows/surfline/sqlfluff_lint_dbt_models.yml#L17-L39) like:
```yaml
    # Set environment variables used throughout GitHub workflow
    env:
      DBT_PROFILES_DIR: /home/runner/work/${{ github.event.repository.name }}/${{ github.event.repository.name }}/ci_cd

      # SPECIFY database connection credentials as env vars below.
      # Env var values to be fetched from as GitHub Secrets.
      # HIGHLY recommended you use a unique set of connection credentials for this workflow alone.

      # IF USING REDSHIFT, workflow will use these in dummy profiles.yml (else, ignored)
      PROFILES_YML_HOST: ${{ secrets.PROFILES_YML_HOST }}
      PROFILES_YML_PORT: ${{ secrets.PROFILES_YML_PORT }}
      PROFILES_YML_USER: ${{ secrets.PROFILES_YML_USER }}
      PROFILES_YML_PASS: ${{ secrets.PROFILES_YML_PASS }}
      PROFILES_YML_DBNAME: ${{ secrets.PROFILES_YML_DBNAME }}
      PROFILES_YML_SCHEMA: ${{ secrets.PROFILES_YML_SCHEMA }}
```
The exact secrets you need to configure in your Github repo are listed below.

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

You will set these as credentials as [GitHub Secrets](https://docs.github.com/en/actions/reference/encrypted-secrets),
which are used in the workflow. In some setups you may not require the first two, in that case, you don't need to
set them - and you should comment out the two lines setting them in the workflow yaml file.
- `OVPN_USERNAME`
- `OVPN_PASSWORD`
- `OVPN_CERT` (the contents of your `.ovpn` file).

Now the workflow can connect to your warehouse, again, required when `sqlfluff` uses `templater = dbt`.

### Support for other database types:
- A similar approach for creating dummy `profiles.yml` for other databases (e.g., BigQuery, Presto, etc.) should be possible.
- You will have to modify your dummy `profiles.yml` to match the format found in your local `profiles.yml`, typically found at `~/.dbt/profiles.yml`. Or see the dbt [docs](https://docs.getdbt.com/reference/profiles.yml#!) for connection instructions for other database types.
- Then set the connection credentials as environment variables in your `sqlfluff_lint.yml`.
- Store the environment variables values as GitHub Secrets.
