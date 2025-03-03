---
title: Configure Question Answering service
description: This document outlines advanced configurations for custom question answering enabled resources.
ms.service: cognitive-services
ms.subservice: language-service
ms.topic: conceptual
ms.date: 11/02/2021
ms.custom: language-service-question-answering, ignite-fall-2021
---

# Configure custom question answering enabled resources

You can configure question answering to use a different Cognitive Search resource.

## Change Cognitive Search resource

> [!WARNING]
> If you change the Azure Search service associated with your language resource, you will lose access to all the projects/knowledge bases already present in it. Make sure you export the existing projects before you change the Azure Search service.

If you create a language resource and its dependencies (such as Search) through the Azure portal, a Search service is created for you and linked to the language resource. After these resources are created, you can update the Search resource in the **Features** tab.

1.  Go to your language resource in the Azure portal.

2.  Select **Features** and select the Azure Cognitive Search service you want to link with your language resource.

> [!div class="mx-imgBorder"]
> ![Add QnA to TA](../media/configure-resources/update-custom-feature.png)

3.  Select **Save**.

## Next steps

* [Encrypt data at rest](./encrypt-data-at-rest.md)
