# ghast

GHAST (GitHub Actions Static Analysis Tool) is a tool to analyze the security posture of your GitHub Actions.

![ghast-stdout](/images/ghast-stdout.png)

### Installation

> Make sure you have `$HOME/.local/bin` in your PATH

```
pip install ghast-scanner
```

### Usage

```
ghast --file action.yml
ghast -d directory-with-actions/ --verbose
ghast --file action.yml --ignore-warnings
ghast --list-checks
```

### Use `ghast` in Your GitHub Workflows

#### Example

```yaml
name: 'RunGhast'
on:
  push:
    branches:
      - main
      - dev
    paths:
      - '.github/workflows/**'
jobs:
  RunGhast:
    runs-on: ubuntu-latest
    steps:
    - name: "Checkout repo"
      uses: actions/checkout@96f53100ba2a5449eb71d2e6604bbcd94b9449b5 # v3.5.3
    - name: "Run Ghast"
      uses: "bin3xish477/ghast@ee733379e314d44f1a960a70339ee5e5d19e404d"
      with:
        dir: "./actions/"
        verbose: true
        no-summary: true
        ignore-checks: 'check_for_inline_script check_for_cache_action_usage'
```

### Checks Performed by `ghast`

1. Name: `check_for_3p_actions_without_hash`, Level: `FAIL`

    - This check identifies any third-party GitHub Actions in use that have been referenced via a version number such as `v1.1` instead of commit SHA haah. Using a hash can help mitigate supply chain threats in a scenario where a threat actor has compromised the source repository where the 3P action lives.

2. Name: `check_for_allow_unsecure_commands`, Level: `FAIL`

    - This check looks for the usage of environment variable called `ACTIONS_ALLOW_UNSECURE_COMMANDS` which allows for an Action to get access to dangerous commands (`get-env`, `add-path`) which can lead to code injection and credential thefts opportunities.

3. Name: `check_for_cache_action_usage`, Level: `WARN`

    - This check finds any usage of GitHub's caching Action (`actions/cache`) which may result in sensitive information disclosure or cache poisoning.

4. Name: `check_for_dangerous_write_permissions`, Level: `FAIL`

    - This check looks for write permissions granted to potentially dangerous scopes such as the `contents` scope which may allow an adversary write code into the target repository if they're able to compromise the workflow. It's also looks for usage of the `write-all` which gives the action complete write access to all scopes.

5. Name: `check_for_inline_script`, Level: `WARN`

    - This check simply warns that you're using an inline script instead of GitHub Action. Inline scripts are susceptible to script injection attacks (another check covered by `ghast`). It is recommended to write an action and pass any required context values as inputs to that action which removes script injection vector because action input are properly treated as arguments and are not evaluated as part of a script.

6. Name: `check_for_pull_request_target`, Level: `FAIL`

    - This check looks for the usage of the dangerous event trigger `pull_request_target` which allows workflow executions to run in the context of the repository that defines the workflow, not the repository that the pull request originated from, potentially allowing a threat actor to gain access to a repositories sensitive secrets!

7. Name: `check_for_script_injection`, Level: `FAIL`

    - This check looks for the most commonly known security risk to GitHub Action - script injection. Script injection occurs when an action directly includes (using the `${{ ... }}` syntax) a GitHub Context variable(s) in an inline script that can be controlled by an untrusted actor, resulting in command execution in the interpreted shell. These user-controllable parameters should be passed into an inline script as environment variables.

8. Name: `check_for_self_hosted_runners`, Level: `WARN` 

    - This checks attempts to identify the usage of self-hosted runners. Self-hosted runners are dangerous because if the Action is compromised it may allow a threat actor to gain access to on premise environment or establish persistence mechanisms on a server you own/rent.

9. Name: `check_for_aws_configure_credentials_non_oidc`, Level: `WARN`

    - This checks looks for the usage of AWS's `aws-actions/configure-aws-credentials` action and attempts to identify non-OIDC authentication parameters. Non-OIDC authentication types are less secure than OIDC because they require the creation of long-term credentials which can be compromised, however, OIDC tokens are short-lived and are usually scoped to only the permissions that are essential to a workflow and thus help reduce the attack surface.

10. Name: `check_for_create_or_approve_pull_request`, Level: `WARN`

    - This check looks for Action that have logic related to creating or improving pull requests. Creating or approving pull requests via automation poses a security risk if sufficient controls aren't in place to protect against malicious code being merged into a repository.

11. Name: `check_for_remote_script`, Level: `WARN`

    - This check looks for a URL in an inline script of a GitHub Action which usually signals the inclusion of a remote script which can be dangerous.
### References

- [Security hardening for GitHub Actions](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
