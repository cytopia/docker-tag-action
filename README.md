# Docker tag action

[![GitHub release](https://img.shields.io/github/release/cytopia/docker-tag-action.svg)](https://github.com/cytopia/docker-tag-action/releases/latest)
[![GitHub marketplace](https://img.shields.io/badge/marketplace-cytopia--docker--tag--action-blue?logo=github)](https://github.com/marketplace/actions/docker-tag-action)
[![test](https://github.com/cytopia/docker-tag-action/actions/workflows/test.yml/badge.svg)](https://github.com/cytopia/docker-tag-action/actions/workflows/test.yml)
[![](https://img.shields.io/badge/github-cytopia%2Fdocker--tag-red.svg)](https://github.com/cytopia/docker-tag-action "github.com/cytopia/docker-tag-action")


This action determines the Docker tag based on what git branch, git tag or git commit you are on.


## Determination

The Docker tag output (`docker-tag`) of this action is determined in its default configuration as shown by the following table:

| Git state          | Resulting Docker tag |
|--------------------|----------------------|
| On branch `master` | `latest`             |
| On another branch  | `<branch-name>`      |
| On a tag           | `<tag-name>`         |
| Detached commit    | `<git-commit-hash>`  |


## Inputs

The following inputs can be used to alter the Docker tag name determination:

| Input                          | Required | Default  | Description                                                                     |
|--------------------------------|----------|----------|---------------------------------------------------------------------------------|
| `latest_git_branch`            | No       | `master` | Optionally change the git branch, which determines the the Docker tag `latest`. |
| `latest_docker_tag_name`       | No       | `latest` | Optionally specify an alternative Docker tag name for `latest`.                 |
| `non_latest_docker_tag_prefix` | No       | ``       | Optionally add a prefix to all non-latest docker tags.                          |




## Outputs

| Output       | Description |
|--------------|-------------|
| `docker-tag` | The determined Docker tag name. |


## Usage

### Basic

The following Docker images will be pushed:

| Git state                                            | Docker images             |
|------------------------------------------------------|---------------------------|
| On branch `master`                                   | `cytopia/php:latest`      |
| On branch `release-0.1`                              | `cytopia/php:release-0.1` |
| On tag `v1.0.0`                                      | `cytopia/php:v1.0.0`      |
| On commit `1aaf12a472ac794d00b75b826db4d223c3ef2d96` | `cytopia/php:1aaf12a`     |


```yaml
on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    name: Deploy docker image
    steps:

      - name: Checkout repository
        uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - name: Set Docker tag
        id: tag
        uses: cytopia/docker-tag@v0.4

      - name: build
        run: |
          docker build -t cytopia/php:${{ steps.tag.outputs.docker-tag }} .

      - name: push
        run: |
          docker push cytopia/php:${{ steps.tag.outputs.docker-tag }}
```

### Matrix

The following Docker images will be pushed:

| Git state                                            | Docker images                                                   |
|------------------------------------------------------|-----------------------------------------------------------------|
| On branch `master`                                   | `cytopia/php:8.0`<br/>`cytopia/php:8.1`                         |
| On branch `release-0.1`                              | `cytopia/php:8.0-release-0.1`<br/>`cytopia/php:8.1-release-0.1` |
| On tag `v1.0.0`                                      | `cytopia/php:8.0-v1.0.0`<br/>`cytopia/php:8.1-v1.0.0`           |
| On commit `1aaf12a472ac794d00b75b826db4d223c3ef2d96` | `cytopia/php:8.0-1aaf12a`<br/>`cytopia/php:8.1-1aaf12a`         |

```yaml
on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    name: Deploy docker image
    strategy:
      fail-fast: false
      matrix:
        version:
          - '8.0'
          - '8.1'
    steps:

      - name: Checkout repository
        uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - name: Set Docker tag
        id: tag
        uses: cytopia/docker-tag@v0.4
        with:
          latest_git_branch: master
          latest_docker_tag_name: ${{ matrix.version }}
          non_latest_docker_tag_prefix: "${{ matrix.version }}-"

      - name: build
        run: |
          docker build -t cytopia/php:${{ steps.tag.outputs.docker-tag }} .

      - name: push
        run: |
          docker push cytopia/php:${{ steps.tag.outputs.docker-tag }}
```


## Keep up-to-date with GitHub Dependabot

Since [Dependabot](https://docs.github.com/en/github/administering-a-repository/keeping-your-actions-up-to-date-with-github-dependabot) has [native GitHub Actions support](https://docs.github.com/en/github/administering-a-repository/configuration-options-for-dependency-updates#package-ecosystem), to enable it on your GitHub repo all you need to do is add the `.github/dependabot.yml` file:

```yml
version: 2
updates:
  # Maintain dependencies for GitHub Actions
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "daily"
```


## License

**[MIT License](LICENSE)**

Copyright (c) 2022 [cytopia](https://github.com/cytopia)
