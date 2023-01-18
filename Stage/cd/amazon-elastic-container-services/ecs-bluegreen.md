
# Amazon ECS Blue-Green Deployment Stage Template

## Blue Green Deployment Stage Template YAML

### Inputs

Users will need to provide a set of inputs for the template to work

1. ECS Deployment type - Harness Service
2. Environment
3. Infrastructure Definition - AWS Cloud Infrastructure
4. AWS Connector

### Template

```YAML
template:
  name: ECS Blue Green
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
    failureStrategies:
      - onFailure:
          errors:
            - AllErrors
          action:
            type: StageRollback
  identifier: ECS_Blue_Green
  description: Harness Stage Template Sample
  versionLabel: "1.0"
  tags:
    owner: Harness

```