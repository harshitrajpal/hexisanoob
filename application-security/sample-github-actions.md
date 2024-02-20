---
description: This Github Actions "hello.yml" would run make file when an update is pushed
---

# Sample Github Actions

In accordance to CI/CD concepts, regression test is done. On git push, following YAML file would be run if placed in repository under .github/workflows and configured to run on push under github actions.

## hello.yml

{% code lineNumbers="true" %}
```
   name: Assignment 1 Part 2

   on:
     push:
       branches: [ master ]

   jobs:
    build:
      runs-on: ubuntu-latest
      steps:
      - name: checkout repo
        uses: actions/checkout@v3
      - name: build application
        run: make
      - name: build application (test cases)
        run: make test
```
{% endcode %}
