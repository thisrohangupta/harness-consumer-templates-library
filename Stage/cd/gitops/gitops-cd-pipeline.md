# GitOps Deployment Stage Template

### inputs

- `service` - User's needs to provide a Harness Kubernetes GitOps Service
- `environment` - User needs to configure or provide the template a Harness environment
- `GitOps Cluster` - GitOps Cluster must be configured in order to orchestrate with the target cluster, this will be passed as input post environment input


### template

```YAML
template:
  name: Gitops PR Deployment Stage
  type: Stage
  spec:
    type: Deployment
    spec:
      deploymentType: Kubernetes
      gitOpsEnabled: true
      execution:
        steps:
          - step:
              type: GitOpsUpdateReleaseRepo
              name: Update Release Repo
              identifier: updateReleaseRepo
              timeout: 10m
              spec: {}
          - step:
              type: MergePR
              name: Merge PR
              identifier: mergePR
              spec:
                deleteSourceBranch: true
              timeout: 10m
        rollbackSteps: []
      service:
        serviceRef: <+input>
        serviceInputs: <+input>
      environment:
        environmentRef: <+input>
        deployToAll: <+input>
        environmentInputs: <+input>
        gitOpsClusters: <+input>
    failureStrategies:
      - onFailure:
          errors:
            - AllErrors
          action:
            type: StageRollback
  versionLabel: "1.0"
  tags:
    owner: Harness
  description: Harness Sample Stage Template
  identifier: Gitops_PR_Deployment_Stage
```
