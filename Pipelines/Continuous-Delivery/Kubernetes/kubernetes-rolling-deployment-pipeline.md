# Kubernetes Rolling Deployment Pipeline Template

## Introduction

- This template is designed to perform a rolling deployment of a kubernetes service into a given environment
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
  name: Kubernetes Rolling Deployment
  identifier: Kubernetes_Rolling_Deployment
  versionLabel: "1.0"
  type: Pipeline
  tags: 
    owner: Harness
  spec:
    stages:
      - stage:
          name: Rolling Deployment
          identifier: Rolling_Deployment
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
                    name: Rollout Deployment
                    identifier: rolloutDeployment
                    type: K8sRollingDeploy
                    timeout: 10m
                    spec:
                      skipDryRun: false
                      pruningEnabled: false
              rollbackSteps:
                - step:
                    name: Rollback Rollout Deployment
                    identifier: rollbackRolloutDeployment
                    type: K8sRollingRollback
                    timeout: 10m
                    spec:
                      pruningEnabled: false
          tags: {}
          failureStrategies:
            - onFailure:
                errors:
                  - AllErrors
                action:
                  type: StageRollback
  description: Harness Sample Pipeline Template
```
