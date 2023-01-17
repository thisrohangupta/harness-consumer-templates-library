# Kubernetes Blue-Green Deployment Pipeline Template

## Introduction

- This template is designed to perform a blue-green deployment of a kubernetes service into a given environment
- User can provide any service as input 
- User can provide any environment as input along with related infrastructure definition.
- You can copy the YAML into your Account Level Template Library - Template Studio
- Optional: you can save this YAML below in Git and "Import from Git", Harness will read the configuration file from Github.


### Inputs

1. `Service`
2. `Environment`
3. `Infrastructure Definition`
4. `Kubernetes Connector`

```YAML
template:
  name: Kubernetes Blue Green Deployment
  identifier: Kubernetes_Blue_Green_Deployment
  versionLabel: "1.0"
  type: Pipeline
  tags: {}
  spec:
    stages:
      - stage:
          name: Blue Green Deployment
          identifier: Blue_Green_Deployment
          description: ""
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
          tags: {}
          failureStrategies:
            - onFailure:
                errors:
                  - AllErrors
                action:
                  type: StageRollback

```