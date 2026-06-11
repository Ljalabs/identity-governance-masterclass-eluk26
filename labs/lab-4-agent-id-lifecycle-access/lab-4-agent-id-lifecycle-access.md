# Lab 4 - Entra Agent ID Lifecycle & Access Management

In this lab you will explore how you can create the different building blocks of Entra Agent ID, and how you can govern the lifecycle and access of the Agent.

&nbsp;

## Lab 4.1 - Create an Agent Blueprint

### Option 1: Create an Agent Blueprint in the Entra portal

1. Go to **Microsoft Entra admin center**.
2. Browse to **Agent ID** \> **Blueprints**.
3. Select **New blueprint**.
4. Enter blueprint details (name, description, owner/sponsor settings as needed).
5. Save and verify the blueprint is created.

### Option 2: Create an Agent Blueprint with Microsoft Graph (Graph Explorer)

First we create the Agent Blueprint:

**Resource URI**

`POST https://graph.microsoft.com/v1.0/applications/`

**Sample request body**  
> Update property values to match your tenant and the latest beta schema.

```json
{
  "@odata.type": "Microsoft.Graph.AgentIdentityBlueprint",
  "displayName": "My Agent Identity Blueprint",
  "sponsors@odata.bind": [
    "https://graph.microsoft.com/v1.0/users/<id>"
  ],
  "owners@odata.bind": [
    "https://graph.microsoft.com/v1.0/users/<id>"
  ]
}
```

Next we create a credential to be used for agent identity authentication:

**Resource URI**

`POST https://graph.microsoft.com/v1.0/applications/<agent-blueprint-id>/addPassword`

**Sample request body**  
> Update property values to match your tenant and the latest beta schema.

```json
{
  "passwordCredential": {
    "displayName": "My Secret",
    "endDateTime": "2026-08-05T23:59:59Z"
  }
}
```

Then to be able to  receive incoming requests from users and other agents, like for any web API, you need to define an identifier URI and OAuth scope for your agent identity blueprint:

**Resource URI**

`PATCH https://graph.microsoft.com/v1.0/applications/<agent-blueprint-id>`

**Sample request body**  
> Update property values to match your tenant and the latest beta schema.

```json
{
    "identifierUris": ["api://<agent-blueprint-id>"],
    "api": {
      "oauth2PermissionScopes": [
        {
          "adminConsentDescription": "Allow the application to access the agent on behalf of the signed-in user.",
          "adminConsentDisplayName": "Access agent",
          "id": "<generate-a-guid>",
          "isEnabled": true,
          "type": "User",
          "value": "access_agent"
        }
      ]
  }
}
```

Last, create a service principal for the blueprint:

**Resource URI**

`POST https://graph.microsoft.com/v1.0/serviceprincipals/microsoft.graph.agentIdentityBlueprintPrincipal`

**Sample request body**  
> Update property values to match your tenant and the latest beta schema.

```json
{
  "appId": "<agent-blueprint-app-id>"
}
```

### Option 3: Create an Agent Blueprint with Microsoft Graph PowerShell SDK

First we create the Agent Blueprint:

```azurepowershell
# Connect with delegated permission to create Agent ID blueprints

Connect-MgGraph -Scopes "AgentIdentityBlueprint.Create","User.Read" -TenantId <your-tenant-id>

$currentUser = Get-MgContext | Select-Object -ExpandProperty Account
$user = Get-MgUser -UserId $currentUser

Write-Host "Current user: $($user.DisplayName) ($($user.Id))"
Write-Host "Sponsor user: $($user.DisplayName) ($($user.Id))"

$body = @{
    "@odata.type" = "Microsoft.Graph.AgentIdentityBlueprint"
    "displayName" = "My Agent Identity Blueprint"
    "sponsors@odata.bind" = @("https://microsoft.graph.microsoft.com/v1.0/users/$($user.Id)")
    "owners@odata.bind" = @("https://microsoft.graph.microsoft.com/v1.0/users/$($user.Id)")
} | ConvertTo-Json -Depth 5

$response = Invoke-MgGraphRequest -Method POST -Uri "https://graph.microsoft.com/v1.0/applications/microsoft.graph.agentIdentityBlueprint" -Body $body -ContentType "application/json"

$response
```

Next we create a credential to be used for agent identity authentication:

```azurepowershell
Connect-MgGraph -Scopes "AgentIdentityBlueprint.AddRemoveCreds.All" -TenantId <your-tenant-id>

$applicationId = "<agent-blueprint-application-id>"

# Define the secret properties
$displayName = "My Secret"
$endDate = (Get-Date).AddYears(1).ToString("o")  # 1 year from now, in ISO 8601 format

# Construct the password credential
$passwordCredential = @{
    displayName = $displayName
    endDateTime = $endDate
}

# Add the password (client secret)
$response = Add-MgApplicationPassword -ApplicationId $applicationId -PasswordCredential $passwordCredential

# Output the generated secret (only returned once!)
Write-Host "Secret Text: $($response.secretText)"
```

Then to be able to  receive incoming requests from users and other agents, like for any web API, you need to define an identifier URI and OAuth scope for your agent identity blueprint:

```powershell
Connect-MgGraph -Scopes "AgentIdentityBlueprint.UpdateAuthProperties.All" -TenantId <your-tenant-id>

$AppId = "<agent-blueprint-id>"
$IdentifierUri = "api://<agent-blueprint-id>"
$ScopeId = [guid]::NewGuid()

# Construct the OAuth2 permission scope
$scope = @{
    adminConsentDescription = "Allow the application to access the agent on behalf of the signed-in user."
    adminConsentDisplayName = "Access agent"
    id = $ScopeId
    isEnabled = $true
    type = "User"
    value = "access_agent"
}

Update-MgApplication -ApplicationId $AppId -IdentifierUris @($IdentifierUri) -Api @{ oauth2PermissionScopes = @($scope) }
```

Last, create a service principal for the blueprint:

```azurepowershell
Connect-MgGraph -Scopes "AgentIdentityBlueprintPrincipal.Create" -TenantId
$body = @{
    appId   = "<agent-blueprint-client-id>"
}
Invoke-MgGraphRequest -Method POST -Uri "https://graph.microsoft.com/v1.0/serviceprincipals/microsoft.graph.agentIdentityBlueprintPrincipal" -Headers @{ "OData-Version" = "4.0" } -Body ($body | ConvertTo-Json)
```

## Lab 4.2 - Create an Agent Identity

### Option 1: Create an Agent Identity in the Entra portal

1. Go to **Microsoft Entra admin center**.
2. Browse to **Agent ID** \> **Agent Identities**.
3. Select **New Agent Identity**.
4. Select blueprint and specify details (name, description, owner/sponsor settings as needed).
5. Save and verify the agent identity is created.

### Option 2a: Create an Agent Identity with Microsoft Graph (Graph Explorer)

**Resource URI**

`POST https://graph.microsoft.com/beta/serviceprincipals/Microsoft.Graph.AgentIdentity`

**Sample request body**  
> Update property values to match your tenant and the latest beta schema.

```json
{
	"displayName": "My Agent Identity",
	"agentIdentityBlueprintId": "<my-agent-blueprint-id>",
	"sponsors@odata.bind": [
		"https://graph.microsoft.com/v1.0/users/<id>",
		"https://graph.microsoft.com/v1.0/groups/<group-id>"
	]
}
```

### Option 2b: Delete an Agent Identity with Microsoft Graph (Graph Explorer)


**Resource URI**

`DELETE https://graph.microsoft.com/beta/serviceprincipals/<agent-identity-id>`

## Lab 4.3 - Create a CA Policy targeting Agents

Create a simple Condititional Access Policy targeting Agent Identities and select the one you created above. Restrict allowed location.

## Lab 4.4 - Create an Access Package that Agents Sponsors can request

This can only be done if you have Entra Agent ID licenses.

## Lab 4.5 - OPTIONAL (ADVANCED) - Explore Entra ID Agent Samples

This lab is optional and for referall only if you really want to deep dive into Agent ID and Agent Building. For advanced concepts, look at the https://github.com/microsoft/entra-agentid-samples, and explore the samples there.
