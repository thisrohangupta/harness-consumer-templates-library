# Native Helm Deployment Pipeline Template

## Introduction

- This template is designed to perform a helm deployment of a kubernetes service into a given environment
- User can provide any service as input 
- User can provide any environment as input along with related infrastructure definition.
- You can copy the YAML into your Account Level Template Library - Template Studio
- Optional: you can save this YAML below in Git and "Import from Git", Harness will read the configuration file from Github.

### Inputs

1. `Service`
2. `Environment`
3. `Infrastructure Definition`
4. `Kubernetes Connector`

### Template

```YAML
template:
  name: Helm Deployment Pipeline
  identifier: Helm_Deployment_Pipeline
  versionLabel: "1.0"
  type: Pipeline
  tags: 
    owner: Harness
  spec:
    stages:
      - stage:
          name: Deploy Helm
          identifier: Deploy_Helm
          description: ""
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
          tags: {}
          failureStrategies:
            - onFailure:
                errors:
                  - AllErrors
                action:
                  type: StageRollback
  description: Harness Sample Pipeline Template

```
