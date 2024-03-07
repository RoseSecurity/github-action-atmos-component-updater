<!-- markdownlint-disable -->
# github-action-atmos-component-updater <a href="https://cpco.io/homepage?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-component-updater&utm_content="><img align="right" src="https://cloudposse.com/logo-300x69.svg" width="150" /></a>
<a href="https://github.com/cloudposse/github-action-atmos-component-updater/releases/latest"><img src="https://img.shields.io/github/release/cloudposse/github-action-atmos-component-updater.svg" alt="Latest Release"/></a><a href="https://slack.cloudposse.com"><img src="https://slack.cloudposse.com/badge.svg" alt="Slack Community"/></a>
<!-- markdownlint-restore -->

<!--




  ** DO NOT EDIT THIS FILE
  **
  ** This file was automatically generated by the `cloudposse/build-harness`.
  ** 1) Make all changes to `README.yaml`
  ** 2) Run `make init` (you only need to do this once)
  ** 3) Run`make readme` to rebuild this file.
  **
  ** (We maintain HUNDREDS of open source projects. This is how we maintain our sanity.)
  **





-->

This is GitHub Action that can be used as a workflow for automatic updates via Pull Requests in your infrastructure repository according to versions in components sources.


---
> [!NOTE]
> This project is part of Cloud Posse's comprehensive ["SweetOps"](https://cpco.io/homepage?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-component-updater&utm_content=) approach towards DevOps.
> <details><summary><strong>Learn More</strong></summary>
>
> It's 100% Open Source and licensed under the [APACHE2](LICENSE).
>
> </details>

<a href="https://cloudposse.com/readme/header/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-component-updater&utm_content=readme_header_link"><img src="https://cloudposse.com/readme/header/img"/></a>


## Introduction

This is GitHub Action that can be used as a workflow for automatic updates via Pull Requests in your infrastructure repository according to versions in components sources.

### Key Features:

- **Selective Component Processing:** Configure the action to `exclude` or `include` specific components using wildcards, ensuring that only relevant updates are processed.
- **PR Management:** Limit the number of PRs opened at a time, making it easier to manage large-scale updates without overwhelming the system. Automatically close old component-update PRs, so they don't pile up.
- **Material Changes Focus:** Automatically open pull requests only for components with significant changes, skipping minor updates to `component.yaml` files to reduce unnecessary PRs and maintain a streamlined system.
- **Informative PRs:** Link PRs to release notes for new components, providing easy access to relevant information, and use consistent naming for easy tracking.
- **Scheduled Updates:** Run the action on a cron schedule tailored to your organization's needs, ensuring regular and efficient updates.



## Usage



### Prerequisites

This GitHub Action once used in workflow needs permissions to create/update branches and open/close pull requests so the access token needs to be passed.

It can be done in two ways:
- create a dedicated Personal Access Token (PAT)
- use [`GITHUB_TOKEN`](https://docs.github.com/en/actions/security-guides/automatic-token-authentication#about-the-github_token-secret)

If you would like to use `GITHUB_TOKEN` make sure to set permissions in the workflow as follow:

```yaml
permissions:
  contents: write
  pull-requests: write
```

Also, make sure that you set to `Allow GitHub Actions to create and approve pull requests` on both organization and repository levels:
- `https://github.com/organizations/YOUR-ORG/settings/actions`
- `https://github.com/YOUR-ORG/YOUR-REPO/settings/actions`

### Workflow example

```yaml
  name: "atmos-components"

  on:
    workflow_dispatch: {}

    schedule:
      - cron:  '0 8 * * 1'         # Execute every week on Monday at 08:00

  permissions:
    contents: write
    pull-requests: write

  jobs:
    update:
      runs-on: ubuntu-latest
      steps:
        - name: Checkout
          uses: actions/checkout@v3
          with:
            fetch-depth: 0

        - name: Update Atmos Components
          uses: cloudposse/github-action-atmos-component-updater@v2
          with:
            github-access-token: ${{ secrets.GITHUB_TOKEN }}
            max-number-of-prs: 5
            include: |
              aws-*
              eks/*
              bastion
            exclude: aws-sso,aws-saml
```

### Using a Custom Atmos CLI Config Path (`atmos.yaml`)

If your [`atmos.yaml` file](https://atmos.tools/cli/configuration) is not located in the root of the infrastructure repository, you can specify the path to it using [`ATMOS_CLI_CONFIG_PATH` env variable](https://atmos.tools/cli/configuration/#environment-variables).

```yaml
  # ...
  - name: Update Atmos Components
    uses: cloudposse/github-action-atmos-component-updater@v2
    env:
      # Directory containing the `atmos.yaml` file
      ATMOS_CLI_CONFIG_PATH: ${{ github.workspace }}/rootfs/usr/local/etc/atmos/
    with:
      github-access-token: ${{ secrets.GITHUB_TOKEN }}
      max-number-of-prs: 5
```

### Customize Pull Request labels, title and body

```yaml
  # ...
  - name: Update Atmos Components
    uses: cloudposse/github-action-atmos-component-updater@v2
    with:
      github-access-token: ${{ secrets.GITHUB_TOKEN }}
      max-number-of-prs: 5
      pr-title: 'Update Atmos Component \`{{ component_name }}\` to {{ new_version }}'
      pr-body: |
        ## what
        Component \`{{ component_name }}\` was updated [{{ old_version }}]({{ old_version_link }}) → [{{ old_version }}]({{ old_version_link }}).

        ## references
        - [{{ source_name }}]({{ source_link }})
      pr-labels: |
        component-update
        automated
        atmos
```

**IMPORTANT:** The backtick symbols must be escaped in the GitHub Action parameters. This is because GitHub evaluates whatever is in the backticks and it will render as an empty string.

  #### For `title` template these placeholders are available:
  - `component_name`
  - `source_name`
  - `old_version`
  - `new_version`

  #### For `body` template these placeholders are available:
  - `component_name`
  - `source_name`
  - `source_link`
  - `old_version`
  - `new_version`
  - `old_version_link`
  - `new_version_link`
  - `old_component_release_link`
  - `new_component_release_link`

## FAQ

### The action cannot find any components

You may see that the action returns zero components:

```console
[06-03-2024 17:53:47] INFO    Found 0 components
[]
```

This is a common error when the workflow has not checked out the repository before calling this action. Add the following before calling this action.

```yaml
- name: Checkout
  uses: actions/checkout@v3
```

### The action cannot find the component.yaml file

You may see the action fail to find the component.yaml file for a given component as such:

```console
FileNotFoundError: [Errno 2] No such file or directory: 'components/terraform/account-map/component.yaml'
```

This is likely related to a missing or invalid `atmos.yaml` configuration file. Set `ATMOS_CLI_CONFIG_PATH` to the path to your Atmos configuration file.

```yaml
env:
  ATMOS_CLI_CONFIG_PATH: ${{ github.workspace }}/rootfs/usr/local/etc/atmos/
```

### The action does not have permission to create Pull Requests

Your action may fail with the following message:

```console
github.GithubException.GithubException: 403 {"message": "GitHub Actions is not permitted to create or approve pull requests.", "documentation_url": "https://docs.github.com/rest/pulls/pulls#create-a-pull-request"}
```

In order to create Pull Requests in your repository, we need to set the permissions for the workflow:

```yaml
permissions:
  contents: write
  pull-requests: write
```

_And_ you need to allow GitHub Actions to create and approve pulls requests in both the GitHub Organization and Repository:

1. https://github.com/organizations/YOUR-ORG/settings/actions > `Allow GitHub Actions to create and approve pull requests`
2. https://github.com/YOUR-ORG/YOUR-REPO/settings/actions > `Allow GitHub Actions to create and approve pull requests`






<!-- markdownlint-disable -->

## Inputs

| Name | Description | Default | Required |
|------|-------------|---------|----------|
| atmos-version | Atmos version to use for vendoring. Default 'latest' | latest | false |
| dry-run | Skip creation of remote branches and pull requests. Only print list of affected componented into file that is defined in 'outputs.affected-components-file' | false | false |
| exclude | Comma or new line separated list of component names to exclude. For example: 'vpc,eks/\*,rds'. By default no components are excluded. Default '' |  | false |
| github-access-token | GitHub Token used to perform git and GitHub operations | ${{ github.token }} | false |
| include | Comma or new line separated list of component names to include. For example: 'vpc,eks/\*,rds'. By default all components are included. Default '\*' | \* | false |
| infra-terraform-dirs | Comma or new line separated list of terraform directories in infra repo. For example 'components/terraform,components/terraform-old. Default 'components/terraform' | components/terraform | false |
| log-level | Log level for this action. Default 'INFO' | INFO | false |
| max-number-of-prs | Number of PRs to create. Maximum is 10. | 10 | false |
| pr-body-template | A string representing a Jinja2 formatted template to be used as the content of a Pull Request (PR) body. If not set template from `src/templates/pr\_body.j2.md` will be used |  | false |
| pr-labels | Comma or new line separated list of labels that will added on PR creation. Default: `component-update` | component-update | false |
| pr-title-template | A string representing a Jinja2 formatted template to be used as the content of a Pull Request (PR) title. If not, set template from `src/templates/pr\_title.j2.md` will be used |  | false |
| vendoring-enabled | Do not perform 'atmos vendor component-name' on components that wasn't vendored | true | false |


## Outputs

| Name | Description |
|------|-------------|
| affected | The affected components |
| has-affected-stacks | Whether there are affected components |
<!-- markdownlint-restore -->


## Related Projects

Check out these related projects.



## References

For additional context, refer to some of these links.

- [github-actions-workflows](https://github.com/cloudposse/github-actions-workflows) - Reusable workflows for different types of projects
- [example-github-action-release-workflow](https://github.com/cloudposse/example-github-action-release-workflow) - Example application with complicated release workflow


## ✨ Contributing

This project is under active development, and we encourage contributions from our community.
Many thanks to our outstanding contributors:

<a href="https://github.com/cloudposse/github-action-atmos-component-updater/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=cloudposse/github-action-atmos-component-updater&max=24" />
</a>

### 🐛 Bug Reports & Feature Requests

Please use the [issue tracker](https://github.com/cloudposse/github-action-atmos-component-updater/issues) to report any bugs or file feature requests.

### 💻 Developing

If you are interested in being a contributor and want to get involved in developing this project or help out with Cloud Posse's other projects, we would love to hear from you! 
Hit us up in [Slack](https://cpco.io/slack?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-component-updater&utm_content=slack), in the `#cloudposse` channel.

In general, PRs are welcome. We follow the typical "fork-and-pull" Git workflow.
 1. Review our [Code of Conduct](https://github.com/cloudposse/github-action-atmos-component-updater/?tab=coc-ov-file#code-of-conduct) and [Contributor Guidelines](https://github.com/cloudposse/.github/blob/main/CONTRIBUTING.md).
 2. **Fork** the repo on GitHub
 3. **Clone** the project to your own machine
 4. **Commit** changes to your own branch
 5. **Push** your work back up to your fork
 6. Submit a **Pull Request** so that we can review your changes

**NOTE:** Be sure to merge the latest changes from "upstream" before making a pull request!

### 🌎 Slack Community

Join our [Open Source Community](https://cpco.io/slack?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-component-updater&utm_content=slack) on Slack. It's **FREE** for everyone! Our "SweetOps" community is where you get to talk with others who share a similar vision for how to rollout and manage infrastructure. This is the best place to talk shop, ask questions, solicit feedback, and work together as a community to build totally *sweet* infrastructure.

### 📰 Newsletter

Sign up for [our newsletter](https://cpco.io/newsletter?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-component-updater&utm_content=newsletter) and join 3,000+ DevOps engineers, CTOs, and founders who get insider access to the latest DevOps trends, so you can always stay in the know.
Dropped straight into your Inbox every week — and usually a 5-minute read.

### 📆 Office Hours <a href="https://cloudposse.com/office-hours?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-component-updater&utm_content=office_hours"><img src="https://img.cloudposse.com/fit-in/200x200/https://cloudposse.com/wp-content/uploads/2019/08/Powered-by-Zoom.png" align="right" /></a>

[Join us every Wednesday via Zoom](https://cloudposse.com/office-hours?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-component-updater&utm_content=office_hours) for your weekly dose of insider DevOps trends, AWS news and Terraform insights, all sourced from our SweetOps community, plus a _live Q&A_ that you can’t find anywhere else.
It's **FREE** for everyone!

## About

This project is maintained by <a href="https://cpco.io/homepage?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-component-updater&utm_content=">Cloud Posse, LLC</a>.
<a href="https://cpco.io/homepage?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-component-updater&utm_content="><img src="https://cloudposse.com/logo-300x69.svg" align="right" /></a>

We are a [**DevOps Accelerator**](https://cpco.io/commercial-support?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-component-updater&utm_content=commercial_support) for funded startups and enterprises.
Use our ready-to-go terraform architecture blueprints for AWS to get up and running quickly.
We build it with you. You own everything. Your team wins. Plus, we stick around until you succeed.

<a href="https://cpco.io/commercial-support?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-component-updater&utm_content=commercial_support"><img alt="Learn More" src="https://img.shields.io/badge/learn%20more-success.svg?style=for-the-badge"/></a>

*Your team can operate like a pro today.*

Ensure that your team succeeds by using our proven process and turnkey blueprints. Plus, we stick around until you succeed.

<details>
  <summary>📚 <strong>See What's Included</strong></summary>

- **Reference Architecture.** You'll get everything you need from the ground up built using 100% infrastructure as code.
- **Deployment Strategy.** You'll have a battle-tested deployment strategy using GitHub Actions that's automated and repeatable.
- **Site Reliability Engineering.** You'll have total visibility into your apps and microservices.
- **Security Baseline.** You'll have built-in governance with accountability and audit logs for all changes.
- **GitOps.** You'll be able to operate your infrastructure via Pull Requests.
- **Training.** You'll receive hands-on training so your team can operate what we build.
- **Questions.** You'll have a direct line of communication between our teams via a Shared Slack channel.
- **Troubleshooting.** You'll get help to triage when things aren't working.
- **Code Reviews.** You'll receive constructive feedback on Pull Requests.
- **Bug Fixes.** We'll rapidly work with you to fix any bugs in our projects.
</details>

<a href="https://cloudposse.com/readme/commercial-support/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-component-updater&utm_content=readme_commercial_support_link"><img src="https://cloudposse.com/readme/commercial-support/img"/></a>
## License

<a href="https://opensource.org/licenses/Apache-2.0"><img src="https://img.shields.io/badge/License-Apache%202.0-blue.svg?style=for-the-badge" alt="License"></a>

<details>
<summary>Preamble to the Apache License, Version 2.0</summary>
<br/>
<br/>

Complete license is available in the [`LICENSE`](LICENSE) file.

```text
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
```
</details>

## Trademarks

All other trademarks referenced herein are the property of their respective owners.
---
Copyright © 2017-2024 [Cloud Posse, LLC](https://cpco.io/copyright)


<a href="https://cloudposse.com/readme/footer/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-component-updater&utm_content=readme_footer_link"><img alt="README footer" src="https://cloudposse.com/readme/footer/img"/></a>

<img alt="Beacon" width="0" src="https://ga-beacon.cloudposse.com/UA-76589703-4/cloudposse/github-action-atmos-component-updater?pixel&cs=github&cm=readme&an=github-action-atmos-component-updater"/>
