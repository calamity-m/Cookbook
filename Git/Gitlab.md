

# CICD


## Basic Printenv Pipeline Job

```yaml
stages:
  - env

env:
  stage: env
  image: busybox:latest
  script:
    - printenv
```

## Include a component

```yaml
# Component to include/import
- component: component-repo/component@brancxh
    inputs:
      # Inputs to override/set in component
      key: "value"
    rules:
      # Example rule that only causes this component
      # to be used on default branches
      - if: $CI_COMMIT_BRANCH != $CI_DEFAULT_BRANCH
```

