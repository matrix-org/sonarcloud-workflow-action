# sonarcloud-workflow-action
A Github Action for wrapping `SonarSource/sonarcloud-github-action` with all the boilerplate necessary to do the right thing for pull requests from forks in workflow runs.

## Example for running from a `workflow_run` workflow.

```yaml
name: SonarQube
on:
  workflow_run:
    workflows: [ "Tests" ]
    types:
      - completed
concurrency:
  group: ${{ github.workflow }}-${{ github.event.workflow_run.head_branch }}
  cancel-in-progress: true
jobs:
  sonarqube:
    name: SonarQube
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          repository: ${{ github.event.workflow_run.head_repository.full_name }}
          ref: ${{ github.event.workflow_run.head_branch }} # checkout commit that triggered this workflow
          fetch-depth: 0 # Shallow clones should be disabled for a better relevancy of analysis
      - uses: matrix-org/setup-python-poetry@v1.2.2
      - name: SonarCloud Scan
        uses: matrix-org/sonarcloud-workflow-action@v3.2
        with:
          skip_checkout: true
          is_pr: ${{ github.event.workflow_run.event == 'pull_request' }}
          repository: ${{ github.event.workflow_run.head_repository.full_name }}
          version_cmd: "poetry version --short"
          branch: ${{ github.event.workflow_run.head_branch }}
          revision: ${{ github.event.workflow_run.head_sha }}
          token: ${{ secrets.SONAR_TOKEN }}
          coverage_run_id: ${{ github.event.workflow_run.id }}
          coverage_workflow_name: tests.yml
```
