# Health Check Step Template

## Introduction

- Harness has an HTTP Step that can be used to perform Health Checks on Applicaton Endpoints
- HTTP Steps can be created and managed as Step Templates.
- You can take this step template, add it to your account level Template Library, and then link it in your pipeline.

### Template

#### Inputs

- `url`: This is the URL endpoint to make the GET call against
- `http.ResponseCode` - this is the response code received with the GET request

```YAML
template:
  name: HTTP Check
  identifier: HTTP_Check
  versionLabel: "1.0"
  type: Step
  description: "This is http template for health check "
  tags:
    owner: Harness
  spec:
    type: Http
    timeout: 10s
    spec:
      url: <+input>
      method: GET
      headers:
        - key: content-type
          value: application/json
      outputVariables: []
      requestBody: ""
      assertion: <+http.ResponseCode>==200
```
