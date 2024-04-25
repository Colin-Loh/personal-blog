---
title: "Automating Azure Service Principal with Terraform"
date: 2024-04-24
# weight: 1
# aliases: ["/first"]
tags: ["Terraform", "Admin Consent", "Service Principals", "Application Registration"]
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
# Introduction

In this blog post, we will explore how to efficiently manage Azure Active Directory (AD) Service Principals using Terraform. Before we delve into automating this process, let's discuss what a Service Principal is, what App Registration entails, and why it's necessary.

# App Registration

As the name suggests, App Registration provides Azure AD **authentication / authorization** for any applications (e.g. Azure DevOps). 

This is how it looks like : 

![app-reg](https://learn.microsoft.com/en-us/graph/images/quickstart-register-app/portal-02-app-reg-01.png)

Once registered, the application appears in  `App Registrations` and  `Enterprise Registrations` also known as **Service Principals.** You can use the Enterprise Application page in Azure AD to view a list of Service Principals within the tenant.

Confused yet? Lets talk about the differences: 

**Application Registration -** This is the actual instance of the application or its local representation. Simply put, it's where you configure the authentication and authorization settings for your application.

Important aspects in the portal include:

- `API Permission` : The API permission for the App Registration/Service Principal, specifying which services or APIs the application can access. (e.g. Microsoft Graph)
- `Application Permission`: Application Permissions allow the application to operate autonomously, without user intervention, and typically require admin consent.
- `Delegated Permission`: Delegated Permissions do not require admin consent and allow the application to act on behalf of a user, accessing resources within the permissions granted to that user.
- `Client Secrets`:  Your app will pass credentials, i.e. either via a client secret or a certificate to Azure AD to prove its identity, and therefore request tokens

**Enterprise Registration -** 

For single-tenant applications, an Enterprise App with the corresponding name is automatically created upon registration. This setup focuses on how your app functions within your 'local' tenant, including:

- `API Permission Management`: Admins can review and manage scope permissions set by your app under App Registration.
- `Access Management` : Configure owners or groups to manage your app, set up SSO, and, if necessary, establish additional approval access before granting permissions.

<aside>
💡 **Application Registration is responsible for setting client secrets / certificates, scopes and API permission whereas Service Principal controls identity, authorisation and policies.**

</aside>

For this blog post and Terraform automation we will be focusing on the following: 

1. Creating application registration within Azure Active Directory.
2. Creating a service principal associated with an application within Azure Active Directory.
3. Grant admin consent for application permissions.
4. Understanding Terraform `for_each` expressions, mutating, filtering and grouping.

This is how it looks like in the Portal: 

![admin-consent](https://learn.microsoft.com/en-us/entra/identity-platform/media/consent-framework/grant-consent.png)

<aside>
💡 Granting admin consent to an application will grant the app and the app's publisher access to your tenant data. Carefully review what permission the application is requesting before granting consent.

</aside>

# Terraform Module Call

Below is our call to  `service-principal` module. This module will handle everything from creating the application registration to associating the service principal and granting admin consent.

```jsx
module "service_principal" {
  source = "../.."

  service_principals = {
    SPN-ONE = {
      permissions = [
        {
          api         = "MicrosoftGraph"
          delegated   = ["User.ReadWrite"]
        },
        {
          api         = "DynamicsCrm"
          delegated   = ["user_impersonation"]
        }
      ],
      web = {
        urls = {
          homePageURL  = "https://app.example.net"
          logoutURL    = "https://app.example.net/logout"
          redirectURLs = ["https://app.example.net/account", "https://app.example.net/account2"]

          grant = {
            useAccessTokens = true
            useIdTokens     = true
          }
        }
      }
    },
    SPN-TWO = {
      permissions = [
        {
          api         = "DynamicsCrm"
          delegated   = ["user_impersonation"]
        },
        {
          api         = "PowerBiService",
          application = ["Tenant.ReadWrite.All"]
        }
      ],
      web = {
        urls = {
          homePageURL  = "https://app.example.net"
          logoutURL    = "https://app.example.net/logout"
          redirectURLs = ["https://app.example.net/account", "https://app.example.net/account2"]
        }
      }
    },
    SPN-THREE = {}
  }
}
```

The most important thing to note is `permissions` array as it defines the capabilities that the application object can perform, which are then inherited by the service principal object. Here's a breakdown:

- `API` : The API permission for the App Registration/Service Principal, specifying which services or APIs the application can access. (e.g. Microsoft Graph)
- `Application`: Application Permissions allow the application to operate autonomously, without user intervention, and typically require admin consent.
- `Delegated`: Delegated Permissions do not require admin consent and allow the application to act on behalf of a user, accessing resources within the permissions granted to that user.

# The Module

There are several `well_known` Enterprise Application that is already provided by Microsoft within Azure universe. `azure_service_principal` provides access to various Microsoft cloud service resources. 

When you specify `use_existing = true`, you indicate that the service principal should be linked to an already existing application in Azure AD.

```jsx
resource "azuread_service_principal" "well_known" {
  for_each = toset(flatten([
    for k, v in var.service_principals : [
      for values in v.permissions : values.api
    ]
  ]))

  client_id    = data.azuread_application_published_app_ids.well_known.result[each.value]
  use_existing = true
}
```

It is important to know some Terraform `for_each` expressions, in my block above we iterates over each **`k`** in **`var.service_principals`** For each service principal (**`k`**), it generates the list of **`api`** values from the inner comprehension. This results in a list of lists, where each inner list contains the **`api`** values corresponding to the permissions of a specific service principal.

**`flatten`** function takes the list of lists produced by the outer list comprehension and flattens it into a single list. This means that rather than having a separate list for each service principal, you get a single list that combines all **`api`** values across all service principals and their permissions.

**`toset`** function converts the flattened list of **`api`** values into a set, which automatically removes any duplicate **`api`** values. Sets in Terraform are collections of unique elements, so this step ensures that each **`api`** value is listed only once, regardless of how many times it appears across different service principals or permissions. 

This is the final output - removing any duplicates and list object. 

```
{"MicrosoftGraph", "DynamicsCrm"}
```

# Creating Application Registration

Once we have provisioned our `well_known` Service Principal, we will need to create an Application Registration before our SPN. We will use `dynamic` block which iterate over roles and scopes to configure access permissions dynamically based on input variables.

```jsx
dynamic "resource_access" {
  for_each = can(required_resource_access.value.application) ? toset(required_resource_access.value.application) : toset([])

  content {
    id   = azuread_service_principal.well_known[required_resource_access.value.api].app_role_ids[resource_access.value]
    type = "Role"
  }
}

dynamic "resource_access" {
  for_each = can(required_resource_access.value.delegated) ? toset(required_resource_access.value.delegated) : toset([])

  content {
    id   = azuread_service_principal.well_known[required_resource_access.value.api].oauth2_permission_scope_ids[resource_access.value]
    type = "Scope"
  }
}
```

This configuration sets up roles and delegated permissions for the application's required resource access. The application.app_role_ids and oauth2_permission_scope_ids mappings are crucial for referencing in other resources. Its good to note that both `application` uses **`application.app_role_ids`** and `delegated` uses **`oauth2_permission_scope_ids`.** 

Somethings to note about the special attributes: 

- **`application.app_role_ids` -**  A mapping of app role values to app role IDs, as published by the associated application, intended to be useful when referencing app roles in other resources in your configuration.
- **`oauth2_permission_scope_ids` -** A mapping of OAuth2.0 permission scope values to scope IDs, as exposed by the associated application, intended to be useful when referencing permission scopes in other resources in your configuration.

Once the App Registration is created, we can finally create our SPN with the following block:

```jsx
resource "azuread_service_principal" "principal_id" {
  for_each = var.service_principals

  client_id = azuread_application.this[each.key].client_id
}
```

# Granting Admin Consent

Granting admin consent for our App Registration requires us to mutate the structure of our variable. Our goal is to combine the service_principal object and the application list object to create a unique key. 

We want to be able to run a for_each over every **`application`** in **`v.application`**. For each application, it creates a map containing:

- **spn**: The name of the service principal (**`v.name`**).
- **role**: The current application being iterated over.
- **permission**: The API associated with the permission (**`perm.api`**).

The conditional **`if length(perm.application) > 0`** ensures that this inner comprehension only executes if there are applications listed in **`perm.application`**, avoiding processing empty lists.

```jsx
resource "azuread_app_role_assignment" "admin_consent" {
  for_each = {
    for i in flatten([
      for k, v in var.service_principals : [
        for perm in v.permissions : [
          for application in perm.application : {
            spn        = k
            role       = application
            permission = perm.api
          } if length(perm.application) > 0
        ]
      ]
    ]) : format("%s_%s", i.spn, i.role) => i
  }

  principal_object_id = azuread_service_principal.principal_id[each.value.spn].object_id
  resource_object_id  = azuread_service_principal.well_known[each.value.permission].object_id
  app_role_id         = azuread_service_principal.well_known[each.value.permission].app_role_ids[each.value.role]
}
```

This is the final output - notice that it only takes into account the object that has value within the application list(object)

```jsx
{
  "SPN-TWO_Tenant.ReadWrite.All" = {
    spn = "SPN-TWO"
    role = "Tenant.ReadWrite.All"
    permission = "PowerBiService"
  }
}
```

If you would like to learn more about `for_each` expression, explore Brendan Thompson's insightful blog post [here](https://brendanthompson.com/posts/2022/10/terraform-for-expression). Brendan covers both basic and advanced usage scenarios.

# Conclusion:

One of the persistent challenges in managing Azure AD Service Principals is the requirement for admin consent when setting application permissions. This process often requires manual intervention, which can lead to delays or even be overlooked, posing a risk to both security and efficiency.

Our Terraform module addresses this issue by automating the assignment of application permissions, eliminating the need for manual admin consent and UUID entries. This automation not only streamlines the process but also ensures that each service principal is configured consistently and securely, adhering to best practices without requiring direct administrative action. Embrace the power of automation with our solution to enhance your Azure environment’s management and security.

You can find this module on GitHub [here](https://github.com/Colin-Loh/terraform-azuread-service-principal)