# Advanced GitHub Actions

In this lesson, I decided to implement GitHub Actions best practices. The objective is to understand how to write maintainable GitHub workflows.

In my first workflow, I wrote maintainable workflows using clear and descriptive names, using comments in my workflows and modularizing common tasks as follows:

```
# This workflow will do a clean installation of node dependencies, cache/restore them, build the source code and run tests across different versions of node
# For more information see: https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-nodejs

name: Node.js CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:

    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18.x, 20.x, 22.x]
        # See supported Node.js release schedule at https://nodejs.org/en/about/releases/

    steps:
    - uses: actions/checkout@v4
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    - run: npm ci
    - name: Run Linter
      run: npx eslint .
    - name: Cache node_modules
      id: cache-node-modules
      uses: actions/cache@v4
      # actions/cache is a GitHub Action to cache dependencies and build outputs to improve workflow execution time.
      with:
        path: node_modules
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    - name: Build and test
      run: |
        npm run build --if-present
        npm test
```

Implementing best practices allowed me to create a more readable workflow.
![node.js.yml](./img/1%20node.js%20success.jpg)

## Caching for Performance Optimization

Caching helps make runs faster by creating a shortcut to dependencies and build outputs.
Here is the workflow snippet for caching:

```
    - name: Cache node_modules
      id: cache-node-modules
      uses: actions/cache@v4
      # actions/cache is a GitHub Action to cache dependencies and build outputs to improve workflow execution time.
      with:
        path: node_modules
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
```

## Security Considerations

I implemented best practices by securing sensitive information in GitHub Secrets, which provides encryption for sensitive information. I also used different files for different workflows as follows:

```
name: Build and push docker image

on:
  workflow_run:
    workflows: ["Node.js CI"]
    types:
      - completed
    branches:
        - main

jobs:
    build:
        runs-on: ubuntu-latest

        if: ${{ github.event.workflow_run.conclusion == 'success' }}
        # The if conditional checks if the workflow_run event was successful before running the job.

        steps:
        - uses: actions/checkout@v2
        -
            name: Login to Docker Hub
            uses: docker/login-action@v3
            with:
              username: ${{ secrets.DOCKERHUB_USERNAME }}
              password: ${{ secrets.DOCKERHUB_TOKEN }}
        
        - name: Set up QEMU
          uses: docker/setup-qemu-action@v3
    
        - name: Set up Docker Buildx
          uses: docker/setup-buildx-action@v3

        - name: Build and push
          uses: docker/build-push-action@v6
          with:
              context: .
              push: true
              tags: distinctugo/advanced_github_actions:latest
```

The above workflow demonstrates the implementation of using GitHub secrets.

Overall, I have learned about the importance of using different workflows, create modular workflows and improving performance via caching.