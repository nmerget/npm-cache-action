# NPM Cache Action

Simple initialization of node version and npm cache to reuse in one/multiple workflows.

## How to use

`````yaml
---
name: Default Pipeline

on:
  pull_request:
  push:
    branches:
      - "main"

concurrency:
  group: ${{ github.head_ref || github.run_id }}
  cancel-in-progress: true
  
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - name: ⏬ Checkout repo
        uses: actions/checkout@v4

      - name: 🔄 Init Cache Default
        uses: nmerget/npm-cache-action@v1

      - name: 🔨 Build
        run: npm run build

`````

## Configuration

````yaml
...
- name: 🔄 Init Cache Default
  uses: nmerget/npm-cache-action@v1
  with:
    nodeModulesPath: "**/node_modules"
    packageLockPath: "**/package-lock.json"
    nodeVersion: "18"
...
````

## Gotchas

If you have to use the cache in multiple reusable workflows, try to initialize it as the first job and set ``needs`` to the other workflows to speed up the pipeline:

````yaml
name: Default Pipeline

on:
  pull_request:
  push:
    branches:
      - "main"

jobs:
  init:
    uses: ./.github/workflows/init.yml # cache + npm ci

  build:
    uses: ./.github/workflows/build.yml # only cache
    needs: [init]
    secrets: inherit

  lint:
    uses: ./.github/workflows/lint.yml # only cache
    needs: [init]
    secrets: inherit

  e2e:
    uses: ./.github/workflows/e2e.yml # only cache
    needs: [init]
    secrets: inherit

  deploy:
    if: github.ref_name == 'main'
    uses: ./.github/workflows/deploy-gh-pages.yml
    needs: [e2e, build, lint]
````

