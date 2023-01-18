# Amazon ECS Canary Deployment Stage Template


## Canary Deployment Stage Template

### Inputs

Users will need to provide a set of inputs for the template to work

1. ECS Deployment type - Harness Service
2. Environment
3. Infrastructure Definition - AWS Cloud Infrastructure
4. AWS Connector

### Template

```YAML
template:
  name: ECS Canary Deployment Stage
  type: Stage
  spec:
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
    failureStrategies:
      - onFailure:
          errors:
            - AllErrors
          action:
            type: StageRollback
  identifier: ECS_Canary_Deployment_Stage
  versionLabel: "0.1"
  description: Harness Sample Stage Template
  tags:
    owner: Harness

```
