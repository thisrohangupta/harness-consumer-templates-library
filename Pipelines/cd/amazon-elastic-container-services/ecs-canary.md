# Amazon Elastic Container Services Canary Deployment Pipeline Template

## Introduction

- This template is designed to perform a canary deployment of a ECS service into a given environment
- User can provide any service as input
- User can provide any environment as input along with related infrastructure definition.
- You can copy the YAML into your Account Level Template Library - Template Studio
- Optional: you can save this YAML below in Git and "Import from Git", Harness will read the configuration file from Github.

### Template

```YAML
template:
  name: ECS Canary Deployment
  identifier: ECS_Canary_Deployment
  versionLabel: "1.0"
  type: Pipeline
  tags:
    owner: Harness
  spec:
    stages:
      - stage:
          name: Canary Deployment
          identifier: Canary_Deployment
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
                    name: Canary Deployment
                    identifier: canaryDepoyment
                    steps:
                      - step:
                          name: ECS Canary Deploy
                          identifier: ecsCanaryDeploy
                          type: EcsCanaryDeploy
                          timeout: 10m
                          spec: {}
                      - step:
                          name: ECS Canary Delete
                          identifier: ecsCanaryDelete
                          type: EcsCanaryDelete
                          timeout: 10m
                          spec: {}
                - stepGroup:
                    name: Primary Deployment
                    identifier: primaryDepoyment
                    steps:
                      - step:
                          name: ECS Rolling Deploy
                          identifier: ecsRollingDeploy
                          type: EcsRollingDeploy
                          timeout: 10m
                          spec: {}
              rollbackSteps:
                - step:
                    name: ECS Canary Delete
                    identifier: ecsRollbackCanaryDelete
                    type: EcsCanaryDelete
                    timeout: 10m
                    spec: {}
                - step:
                    name: ECS Rolling Rollback
                    identifier: ecsRollingRollback
                    type: EcsRollingRollback
                    timeout: 10m
                    spec: {}
          tags: {}
          failureStrategies:
            - onFailure:
                errors:
                  - AllErrors
                action:
                  type: StageRollback
  description: Harness Sample Template
```