# Helm Deployment Stage Template

## Introduction

- This template is designed to perform a helm deployment of a kubernetes service into a given environment
- User can provide any service as input 
- User can provide any environment as input along with related infrastructure definition.
- You can copy the YAML into your Account Level Template Library - Template Studio
- Optional: you can save this YAML below in Git and "Import from Git", Harness will read the configuration file from Github.
- You can use this template in your pipelines, by selecting "Use Template" option when creating a stage.

### Inputs

1. `Service`
2. `Environment`
3. `Infrastructure Definition`
4. `Kubernetes Connector`

```YAML
template:
  name: Deploy Helm
  type: Stage
  spec:
    type: Deployment
    spec:
      deploymentType: NativeHelm
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
              name: Helm Deployment
              identifier: helmDeployment
              type: HelmDeploy
              timeout: 10m
              spec:
                skipDryRun: false
        rollbackSteps:
          - step:
              name: Helm Rollback
              identifier: helmRollback
              type: HelmRollback
              timeout: 10m
              spec: {}
    failureStrategies:
      - onFailure:
          errors:
            - AllErrors
          action:
            type: StageRollback
  identifier: Deploy_Helm
  versionLabel: "1.0"

```