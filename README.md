# Docker tag action

## Usage

```yaml
on: [push]

jobs:
  docker_tag:
    runs-on: ubuntu-latest
    name: Print Docker tag
    steps:

      - name: Checkout repository
        uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - name: Set Docker tag
        id: tag
        uses: cytopia/docker-tags@v0.2

      - name: build
        run: |
          docker build -t image_name:${{ steps.tag.outputs.docker-tag }} .

      - name: push
        run: |
          docker push image_name:${{ steps.tag.outputs.docker-tag }}
```
