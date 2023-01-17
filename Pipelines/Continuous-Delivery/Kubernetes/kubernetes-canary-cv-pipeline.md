# Kubernetes Canary Deployment Pipeline with Continuous Verification Template

## Introduction

- This template is designed to perform a canary deployment of a kubernetes service into a given environment
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
  name: Kubernetes Canary Deployment
  identifier: Kubernetes_Canary_Deployment
  versionLabel: "1.0"
  type: Pipeline
  tags: {}
  spec:
    stages:
      - stage:
          name: Canary Deployment
          identifier: Canary_Deployment
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
                          type: Verify
                          name: Canary Verification
                          identifier: Canary_Verification
                          spec:
                            type: Canary
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
          tags: {}
          failureStrategies:
            - onFailure:
                errors:
                  - AllErrors
                action:
                  type: StageRollback
```
