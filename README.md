# <img width="24" height="24" src="docs/assets/logo.svg"> Create Pull Request
[![CI](https://github.com/peter-evans/create-pull-request/workflows/CI/badge.svg)](https://github.com/peter-evans/create-pull-request/actions?query=workflow%3ACI)
[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Create%20Pull%20Request-blue.svg?colorA=24292e&colorB=0366d6&style=flat&longCache=true&logo=data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAA4AAAAOCAYAAAAfSC3RAAAABHNCSVQICAgIfAhkiAAAAAlwSFlzAAAM6wAADOsB5dZE0gAAABl0RVh0U29mdHdhcmUAd3d3Lmlua3NjYXBlLm9yZ5vuPBoAAAERSURBVCiRhZG/SsMxFEZPfsVJ61jbxaF0cRQRcRJ9hlYn30IHN/+9iquDCOIsblIrOjqKgy5aKoJQj4O3EEtbPwhJbr6Te28CmdSKeqzeqr0YbfVIrTBKakvtOl5dtTkK+v4HfA9PEyBFCY9AGVgCBLaBp1jPAyfAJ/AAdIEG0dNAiyP7+K1qIfMdonZic6+WJoBJvQlvuwDqcXadUuqPA1NKAlexbRTAIMvMOCjTbMwl1LtI/6KWJ5Q6rT6Ht1MA58AX8Apcqqt5r2qhrgAXQC3CZ6i1+KMd9TRu3MvA3aH/fFPnBodb6oe6HM8+lYHrGdRXW8M9bMZtPXUji69lmf5Cmamq7quNLFZXD9Rq7v0Bpc1o/tp0fisAAAAASUVORK5CYII=)](https://github.com/marketplace/actions/create-pull-request)

A GitHub action to create a pull request for changes to your repository in the actions workspace.

Changes to a repository in the Actions workspace persist between steps in a workflow.
This action is designed to be used in conjunction with other steps that modify or add files to your repository.
The changes will be automatically committed to a new branch and a pull request created.

Create Pull Request action will:

1. Check for repository changes in the Actions workspace. This includes:
   - untracked (new) files
   - tracked (modified) files
   - commits made during the workflow that have not been pushed
2. Commit all changes to a new branch, or update an existing pull request branch.
3. Create a pull request to merge the new branch into the base&mdash;the branch checked out in the workflow.

## Documentation

- [Concepts, guidelines and advanced usage](docs/concepts-guidelines.md)
- [Examples](docs/examples.md)
- [Updating to v3](docs/updating.md)

## Usage

```yml
      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v3
```

You can also pin to a [specific release](https://github.com/peter-evans/create-pull-request/releases) version in the format `@v3.x.x`

### Action inputs

All inputs are **optional**. If not set, sensible defaults will be used.

**Note**: If you want pull requests created by this action to trigger an `on: push` or `on: pull_request` workflow then you cannot use the default `GITHUB_TOKEN`. See the [documentation here](https://github.com/peter-evans/create-pull-request/blob/master/docs/concepts-guidelines.md#triggering-further-workflow-runs) for workarounds.

| Name | Description | Default |
| --- | --- | --- |
| `token` | `GITHUB_TOKEN` or a `repo` scoped [Personal Access Token (PAT)](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line). | `GITHUB_TOKEN` |
| `path` | Relative path under `GITHUB_WORKSPACE` to the repository. | `GITHUB_WORKSPACE` |
| `commit-message` | The message to use when committing changes. | `[create-pull-request] automated change` |
| `committer` | The committer name and email address in the format `Display Name <email@address.com>`. Defaults to the GitHub Actions bot user. | `GitHub <noreply@github.com>` |
| `author` | The author name and email address in the format `Display Name <email@address.com>`. Defaults to the user who triggered the workflow run. | `${{ github.actor }} <${{ github.actor }}@users.noreply.github.com>` |
| `branch` | The pull request branch name. | `create-pull-request/patch` |
| `base` | Sets the pull request base branch. | Defaults to the branch checked out in the workflow. |
| `push-to-fork` | A fork of the checked out parent repository to which the pull request branch will be pushed. e.g. `owner/repo-fork`. The pull request will be created to merge the fork's branch into the parent's base. See [push pull request branches to a fork](https://github.com/peter-evans/create-pull-request/blob/master/docs/concepts-guidelines.md#push-pull-request-branches-to-a-fork) for details. | |
| `title` | The title of the pull request. | `Changes by create-pull-request action` |
| `body` | The body of the pull request. | `Automated changes by [create-pull-request](https://github.com/peter-evans/create-pull-request) GitHub action` |
| `labels` | A comma or newline separated list of labels. | |
| `assignees` | A comma or newline separated list of assignees (GitHub usernames). | |
| `reviewers` | A comma or newline separated list of reviewers (GitHub usernames) to request a review from. | |
| `team-reviewers` | A comma or newline separated list of GitHub teams to request a review from. Note that a `repo` scoped [PAT](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) may be required. See [this issue](https://github.com/peter-evans/create-pull-request/issues/155). | |
| `milestone` | The number of the milestone to associate this pull request with. | |
| `draft` | Create a [draft pull request](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests#draft-pull-requests). | `false` |

### Action outputs

The pull request number is output as a step output.
Note that in order to read the step output the action step must have an id.

```yml
      - name: Create Pull Request
        id: cpr
        uses: peter-evans/create-pull-request@v3
      - name: Check outputs
        run: |
          echo "Pull Request Number - ${{ steps.cpr.outputs.pull-request-number }}"
```

### Checkout

This action expects repositories to be checked out with `actions/checkout@v2`.

If there is some reason you need to use `actions/checkout@v1` the following step can be added to checkout the branch.

```yml
      - uses: actions/checkout@v1
      - run: git checkout "${GITHUB_REF:11}"
```

### Action behaviour

The action creates a pull request that will be continually updated with new changes until it is merged or closed.
Changes are committed and pushed to a fixed-name branch, the name of which can be configured with the `branch` input.
Any subsequent changes will be committed to the *same* branch and reflected in the open pull request.

How the action behaves:

- If there are changes (i.e. a diff exists with the checked out base branch), the changes will be pushed to a new `branch` and a pull request created.
- If there are no changes (i.e. no diff exists with the checked out base branch), no pull request will be created and the action exits silently.
- If a pull request already exists and there are no further changes (i.e. no diff with the current pull request branch) then the action exits silently.
- If a pull request exists and new changes on the base branch make the pull request unnecessary (i.e. there is no longer a diff between the base and pull request branch), the pull request is automatically closed and the branch deleted.

For further details about how the action works and usage guidelines, see [Concepts, guidelines and advanced usage](docs/concepts-guidelines.md).

### Controlling commits

As well as relying on the action to handle uncommitted changes, you can additionally make your own commits before the action runs.

```yml
    steps:
      - uses: actions/checkout@v2
      - name: Create commits
        run: |
          git config user.name 'Peter Evans'
          git config user.email 'peter-evans@users.noreply.github.com'
          date +%s > report.txt
          git commit -am "Modify tracked file during workflow"
          date +%s > new-report.txt
          git add -A
          git commit -m "Add untracked file during workflow"
      - name: Uncommitted change
        run: date +%s > report.txt
      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v3
```

### Ignoring files

If there are files or directories you want to ignore you can simply add them to a `.gitignore` file at the root of your repository. The action will respect this file.

### Create a project card

To create a project card for the pull request, pass the `pull-request-number` step output to [create-or-update-project-card](https://github.com/peter-evans/create-or-update-project-card) action.

```yml
      - name: Create Pull Request
        id: cpr
        uses: peter-evans/create-pull-request@v3

      - name: Create or Update Project Card
        uses: peter-evans/create-or-update-project-card@v1
        with:
          project-name: My project
          column-name: My column
          issue-number: ${{ steps.cpr.outputs.pull-request-number }}
```

## Reference Example

The following workflow is a reference example that sets all the main inputs.

See [examples](docs/examples.md) for more realistic use cases.

```yml
name: Create Pull Request
on: push
jobs:
  createPullRequest:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Make changes to pull request
        run: date +%s > report.txt

      - name: Create Pull Request
        id: cpr
        uses: peter-evans/create-pull-request@v3
        with:
          token: ${{ secrets.PAT }}
          commit-message: Update report
          committer: GitHub <noreply@github.com>
          author: ${{ github.actor }} <${{ github.actor }}@users.noreply.github.com>
          branch: example-patches
          title: '[Example] Update report'
          body: |
            Update report
            - Updated with *today's* date
            - Auto-generated by [create-pull-request][1]

            [1]: https://github.com/peter-evans/create-pull-request
          labels: |
            report
            automated pr
          assignees: peter-evans
          reviewers: peter-evans
          team-reviewers: |
            owners
            maintainers
          milestone: 1
          draft: false

      - name: Check output
        run: |
          echo "Pull Request Number - ${{ steps.cpr.outputs.pull-request-number }}"
```

An example based on the above reference configuration creates pull requests that look like this:

![Pull Request Example](docs/assets/pull-request-example.png)

## License

[MIT](LICENSE)
