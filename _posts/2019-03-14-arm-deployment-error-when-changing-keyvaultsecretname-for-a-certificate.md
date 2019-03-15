---
layout: post
comments: true
author: Peter
title: ARM deployment error when changing keyVaultSecretName for a certificate
date: 2019-03-14 23:00:00 +0000
slug: arm-deployment-error-when-changing-keyvaultsecretname
wordpress_id: ''
tags:
- ARM
- Azure KeyVault
categories:
- Azure

---
Last week I was busy updating the certificates stored in Azure KeyVault for a project I'm working on. Previously we added the certificates that were referenced in App Services as secrets in the KeyVault with a application/x-pkcs12 type. 

We now wanted to change those to real certificates, so the renewal of the certificates can be managed from Azure KeyVault. 

After storing the certificates in KeyVault and modifying the ARM templates for the certificate resources to reference the new secretNames I ran into an ARM deployment error with the following message: "The parameter KeyVaultId & KeyVaultSecretName has an invalid value.". 

It turns out the new certificate we were referencing was already newer than the certificate stored in the previously referenced secret. Apparently this gives the very cryptic error message above. The solution is to make sure the certicates are the same, before deploying the ARM template with the updated secretname.

You can download the original certificate as .pfx with some Powershell you can find [here](https://coombes.nz/blog/azure-keyvault-export-certificate/).

After that import the .pfx into the new KeyVault certificate with the Import-AzureKeyVaultCertificate cmdlet. 

Now you can redeploy the ARM template to update the keyVaultSecretName. After that you can update the certificate again.