---
title: Includedatei
description: Includedatei
services: active-directory
documentationcenter: dev-center-name
author: andretms
manager: mtillman
editor: ''
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/19/2018
ms.author: andret
ms.custom: include file
ms.openlocfilehash: 6871127ac138fb0113af73ca5222e3e2c2f99d7e
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/28/2018
---
# <a name="call-the-microsoft-graph-api-from-a-javascript-single-page-application-spa"></a>Aufrufen der Microsoft Graph-API aus einer JavaScript-Einzelseitenanwendung

Diese Anleitung veranschaulicht, wie eine JavaScript-Einzelseitenanwendung eine Anmeldung für private Konten und Geschäfts-, Schul- oder Unikonten durchführen, ein Zugriffstoken abrufen und die Microsoft Graph-API oder andere APIs aufrufen kann, die Zugriffstoken von einem Azure Active Directory v2-Endpunkt benötigen.

## <a name="how-the-sample-app-generated-by-this-guide-works"></a>Funktionsweise der über diesen Leitfaden generierten Beispiel-App

![Funktionsweise der über diesen Leitfaden generierten Beispiel-App](media/active-directory-develop-guidedsetup-javascriptspa-introduction/javascriptspa-intro.png)

<!--start-collapse-->
## <a name="more-information"></a>Weitere Informationen

Die über diesen Leitfaden erstellte Beispiel-App ermöglicht eine JavaScript-Einzelseitenanwendung das Abfragen der Microsoft Graph-API oder einer Web-API, die Token von einem Azure Active Directory v2-Endpunkt akzeptiert. In diesem Szenario wird nach der Benutzeranmeldung ein Zugriffstoken angefordert und über den Autorisierungsheader den HTTP-Anforderungen hinzugefügt. Tokenabruf und -verlängerung werden von der Microsoft Authentication Library (MSAL) verarbeitet.

<!--end-collapse-->

<!--start-collapse-->
## <a name="libraries"></a>Bibliotheken

In diesem Leitfaden werden die folgenden Bibliotheken verwendet:

|Bibliothek|BESCHREIBUNG|
|---|---|
|[msal.js](https://github.com/AzureAD/microsoft-authentication-library-for-js)|Microsoft Authentication Library für JavaScript-Vorschau|

> [!NOTE]
> *msal.js* hat als Ziel den *Azure Active Directory-v2-Endpunkt*, der privaten und Geschäfts-, Schul- oder Unikonten das Anmelden sowie das Abrufen von Token ermöglicht. Der *Azure Active Directory-v2-Endpunkt* hat [einige Einschränkungen](..\articles\active-directory\develop\active-directory-v2-limitations.md). Wenn Sie lediglich an Geschäfts-, Schul- oder Unikonten interessiert sind, verwenden Sie *adal.js* und den *v1-Endpunkt*. Um die Unterschiede zwischen v1- und v2-Endpunkten zu verstehen, lesen den [v1-v2-Vergleich](..\articles\active-directory\develop\active-directory-v2-compare.md).

<!--end-collapse-->
