# Kubernetes Rolling Deployment Pipeline with Continuous Verification Template

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
5. `Verification Provider Connector`
6. `Monitored Service`

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
                - step:
                    type: Verify
                    name: Rolling Verification
                    identifier: Rolling_Verification
                    spec:
                      type: Rolling
                      monitoredService:
                        type: Default
                        spec: {}
                      spec:
                        sensitivity: MEDIUM
                        duration: 15m
                        deploymentTag: <+serviceConfig.artifacts.primary.tag>
                    timeout: 2h
                    failureStrategies:
                      - onFailure:
                          errors:
                            - Verification
                          action:
                            type: ManualIntervention
                            spec:
                              timeout: 2h
                              onTimeout:
                                action:
                                  type: StageRollback
                      - onFailure:
                          errors:
                            - Unknown
                          action:
                            type: ManualIntervention
                            spec:
                              timeout: 2h
                              onTimeout:
                                action:
                                  type: Ignore
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
