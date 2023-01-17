# Native Helm Deployment Pipeline Template

## Introduction

- This template is designed to perform a helm deployment of a kubernetes service into a given environment
- User can provide any native helm service as input
- User can provide any environment as input along with related infrastructure definition.
- You can copy the YAML into your Account Level Template Library - Template Studio
- We can run Harness Continuous Verification on the deployed Helm Chart
- For Continuous Verification to work, you need to provide a verification provider connecter in Harness
- You should also configure a Monitored Service in the SRM Product in Harness.
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
  name: Helm Deployment Pipeline
  identifier: Helm_Deployment_Pipeline
  versionLabel: "1.0"
  type: Pipeline
  tags: 
    owner: Harness
  spec:
    stages:
      - stage:
          name: Deploy Helm
          identifier: Deploy_Helm
          description: ""
          type: Deployment
          spec:
            deploymentType: NativeHelm
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
                    name: Helm Deployment
                    identifier: helmDeployment
                    type: HelmDeploy
                    timeout: 10m
                    spec:
                      skipDryRun: false
                - step:
                    type: Verify
                    name: Helm Deployment Verification
                    identifier: Helm_Deployment_Verification
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
                    name: Helm Rollback
                    identifier: helmRollback
                    type: HelmRollback
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
