# Amazon ECS Rolling Deployment Stage Template



## Rolling Deployment Stage Template

### Inputs

Users will need to provide a set of inputs for the template to work

1. ECS Deployment type - Harness Service
2. Environment
3. Infrastructure Definition - AWS Cloud Infrastructure
4. AWS Connector

### Template

```YAML
template:
  name: ECS Rolling Deployment Stage
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
    failureStrategies:
      - onFailure:
          errors:
            - AllErrors
          action:
            type: StageRollback
  identifier: ECS_Rolling_Deployment_Stage
  description: Harness Sample Stage Template
  versionLabel: "1.0"
  tags:
    owner: Harness
```
