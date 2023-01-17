# Amazon Elastic Container Services Rolling Deployment Pipeline Template

## Introduction

- This template is designed to perform a rolling deployment of a ECS service into a given environment
- User can provide any service as input
- User can provide any environment as input along with related infrastructure definition.
- You can copy the YAML into your Account Level Template Library - Template Studio
- Optional: you can save this YAML below in Git and "Import from Git", Harness will read the configuration file from Github.

### Template

```YAML
template:
  name: ECS Rolling Deployment
  identifier: ECS_Rolling_Deployment
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
                - step:
                    name: ECS Rolling Deploy
                    identifier: ecsRollingDeploy
                    type: EcsRollingDeploy
                    timeout: 10m
                    spec: {}
              rollbackSteps:
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
  description: Harness Sample Pipeline Template
```
