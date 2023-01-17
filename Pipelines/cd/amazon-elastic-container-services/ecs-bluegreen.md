# Amazon Elastic Container Services Blue Green Deployment Pipeline Template

## Introduction

- This template is designed to perform a Blue Green deployment of a ECS service into a given environment
- User can provide any service as input
- User can provide any environment as input along with related infrastructure definition.
- You can copy the YAML into your Account Level Template Library - Template Studio
- Optional: you can save this YAML below in Git and "Import from Git", Harness will read the configuration file from Github.

### Template

```YAML
template:
  name: ECS Blue Green Deployment
  identifier: ECS_Blue_Green_Deployment
  versionLabel: "1.0"
  type: Pipeline
  description: Harness Sample Pipeline Template
  tags:
    owner: Harness
  spec:
    stages:
      - stage:
          name: Blue Green Deploy
          identifier: Blue_Green_Deploy
          description: ""
          type: Deployment
          spec:
            deploymentType: ECS
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
                    name: Blue Green Deployment
                    identifier: blueGreenDepoyment
                    steps:
                      - step:
                          name: ECS Blue Green Create Service
                          identifier: EcsBlueGreenCreateService
                          type: EcsBlueGreenCreateService
                          timeout: 10m
                          spec: {}
                      - step:
                          name: ECS Blue Green Swap Target Groups
                          identifier: EcsBlueGreenSwapTargetGroups
                          type: EcsBlueGreenSwapTargetGroups
                          timeout: 10m
                          spec: {}
              rollbackSteps:
                - step:
                    name: ECS Blue Green Rollback
                    identifier: EcsBlueGreenRollback
                    type: EcsBlueGreenRollback
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
