---
title: "Automating Azure Service Principal with Terraform"
date: 2024-04-20
# weight: 1
# aliases: ["/first"]
tags: ["DevOps", "SRE"]
author: "Colin Loh"
showToc: false
TocOpen: false
draft: false
hidemeta: false
comments: false
# canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: false
# cover:
#     image: "<image path/url>" # image path/url
#     alt: "<alt text>" # alt text
#     caption: "<text>" # display caption under cover
#     relative: false # when using page bundles set this to true
#     hidden: true # only hide on current single page
---
## Introduction:

In this blog post, we'll explore how to efficiently manage Azure Active Directory (AD) Service Principals using Terraform. Automating this process helps in maintaining consistent security standards and simplifying the configuration of Azure resources without having the need to input UUID. 

&nbsp;


## Terraform Module Call

```
module "service_principal" {
  source = "../.."

  service_principal = {
    name       =  "SPN-ONE"
    permissions = [
      {
        Permission = "MicrosoftGraph",
        Role       = ["User.ReadWrite.All", "User.Read.All"]
        Scope      = ["User.ReadWrite"]
      },
      {
        Permission = "DynamicsCrm",
        Scope      = ["user_impersonation"]
      }
    ]
  }
}
```

The most important thing to note is `permissions` array, it defines a list of action that the Service Principal can perform.

- `Permission` : The API permission for the App Registration/Service Principal, specifying which services or APIs the application can access.
- `Roles`: Application Permissions allow the application to operate autonomously, without user intervention, and typically require admin consent.
- `Scope`: Delegated Permissions do not require admin consent and allow the application to act on behalf of a user, accessing resources within the permissions granted to that user.

&nbsp;

## The Module 

Now we get into the fun stuff 🎉! Whilst going through the module I am going to split it up into some sub-sections to make it easier for us to talk through. 

```
resource "azuread_service_principal" "resource_id" {
  for_each = { 
    for perm in var.service_principal.permissions : 
    format("%s_%s", var.service_principal.name, perm.Permission) => perm 
  }

  client_id    = data.azuread_application_published_app_ids.well_known.result[each.value.Permission]
  use_existing = true
}
```

There are several `well-known` Enterprise Application that is already provided by Microsoft within Auzre universe. It is a universal API endpoint that provides access to various Microsoft cloud service resources. When you specify `use_existing = true`, you indicate that the service principal should be linked to an already existing application in Azure AD. 

The data source fetch is crucial because it ensures you're referencing the correct, universally recognized Microsoft Graph application ID.

&nbsp;

```
resource "azuread_application" "this" {
  display_name = var.service_principal.name
  owners       = [data.azuread_client_config.current.object_id]

  dynamic "required_resource_access" {
    for_each = var.service_principal.permissions

    content {
      resource_app_id = data.azuread_application_published_app_ids.well_known.result[required_resource_access.value.Permission]

      dynamic "resource_access" {
        for_each = can(required_resource_access.value.Role) ? toset(required_resource_access.value.Role) : toset([])

        content {
          id   = azuread_service_principal.resource_id[format("%s_%s", var.service_principal.name, required_resource_access.value.Permission)].app_role_ids[resource_access.value]
          type = "Role"
        }
      }

      dynamic "resource_access" {
        for_each = can(required_resource_access.value.Scope) ? toset(required_resource_access.value.Scope) : toset([])

        content {
          id   = azuread_service_principal.resource_id[format("%s_%s", var.service_principal.name, required_resource_access.value.Permission)].oauth2_permission_scope_ids[resource_access.value]
          type = "Scope"
        }
      }
    }
  }
}

resource "azuread_service_principal" "principal_id" {
  client_id = azuread_application.this.client_id
}

```

Once the service principal for our `well-known` Enterprise Application, we need to create our own Enterprise Application before we can create our custom SPN. We use `dynamic` block which iterate over roles and scopes to configure access permissions dynamically based on input variables.

Note that `Application permission` or `Role` requires us to use app_role_ids whereas `Delegated permission` or `Scope` requires us to use oauth2_permission_scope_ids basically the `content` specifies the configuration of app role IDs and OAuth2 permission scopes for the service principal.

&nbsp;

```
resource "azuread_app_role_assignment" "admin_consent" {
  for_each = {
    for i in flatten([
        for perm in var.service_principal.permissions : [
            for role in perm.Role : {
                spn = var.service_principal.name
                role = role
                permission = perm.Permission
              } if length(perm.Role) > 0
            ]
    ]) : format("%s_%s", i.spn, i.role) => i
  }

  principal_object_id = azuread_service_principal.principal_id.object_id
  resource_object_id  = azuread_service_principal.resource_id[format("%s_%s", var.service_principal.name, each.value.permission)].object_id
  app_role_id         = azuread_service_principal.resource_id[format("%s_%s", var.service_principal.name, each.value.permission)].app_role_ids[each.value.role]
}
```

The above manages app role assignment for a group, user or service principal. But more importantly it can be used to grant admin consent for application permissions. This require a little interpolation but I will try my best to explain it. 

We use `Flatten` which allow us to convert our list of list into a single flat list. Remember we have a list of Roles and/or Scopes within the list of Service Principal `permissions`. Flattening ensures that the for_each can iterate over a simple list of objects. 

The final mapped structure uses format("%s_%s", i.spn, i.role) to create a unique key for each role assignment, and i represents the value (containing spn, role, and permission).

- `principal_object_id`: This is the object ID of the service principal to which the role is being assigned. It is obtained from another resource, azuread_service_principal.principal_id, which defines the service principal.
- `resource_object_id`: This is the object ID of the application or another service principal against which the role is defined. It dynamically references the object ID using each.value.permission, which corresponds to the permission involved in the role assignment.
- `app_role_id`: This identifies the specific role within the application or service that is being assigned to the service principal. The role ID is retrieved dynamically, matching the role and permission.


&nbsp;

## Conclusion

One of the persistent challenges in managing Azure AD Service Principals is the requirement for admin consent when setting application permissions. This process often requires manual intervention, which can lead to delays or even be overlooked, posing a risk to both security and efficiency. 

Our Terraform module addresses this issue by automating the assignment of application permissions, eliminating the need for manual admin consent and UUID entries. This automation not only streamlines the process but also ensures that each service principal is configured consistently and securely, adhering to best practices without requiring direct administrative action. Embrace the power of automation with our solution to enhance your Azure environment's management and security.

You can find this module [here](https://github.com/Colin-Loh/terraform-azuread-service-principal)
