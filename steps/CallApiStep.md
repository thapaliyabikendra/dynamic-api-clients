# Call API Step

## Purpose
Makes HTTP requests to external APIs using a dynamic HTTP client service.

## Step Type
```
Amnil.AccessControlManagementSystem.Workflows.Steps.CallApiStep, Amnil.AccessControlManagementSystem.Application
```

## Parameters

### Inputs
- `ApiRequest` (JObject): The request payload to be sent to the API
  - Contains HTTP request details like URL, method, headers, body
- `ApiClientName` (string): Name of the API client to be used
  - Must match a configured API client in the system
- `CurrentUserDetail` (JObject, Optional): User data

### Outputs
- `Response` (JObject): Contains the API response after execution
  - Includes status code, headers, and response body

## Usage Example

```json
{
    "Steps": [
    {
      "Id": "GetPMUserId",
      "StepType": "Amnil.AccessControlManagementSystem.Workflows.Steps.CallApiStep, Amnil.AccessControlManagementSystem.Application",
      "NextStepId": "RoleChangeAPICreate",
      "Inputs": {
        "ApiRequest": {
          "@email": "data[\"currentUserDetail\"].Email"
        },
        "ApiClientName": "\"GET_PM_USERS_API\""
      },
      "Outputs": {
        "pmUserId": "step.Response[\"data\"]"
      }
    },
    {
      "Id": "RoleChangeAPICreate",
      "StepType": "Amnil.AccessControlManagementSystem.Workflows.Steps.CallApiStep, Amnil.AccessControlManagementSystem.Application",
      "Inputs": {
        "ApiRequest": {
          "@userId": "data[\"pmUserId\"].processMakerUid",
          "@processId": "data[\"applicationDetails\"].processId",
          "@taskId": "data[\"applicationWorkflowInstanceId\"]",
          "@workflowId": "data[\"applicationWorkflowInstanceId\"]",
          "@employeeId": "data[\"operationFormData\"].userAccountForm.userAccountDetails.employeeId",
          "@employeeName": "data[\"operationFormData\"].userAccountForm.userAccountDetails.userAccountName",
          "@currentfunctionalTitle": "data[\"operationFormData\"].userAccountForm.userAccountDetails.currentfunctionalTitle",
          "@newFunctionalTitle": "data[\"operationFormData\"].roleChangeOperationForm.newfunctionalTitle",
          "@roleChange": "data[\"applicationDetails\"]"
        },
        "ApiClientName": "\"ROLE_CHANGE_API_CREATE\"",
        "CurrentUserDetail": "data[\"currentUserDetail\"]"
      },
      "Outputs": {
        "appUid": "step.Response[\"appUid\"]",
        "appNumber": "step.Response[\"appNumber\"]"
      }
    },
    ]
}
```