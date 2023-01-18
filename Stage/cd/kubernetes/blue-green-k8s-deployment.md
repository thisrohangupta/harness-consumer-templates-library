# Kubernetes Blue Green Deployment Stage Template

## Introduction

- Harness provides out of the box templates for common deployment capabilities
- This template is used to perform Kubernetes Deployments in a [Blue Green](https://developer.harness.io/docs/continuous-delivery/cd-deployments-category/deployment-concepts#bluegreen-deployment) deployment style
- You can copy this YAML into your Template Library Builder in the Harness and save it, or you can upload this file to a Git Repo, and import it into the Harness Template Library

## Blue Green Deployment Stage Template

### Inputs

Users will need to provide a set of inputs for the template to work

1. Service - *NOTE: Please make sure that your Kubernetes application has 2 Kubernetes Object Type:Services to point to primary and stage*
2. Environment
3. Infrastructure Definition
4. Connector to the Kubernetes Cluster


### Template

```YAML
template:
  name: Blue Green Deployment
  identifier: Blue_Green_Deployment
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
              name: Stage Deployment
              identifier: stageDeployment
              type: K8sBlueGreenDeploy
              timeout: 10m
              spec:
                skipDryRun: false
                pruningEnabled: false
          - step:
              name: Swap primary with stage service
              identifier: bgSwapServices
              type: K8sBGSwapServices
              timeout: 10m
              spec:
                skipDryRun: false
        rollbackSteps:
          - step:
              name: Swap primary with stage service
              identifier: rollbackBgSwapServices
              type: K8sBGSwapServices
              timeout: 10m
              spec:
                skipDryRun: false
    failureStrategies:
      - onFailure:
          errors:
            - AllErrors
          action:
            type: StageRollback

```