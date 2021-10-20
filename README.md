# sqlfluff-github-actions
The official resource for SQLFluff related [GitHub Actions and Workflows](https://docs.github.com/en/actions).

## Menu of GitHub Workflows
Workflows are listed below by contributing team with a brief description. To learn more and how to implement each, click the links below.
- [Sunrise Movement](./menu_of_workflows/sunrise_movement)
    - Simple, clean.
    - Start here if you are new to workflows!

- [Surfline](./menu_of_workflows/surfline)
    - Uses `conda` to setup a virtual environment and manage `python`, `dbt`, and `sqlfluff` dependencies.
    - Lints only the _changed_ models in `/models` & `/analysis`.
    - Intended for use with `templater = dbt`.
    - Includes connecting to VPN (if your database checks for this).
    - Checks for a valid connection to your database (required for `templater = dbt`).

- [Drizly](./menu_of_workflows/drizly)
    - Lints modified and changed SQL files in PRs (in your `dbt/models` directory)
    - Annotates failures on the PR, on the line where they occur

## A note on nomenclature
[GitHub Actions](https://docs.github.com/en/actions) is a ___feature___ within GitHub. It allows you to...
> Automate, customize, and execute your software development [___workflows___](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions) right in your repository with GitHub Actions. You can discover, create, and share actions to perform any job you'd like, including CI/CD, and combine actions in a completely customized workflow.

These workflows, defined by YAML files placed in the `.github/workflows/` directory of your repo, are event-driven, meaning that you can run a workflow after a specified event has occurred (e.g., opening a Pull request to the `main` branch).

To add a layer of confusion, the [GitHub Marketplace](https://github.com/marketplace?type=actions) has individual "Actions" (e.g., the [checkout](https://github.com/marketplace/actions/checkout) action) that you can use as steps _inside your workflow(s)_. Using Actions can reduce the code you need to write to define your worflow and accomplish your goal.

Conceptually, you can connect these poorly named pieces like so:
- GitHub Actions (a feature within GitHub allowing you to automate workflows).
    - Workflows (define a list of steps to be executed. Triggered by an event like opening a Pull request).
        - Actions (help you perform individual tasks within workflow, like checking out the pull request branch, without you having to write a bunch of code).

Often you will hear us (and others, including GitHub) use the term "GitHub Action" and "Workflow" interchangeably. If you simply remember that the YAML files are defining a list of steps to be executed when a certain event happens in your repo, you are well on your way to understanding and using the power of workflows in GitHub Actions.

## Resources

See the Github [docs](https://docs.github.com/en/actions) to learn more about GitHub Actions and developing your own worflows.
