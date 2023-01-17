# Kubernetes Golden Release Pipeline Template with CV

## Introduction

- This template is a sample Pipeline Template to perform a Kubernetes Deployment of a service to multiple environments
- The Dev and QA environments will be performed via a Rolling Deployment
- The Prod Environment will be deployed into via a Canary Deployment
- In between the stages, user's will be notified for Harness UI Approvals to approve the progress of the pipeline.

## Inputs

1. Service
2. Environment
3. Kubernetes Connectors for (Dev, QA, or Prod Environments)
4. Continuous Verification Step needs a Monitored Service to be configured

### Template

```YAML
template:
  name: Golden Release Pipeline
  identifier: Golden_Release_Pipeline
  versionLabel: "1.0"
  type: Pipeline
  tags: 
    owner: Harness
  spec:
    stages:
      - stage:
          name: Deploy to Dev
          identifier: Deploy_to_Dev
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
      - stage:
          name: Approve to QA
          identifier: Approve_to_QA
          description: ""
          type: Approval
          spec:
            execution:
              steps:
                - step:
                    name: Approval
                    identifier: Approval
                    type: HarnessApproval
                    timeout: 1d
                    spec:
                      approvalMessage: |-
                        Please review the following information
                        and approve the pipeline progression
                      includePipelineExecutionHistory: true
                      approvers:
                        minimumCount: 1
                        disallowPipelineExecutor: false
                        userGroups:
                          - account._account_all_users
                      approverInputs: []
          tags: {}
      - stage:
          name: Deploy to QA
          identifier: Deploy_to_QA
          description: ""
          type: Deployment
          spec:
            deploymentType: Kubernetes
            service:
              useFromStage:
                stage: Deploy_to_Dev
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
      - stage:
          name: Approve to Prod
          identifier: Approve_to_Prod
          description: ""
          type: Approval
          spec:
            execution:
              steps:
                - step:
                    name: Approval
                    identifier: Approval
                    type: HarnessApproval
                    timeout: 1d
                    spec:
                      approvalMessage: |-
                        Please review the following information
                        and approve the pipeline progression
                      includePipelineExecutionHistory: true
                      approvers:
                        minimumCount: 1
                        disallowPipelineExecutor: false
                        userGroups:
                          - account._account_all_users
                      approverInputs: []
          tags: {}
      - stage:
          name: Deploy to Prod
          identifier: Deploy_to_Prod
          description: ""
          type: Deployment
          spec:
            deploymentType: Kubernetes
            service:
              useFromStage:
                stage: Deploy_to_Dev
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
                          name: CV
                          identifier: CV
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
  description: Harness Sample Pipeline Template

```
