# Kubernetes Rolling Deployment Stage Template

## Introduction

- Harness provides out of the box templates for common deployment capabilities
- This template is used to perform Kubernetes Deployments in a [Rolling deployment](https://developer.harness.io/docs/continuous-delivery/cd-deployments-category/deployment-concepts#rolling-deployment) style.
- You can copy this YAML into your Template Library Builder in the Harness and save it, or you can upload this file to a Git Repo, and import it into the Harness Template Library

## Rolling Deployment Stage Template YAML

### Inputs

Users will need to provide a set of inputs for the template to work

Users will need to provide a set of inputs for the template to work

1. Service
2. Environment
3. Infrastructure Definition
4. Connector to the Kubernetes Cluster

### Template

```YAML
template:
  name: CD Rolling
  identifier: CD_Rolling_Deploy
  versionLabel: "1.0"
  type: Stage
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
          - step:
              name: Rollout Deployment
              identifier: rolloutDeployment
              type: K8sRollingDeploy
              timeout: 10m
              spec:
                skipDryRun: false
                pruningEnabled: false
        rollbackSteps:
          - step:
              type: K8sRollingRollback
              name: Rolling Rollback
              identifier: Rolling_Rollback
              spec:
                pruningEnabled: false
              timeout: 10m
    failureStrategies:
      - onFailure:
          errors:
            - AllErrors
          action:
            type: StageRollback
    when:
      pipelineStatus: Success
      condition: <+input>
```