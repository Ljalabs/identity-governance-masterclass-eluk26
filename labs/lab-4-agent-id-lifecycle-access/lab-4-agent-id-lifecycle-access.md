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

**Resource URI**

`POST https://graph.microsoft.com/beta/identity/agentId/blueprints`

**Sample request body**  
> Update property values to match your tenant and the latest beta schema.

```json
{
    "displayName": "Contoso Support Agent Blueprint",
    "description": "Blueprint for support automation agents",
    "category": "operations",
    "owners": [
        {
            "userPrincipalName": "<YOUR-OWNER-USER>@<YOUR-TENANT-DOMAIN>"
        }
    ],
    "sponsors": [
        {
            "userPrincipalName": "<YOUR-SPONSOR-USER>@<YOUR-TENANT-DOMAIN>"
        }
    ]
}
```

### Option 3: Create an Agent Blueprint with Microsoft Graph PowerShell SDK

```azurepowershell
# Connect with delegated permission to create Agent ID blueprints
Connect-MgGraph -Scopes "AgentIdentityManagement.ReadWrite.All"
Select-MgProfile -Name "beta"

$uri = "https://graph.microsoft.com/beta/identity/agentId/blueprints"

$body = @{
    displayName = "Contoso Support Agent Blueprint"
    description = "Blueprint for support automation agents"
    category    = "operations"
    owners      = @(
        @{ userPrincipalName = "<YOUR-OWNER-USER>@<YOUR-TENANT-DOMAIN>" }
    )
    sponsors    = @(
        @{ userPrincipalName = "<YOUR-SPONSOR-USER>@<YOUR-TENANT-DOMAIN>" }
    )
} | ConvertTo-Json -Depth 10

Invoke-MgGraphRequest -Method POST -Uri $uri -Body $body -ContentType "application/json"
```

## Lab 4.2 - Create an Agent Identity

### Option 1: Create an Agent Identity in the Entra portal

1. Go to **Microsoft Entra admin center**.
2. Browse to **Agent ID** \> **Agent Identities**.
3. Select **New Agent Identity**.
4. Select blueprint and specify details (name, description, owner/sponsor settings as needed).
5. Save and verify the agent identity is created.

### Option 2: Create an Agent Identity with Microsoft Graph (Graph Explorer)

#### Resource URI

`POST https://graph.microsoft.com/beta/identity/agentId/identities

**Sample request body**  
> Update property values to match your tenant and the latest beta schema.

```json
{
    "displayName": "Contoso Support Agent Identity",
    "description": "Agent identity for support automation",
    "blueprintId": "<YOUR-BLUEPRINT-ID>",
    "owners": [
        {
            "userPrincipalName": "<YOUR-OWNER-USER>@<YOUR-TENANT-DOMAIN>"
        }
    ],
    "sponsors": [
        {
            "userPrincipalName": "<YOUR-SPONSOR-USER>@<YOUR-TENANT-DOMAIN>"
        }
    ]
}
```

### Option 3: Create an Agent Blueprint with Microsoft Graph PowerShell SDK

```azurepowershell
# Connect with delegated permission to create Agent ID blueprints
Connect-MgGraph -Scopes "AgentIdentityManagement.ReadWrite.All"
Select-MgProfile -Name "beta"

$uri = "https://graph.microsoft.com/beta/identity/agentId/identities"

$body = @{
    displayName = "Contoso Support Agent Identity"
    description = "Agent identity for support automation"
    blueprintId = "<YOUR-BLUEPRINT-ID>"
    owners      = @(
        @{ userPrincipalName = "<YOUR-OWNER-USER>@<YOUR-TENANT-DOMAIN>" }
    )
    sponsors    = @(
        @{ userPrincipalName = "<YOUR-SPONSOR-USER>@<YOUR-TENANT-DOMAIN>" }
    )
} | ConvertTo-Json -Depth 10

Invoke-MgGraphRequest -Method POST -Uri $uri -Body $body -ContentType "application/json"
```

## Lab 4.3 - Create a CA Policy targeting Agents

Create a simple Condititional Access Policy targeting Agent Identities and select the one you created above. Restrict allowed location.

## Lab 4.4 - Create an Access Package that Agents Sponsors can request

This can only be done if you have Entra Agent ID licenses.

## Lab 4.5 - OPTIONAL (ADVANCED) - Explore Entra ID Agent Samples

This lab is optional and for referall only if you really want to deep dive into Agent ID and Agent Building. For advanced concepts, look at the https://github.com/microsoft/entra-agentid-samples, and explore the samples there.
