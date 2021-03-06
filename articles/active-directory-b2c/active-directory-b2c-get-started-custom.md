﻿---
title: 'Azure Active Directory B2C: Get Started With Custom Policies | Microsoft Docs'
description: How to get started with Azure Active Directory B2C custom policies
services: active-directory-b2c
documentationcenter: ''
author: rojasja
manager: krassk
editor: rojasja

ms.assetid: 658c597e-3787-465e-b377-26aebc94e46d
ms.service: active-directory-b2c
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: article
ms.devlang: na
ms.date: 04/04/2017
ms.author: joroja;parahk;gsacavdm
---
# Azure Active Directory B2C: Getting started with custom policies

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

After completing the steps in this article, your custom policy will support "local account" sign-up or sign-in using an email and password. You will also prepare your environment for adding additional identity providers (like Facebook or Azure AD).  Completion of these steps is encouraged before all other uses of the Azure AD B2C’s Identity Experience Framework and many essential concepts are introduced.

## Prerequisites

Before proceeding, ensure that you have an Azure AD B2C tenant. An Azure AD B2C tenant is a container for all your users, apps, policies, and more. If you don't have one already, you need to [create one](active-directory-b2c-get-started.md). We strongly encourage all developers to complete the Azure AD B2C built-in policy walkthroughs and configure their application with built-in policies before proceeding.  Your applications will work with both types of policies with a trivial change to the policy name to invoke the custom policy.

Access to custom policy editing requires a valid Azure subscription linked to your tenant.

## Add signing and encryption keys to your B2C tenant for use by Custom Policies

1. Navigate to the **Identity Experience Framework** blade in your Azure AD B2C tenant settings.
2. Select **Policy Keys** to view the keys available in your tenant.
3. Create `B2C_1A_TokenSigningKeyContainer` if it does not exist:
 * Click on **Add**
 * Options > `Generate`
 * Name >`TokenSigningKeyContainer` The prefix B2C_1A_ may be added automatically.
 * Key type > `RSA`
 * Dates - use defaults
 * Key usage > `Signature`
 * Click **Create**
4. Create `B2C_1A_TokenEncryptionKeyContainer` if it does not exist:
 * Click on **Add**
 * Options > `Generate`
 * Name >`TokenEncryptionKeyContainer` The prefix B2C_1A_ may be added automatically.
 * Key type > `RSA`
 * Dates - use defaults
 * Key usage > `Encryption`
 * Click **Create**
5. Create `B2C_1A_FacebookSecret`. If you already have a Facebook application secret, add it as a policy key to your tenant.  Otherwise, you must create the key with placeholder value so your policies pass validation.
 * Click on **Add**
 * Options > `Manual`
 * Name > `FacebookSecret` The prefix B2C_1A_ may be added automatically.
 * Secret > Enter your FacebookSecret from developers.facebook.com or "0" as a placeholder. *This is not your Facebook App ID*
 * Key usage > Signature
 * Click **Create** and confirm creation

## Register Identity Experience Framework applications

Azure AD B2C requires you to register two extra applications that are used by the engine to sign-up and sign-in users.

>[!NOTE]
>We must create two applications that enable sign-in using local accounts: IdentityExperienceFramework (a web app) and ProxyIdentityExperienceFramework (a native app) with delegated permission from IdentityExperienceFramework app. Local accounts exist only in your tenant. Your end-users sign-up with a unique email:password combination to access your tenant-registered applications.

### Create the IdentityExperienceFramework application

1. In the [Azure portal](https://portal.azure.com), [switch into the context of your Azure AD B2C tenant](active-directory-b2c-navigate-to-b2c-context.md).
1. Open the `Azure Active Directory` blade (not the Azure AD B2C blade). You may need to click **> More Services** to find it.
1. Select **App registrations**.
1. Click **+ New application registration**.
   * Name: `IdentityExperienceFramework`
   * Application type: `Web app/API`
   * Sign-on URL: `https://login.microsoftonline.com/yourtenant.onmicrosoft.com` where `yourtenant` is your Azure AD B2C tenant domain name.
1. Click **Create**.
1. Once created, select the newly created application `IdentityExperienceFramework`, click on **Properties**, copy the Application ID and save it for later.

### Create the proxy Identity Experience Framework application

1. Select **App registrations**.
1. Click **+ New application registration**.
   * Name: `ProxyIdentityExperienceFramework`
   * Application type: `Native`
   * Sign-on URL: `https://login.microsoftonline.com/yourtenant.onmicrosoft.com` where `yourtenant` is your Azure AD B2C tenant.
1. Click **Create**.
1. Once created, select the application `ProxyIdentityExperienceFramework`, click on **Properties**, copy the Application ID and save it for later.
1. Select **Required permissions**, then select **+ Add**, and then **Select an API**.
1. Search for the name `IdentityExperienceFramework`, select IdentityExperienceFramework in the results, and then click **Select**.
1. Check the checkbox next to `Access IdentityExperienceFramework` and then click **Select**.
1. Click **Done**.
1. Click **Grant Permissions** and confirm **Yes**

## Download starter pack and modify policies

Custom policies are a set of XML files that need to be uploaded to your Azure AD B2C tenant. We provide starter packs to get you going quickly. Each starter pack below has the minimum technical profiles and user journeys needed to achieve the scenarios as described:
 * LocalAccounts - enables the use of local accounts only
 * SocialAccounts - enables the use of social (or federated) accounts only
 * **SocialAndLocalAccounts**- we will use this one for this walkthrough
 * SocialAndLocalAccountsWithMFA - social, local, and MFA options are included here

Each starterpack contains:

* The [base file](active-directory-b2c-overview-custom.md#policy-files) of the policy. Few modifications are required to the base.
* The [extension file](active-directory-b2c-overview-custom.md#policy-files) of the policy.  This file is where most configuration changes are made.
* [Relying party files](active-directory-b2c-overview-custom.md#policy-files) These are task specific files called by your application.

>[!NOTE]
>If your XML editor supports validation, you may want to validate the files against the `TrustFrameworkPolicy_0.3.0.0.xsd` XML schema file that is located in the root folder of the starter pack. XML schema validation identifies errors before uploading.

Let's get started:

1. Download the "active-directory-b2c-custom-policy-starterpack" from GitHub.  [Download the zip](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/archive/master.zip) or run

    ```console
    git clone https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack
    ```
2. Open the `SocialAndLocalAccounts` folder.  The base file (`TrustFrameworkBase.xml`) in this folder contains content needed for both local and social/corporate accounts. The social content does not interfere with the steps for getting local accounts up and running.
3. Open `TrustFrameworkBase.xml`.  If you need an XML editor, [try Visual Studio Code](https://code.visualstudio.com/download), a lightweight cross platform editor.
4. In the root `TrustFrameworkPolicy` element, update the `TenantId` and `PublicPolicyUri` attributes, replacing `yourtenant.onmicrosoft.com` with your domain name of your Azure AD B2C tenant:

    ```xml
    <TrustFrameworkPolicy
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
    PolicySchemaVersion="0.3.0.0"
    TenantId="yourtenant.onmicrosoft.com"
    PolicyId="B2C_1A_TrustFrameworkBase"
    PublicPolicyUri="http://yourtenant.onmicrosoft.com">
    ```
>[!NOTE]
>The `PolicyId` is the policy name that you will see in the portal and the name by which this policy file will be referenced by other policy files.

5. Save the file.
6. Open `TrustFrameworkExtensions.xml` and make the same two changes by replacing `yourtenant.onmicrosoft.com` with your Azure AD B2C tenant. Make the same replacement in the element `<TenantId>` for a total of 3 changes.  Save the file.
7. Open `SignUpOrSignIn.xml`and make the same changes by replacing `yourtenant.onmicrosoft.com` with your Azure AD B2C tenant in three places. Save the file.
8. Open the password reset and profile edit files, and make the same changes by replacing `yourtenant.onmicrosoft.com` with your Azure AD B2C tenant in three places in each file. Save the files.

### Add the application IDs to your custom policy
Add the application IDs to the extensions file (`TrustFrameworkExtensions.xml`).

1. In the extensions file (`TrustFrameworkExtensions.xml`), find the element `<TechnicalProfile Id="login-NonInteractive">`.
2. Replace both instances of `IdentityExperienceFrameworkAppId` with the application ID of the Identity Experience Framework application that you created earlier. Here is an example:

```xml
<Item Key="client_id">8322dedc-cbf4-43bc-8bb6-141d16f0f489</Item>
```

3. Replace both instances of `ProxyIdentityExperienceFrameworkAppId` with the application ID of the Proxy Identity Experience Framework application that you created earlier
4. Save your extensions file.

## Upload the policies to your tenant

1. In the [Azure portal](https://portal.azure.com), [switch into the context of your Azure AD B2C tenant](active-directory-b2c-navigate-to-b2c-context.md) and open the Azure AD B2C blade.
2. Click **Identity Experience Framework**
3. Select **Upload Policy** to upload policy files

    >[!WARNING]
    >The custom policy files must be uploaded in the following order:

1. Upload `TrustFrameworkBase.xml`.
2. Upload `TrustFrameworkExtensions.xml`.
3. Upload `SignUpOrSignin.xml`.
4. Upload your other policy files.

When a file is uploaded, the name of the policy file is prepended with `B2C_1A_`.

## Test the custom policy using "Run Now"

1. Open the **Azure AD B2C Settings** and navigate to **Identity Experience Framework**.

>[!NOTE]
>**Run now** Requires at least one application to be pre-registered on the tenant. Applications must be registered in the B2C tenant, using the **Applications** menu selection in B2C or in Identity Experience Framework to invoke both built-in and custom policies. Only one registration per application is needed. 

Learn how to register applications in the Azure AD B2C [Get Started](active-directory-b2c-get-started.md) or the [Application registration](active-directory-b2c-app-registration.md) articles.  

2. Open the Relying Party (RP) custom policy that you uploaded `B2C_1A_signup_signin`, and click the **Run now** button.


3. You should be able to sign-up using an email address.
4. Sign-in with the same account to confirm you have a correct configuration

>[!NOTE]
>A common cause of sign-in failure is an improperly configured IdentityExperienceFramework app.


## Next steps

### Add Facebook as an identity provider
To set up Facebook:
1. [Configure a Facebook application in developers.facebook.com](active-directory-b2c-setup-fb-app.md)
2. [Add the Facebook application secret to your Azure AD B2C Tenant](#add-signing-and-encryption-keys-to-your-b2c-tenant-for-use-by-custom-policies)
3. Add the Facebook application id in the TrustFrameworkExtensions policy file in the `Item Key="client_id"`:

```xml
<TechnicalProfile Id="Facebook-OAUTH">
  <Metadata>
  <!--Replace the value of client_id in this technical profile with your the Facebook App ID"-->
    <Item Key="client_id">00000000000000</Item>
```
4. Upload the TrustFrameworkExtensions.xml policy file to your tenant.
5. Test by using **Run now** or invoke the policy directly from your registered application.

### Add Azure Active Directory as an identity provider
The base file that we used in this getting started guide already contains some of the content that you need for adding other identity providers. To set up login using Azure AD accounts, [continue here](active-directory-b2c-setup-aad-custom.md).


## Reference

[Overview](active-directory-b2c-overview-custom.md) of Custom Policies in Azure AD B2C using the Identity Experience Framework
