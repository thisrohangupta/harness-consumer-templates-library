# Kubernetes Canary Deployment Stage Template

## Introduction

- Harness provides out of the box templates for common deployment capabilities
- This template is used to perform Kubernetes Deployments in a [Canary deployment](https://developer.harness.io/docs/continuous-delivery/cd-deployments-category/deployment-concepts#canary-deployment) style
- You can copy this YAML into your Template Library Builder in the Harness and save it, or you can upload this file to a Git Repo, and import it into the Harness Template Library

## Canary Deployment Stage Template YAML

### Inputs

Users will need to provide a set of inputs for the template to work

1. Service
2. Environment
  a. Infrastructure Definition
  b. Connector to the Kubernetes Infrastructure

```YAML
template:
  name: Canary Deployment
  identifier: Canary_Deployment
  versionLabel: "1.0"
  type: Stage
  description: Canary Deployment Sample Template
  tags: {}
  spec:
    type: Deployment
    spec:
      deploymentType: Kubernetes
      service:
        serviceRef: <+input>
        serviceInputs: <+input>
      environment:
        environmentRef: <+input>
        deployToAll: false
        environmentInputs: <+input>
        infrastructureDefinitions: <+input>
      execution:
        steps:
          - stepGroup:
              name: Canary Deployment
              identifier: canaryDepoyment
              steps:
                - step:
                    name: Canary Deployment
                    identifier: canaryDeployment
                    type: K8sCanaryDeploy
                    timeout: 10m
                    spec:
                      instanceSelection:
                        type: Count
                        spec:
                          count: 1
                      skipDryRun: false
                - step:
                    name: Canary Delete
                    identifier: canaryDelete
                    type: K8sCanaryDelete
                    timeout: 10m
                    spec: {}
          - stepGroup:
              name: Primary Deployment
              identifier: primaryDepoyment
              steps:
                - step:
                    name: Rolling Deployment
                    identifier: rollingDeployment
                    type: K8sRollingDeploy
                    timeout: 10m
                    spec:
                      skipDryRun: false
        rollbackSteps:
          - step:
              name: Canary Delete
              identifier: rollbackCanaryDelete
              type: K8sCanaryDelete
              timeout: 10m
              spec: {}
          - step:
              name: Rolling Rollback
              identifier: rollingRollback
              type: K8sRollingRollback
              timeout: 10m
              spec: {}
    failureStrategies:
      - onFailure:
          errors:
            - AllErrors
          action:
            type: StageRollback
```

