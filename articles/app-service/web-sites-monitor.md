---
title: "Überwachen von Apps in Azure App Service | Microsoft Docs"
description: "In diesem Thema wird die Überwachung von Apps in Azure App Service mit dem Azure-Portal beschrieben."
services: app-service
documentationcenter: 
author: btardif
manager: erikre
editor: 
ms.assetid: d273da4e-07de-48e0-b99d-4020d84a425e
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/28/2017
ms.author: byvinyal
ms.openlocfilehash: fdc4329806d416811352d0d4dbc8dd3bce25aa0b
ms.sourcegitcommit: 088a8788d69a63a8e1333ad272d4a299cb19316e
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/27/2018
---
# <a name="how-to-monitor-apps-in-azure-app-service"></a>Gewusst wie: Überwachen von Apps in Azure App Service
[App Service](http://go.microsoft.com/fwlink/?LinkId=529714) stellt integrierte Überwachungsfunktionen im [Azure-Portal](https://portal.azure.com) bereit.
Das Azure-Portal bietet auch die Möglichkeit zum Überprüfen von **Kontingenten** und **Metriken** für eine App und den App Service-Plan, zum Einrichten von **Warnungen** und sogar zum automatischen **Skalieren** basierend auf diesen Metriken.

[!INCLUDE [app-service-web-to-api-and-mobile](../../includes/app-service-web-to-api-and-mobile.md)]

## <a name="understanding-quotas-and-metrics"></a>Grundlagen von Kontingenten und Metriken
### <a name="quotas"></a>Kontingente
In App Service gehostete Anwendungen unterliegen bestimmten *Grenzwerten* in Bezug auf die verwendbaren Ressourcen. Die Grenzwerte werden mit dem **App Service-Plan** definiert, der der App zugeordnet ist.

[!INCLUDE [app-service-dev-test-note](../../includes/app-service-dev-test-note.md)]

Wenn die Anwendung unter einem Plan vom Typ **Free** oder **Shared** gehostet wird, werden die Grenzwerte für die Ressourcen, die von der App verwendet werden können, mit **Kontingenten** definiert.

Wenn die Anwendung unter einem Plan vom Typ **Basic**, **Standard** oder **Premium** gehostet wird, werden die Grenzwerte der Ressourcen, die verwendet werden können, über die Elemente **Größe** (Klein, Mittel, Groß) und **Instanzanzahl** (1, 2, 3, ...) des **App Service-Plans** festgelegt.

**Kontingente** für **Free**- oder **Shared**-Apps sind:

* **CPU(Short)**
  * Zulässige CPU-Menge für diese Anwendung in einem Fünf-Minuten-Intervall. Dieses Kontingent wird alle fünf Minuten zurückgesetzt.
* **CPU(Day)**
  * Zulässige CPU-Gesamtmenge für diese Anwendung für einen Tag. Dieses Kontingent wird alle 24 Stunden um Mitternacht (UTC) zurückgesetzt.
* **Memory**
  * Zulässige Gesamtmenge an Arbeitsspeicher für diese Anwendung.
* **Bandwidth**
  * Zulässige Gesamtmenge an ausgehender Bandbreite für diese Anwendung für einen Tag.
    Dieses Kontingent wird alle 24 Stunden um Mitternacht (UTC) zurückgesetzt.
* **Filesystem**
  * Gesamtbetrag des zulässigen Speichers.

Das einzige Kontingent, das für Apps gilt, die in den Plänen **Basic**, **Standard** und **Premium** gehostet werden, ist **Filesystem**.

Weitere Informationen zu den spezifischen Kontingenten, Grenzwerten und Features, die für die unterschiedlichen App Service-SKUs verfügbar sind, finden Sie hier: [Diensteinschränkungen des Azure-Abonnements](../azure-subscription-service-limits.md#app-service-limits).

#### <a name="quota-enforcement"></a>Kontingenterzwingung
Wenn eine Anwendung das Kontingent für **CPU(Short)**, **CPU(Day)** oder **Bandwidth** überschreitet, wird die Anwendung beendet, bis das Kontingent zurückgesetzt wird. Während dieses Zeitraums führen alle eingehenden Anforderungen zu einem **HTTP 403**-Fehler.
![][http403]

Wenn das Kontingent **Memory** der Anwendung überschritten wird, wird die Anwendung neu gestartet.

Wenn das Kontingent **Filesystem** überschritten wird, schlagen alle Schreibvorgänge fehl. Dies gilt auch für das Schreiben in Protokolle.

Kontingente können erhöht oder aus Ihrer App entfernt werden, indem Sie ein Upgrade für den App Service-Plan durchführen.

### <a name="metrics"></a>Metriken
**Metriken** liefern Informationen zur App oder zum Verhalten des App Service-Plans.

Für eine **Anwendung**lauten die verfügbaren Metriken:

* **Durchschnittliche Antwortzeit**
  * Die durchschnittliche Zeit in Millisekunden, die die App zum Bereitstellen von Anforderungen benötigt.
* **Durchschnittlicher Arbeitssatz für Arbeitsspeicher**
  * Die durchschnittliche Arbeitsspeichermenge in MiBs, die von der Anwendung verbraucht wird.
* **CPU-Zeit**
  * Die CPU-Menge in Sekunden, die von der App verbraucht wird. Weitere Informationen zu diesen Metriken finden Sie unter [CPU-Zeit und CPU-Prozentsatz](#cpu-time-vs-cpu-percentage).
* **Eingehende Daten**
  * Die Menge an eingehender Bandbreite in MiBs, die von der App verbraucht wird.
* **Ausgehende Daten**
  * Die Menge an ausgehender Bandbreite in MiBs, die von der App verbraucht wird.
* **HTTP 2xx**
  * Anzahl von Anforderungen, die zu einem HTTP-Statuscode >= 200 und < 300 führen.
* **HTTP 3xx**
  * Anzahl von Anforderungen, die zu einem HTTP-Statuscode >= 300 und < 400 führen.
* **HTTP 401**
  * Anzahl von Anforderungen, die zu einem HTTP 401-Statuscode führen.
* **HTTP 403**
  * Anzahl von Anforderungen, die zu einem HTTP 403-Statuscode führen.
* **HTTP 404**
  * Anzahl von Anforderungen, die zu einem HTTP 404-Statuscode führen.
* **HTTP 406**
  * Anzahl von Anforderungen, die zu einem HTTP 406-Statuscode führen.
* **HTTP 4xx**
  * Anzahl von Anforderungen, die zu einem HTTP-Statuscode >= 400 und < 500 führen.
* **HTTP-Serverfehler**
  * Anzahl von Anforderungen, die zu einem HTTP-Statuscode >= 500 und < 600 führen.
* **Arbeitssatz für Arbeitsspeicher**
  * Aktuelle Menge von Arbeitsspeicher in MiBs, die von der App verwendet wird.
* **Anforderungen**
  * Gesamtzahl von Anforderungen, unabhängig vom sich ergebenden HTTP-Statuscode.

Für einen **App Service-Plan**lauten die verfügbaren Metriken:

> [!NOTE]
> App Service-Planmetriken sind nur für Pläne in den Tarifen **Basic**, **Standard** und **Premium** verfügbar.
> 
> 

* **CPU-Prozentsatz**
  * Die durchschnittliche CPU-Nutzung über alle Instanzen des Plans hinweg.
* **Arbeitsspeicherprozentsatz**
  * Die durchschnittliche Arbeitsspeichernutzung über alle Instanzen des Plans hinweg.
* **Eingehende Daten**
  * Die durchschnittliche eingehende Bandbreite, die für alle Instanzen des Plans verwendet wird.
* **Ausgehende Daten**
  * Die durchschnittliche ausgehende Bandbreite, die für alle Instanzen des Plans verwendet wird.
* **Länge der Datenträgerwarteschlange**
  * Die durchschnittliche Anzahl von Lese- und Schreibanforderungen, die im Speicher in die Warteschlange eingereiht wurden. Ein hoher Wert für die Länge der Datenträgerwarteschlange ist ein Anzeichen dafür, dass eine Anwendung aufgrund einer übermäßigen E/A-Aktivität des Datenträgers verlangsamt wird.
* **Länge der HTTP-Warteschlange**
  * Die durchschnittliche Anzahl von HTTP-Anforderungen, die sich in der Warteschlange befinden müssen, bevor sie erfüllt werden. Eine hohe oder zunehmende HTTP-Warteschlangenlänge deutet darauf hin, dass für einen Plan eine hohe Auslastung besteht.

### <a name="cpu-time-vs-cpu-percentage"></a>CPU-Zeit und CPU-Prozentsatz
<!-- To do: Fix Anchor (#CPU-time-vs.-CPU-percentage) -->

Es gibt zwei Metriken, die die CPU-Auslastung widerspiegeln: **CPU-Zeit** und **CPU-Prozentsatz**

**CPU-Zeit** ist für Apps hilfreich, die unter einem Plan vom Typ **Free** oder **Shared** gehostet werden, da eines der Kontingente basierend auf den von der App verbrauchten CPU-Minuten definiert ist.

**CPU-Prozentsatz** ist nützlich für Apps, die in den Plänen **Basic**, **Standard** und **Premium** gehostet werden, da sie horizontal hochskaliert werden können. Der CPU-Prozentsatz ist ein guter Indikator für die allgemeine Nutzung über alle Instanzen hinweg.

## <a name="metrics-granularity-and-retention-policy"></a>Granularität und Aufbewahrungsrichtlinien für Metriken
Metriken für eine Anwendung und einen App Service-Plan werden vom Dienst mit den folgenden Granularitäten und Aufbewahrungsrichtlinien protokolliert und aggregiert:

* Metriken mit der Granularität **Minute** werden für **30 Stunden** beibehalten.
* Metriken mit der Granularität **Stunde** werden für **30 Tage** beibehalten.
* Metriken mit der Granularität **Tag** werden für **30 Tage** beibehalten.

## <a name="monitoring-quotas-and-metrics-in-the-azure-portal"></a>Überwachen von Kontingenten und Metriken im Azure-Portal
Sie können den Status der unterschiedlichen **Kontingente** und **Metriken**, die sich auf eine Anwendung auswirken, im [Azure-Portal](https://portal.azure.com) überprüfen.

![][quotas]
**Kontingente** befindet sich unter „Einstellungen“ &gt; **Kontingente**. Auf der Benutzeroberfläche können Sie Folgendes überprüfen: (1) den Namen der Kontingente, (2) das Zurücksetzungsintervall, (3) die aktuelle Grenze und (4) den aktuellen Wert.

Auf ![][metrics]
**Metriken** kann direkt über die Ressourcenseite zugegriffen werden. Sie können das Diagramm auch wie folgt anpassen: (1) **Klicken** auf das Diagramm und (2) Option **Diagramm bearbeiten** wählen.
Hier können Sie (3) **Zeitbereich**, (4) **Diagrammtyp** und (5) **Metriken** für die Anzeige auswählen.  

Weitere Informationen zu Metriken finden Sie unter [Überwachen von Dienstmetriken](../monitoring-and-diagnostics/insights-how-to-customize-monitoring.md).

## <a name="alerts-and-autoscale"></a>Warnungen und automatische Skalierung
Metriken für eine App oder einen App Service-Plan können mit Warnungen verknüpft werden. Weitere Informationen hierzu finden Sie unter [Erstellen von Metrikwarnungen in Azure Monitor für Azure-Dienste – Azure-Portal](../monitoring-and-diagnostics/insights-alerts-portal.md).

App Service-Apps, die unter App Service-Plänen vom Typ „Basic“, „Standard“ oder „Premium“ gehostet werden, unterstützen die **automatische Skalierung**. Mithilfe der automatischen Skalierung können Sie Regeln zur Überwachung der Metriken des App Service-Plans konfigurieren. Über Regeln kann die Anzahl der Instanzen erhöht oder verringert werden, um bei Bedarf zusätzliche Ressourcen bereitzustellen. Außerdem helfen Regeln auch dabei, Geld zu sparen, wenn die Anwendung zu oft bereitgestellt wird. Weitere Informationen zur automatischen Skalierung finden Sie unter [Scale instance count manually or automatically](../monitoring-and-diagnostics/insights-how-to-scale.md) (Manuelles oder automatisches Skalieren der Instanzanzahl) und [Best practices for Azure Monitor autoscaling](../monitoring-and-diagnostics/insights-autoscale-best-practices.md) (Empfohlene Methoden für die automatische Skalierung in Azure Monitor).

> [!NOTE]
> Wenn Sie Azure App Service ausprobieren möchten, ehe Sie sich für ein Azure-Konto anmelden, können Sie unter [App Service testen](https://azure.microsoft.com/try/app-service/)sofort kostenlos eine kurzlebige Starter-Web-App in App Service erstellen. Keine Kreditkarte erforderlich, keine Verpflichtungen.
> 
> 

[fzilla]:http://go.microsoft.com/fwlink/?LinkId=247914
[vmsizes]:http://go.microsoft.com/fwlink/?LinkID=309169



<!-- Images. -->
[http403]: ./media/web-sites-monitor/http403.png
[quotas]: ./media/web-sites-monitor/quotas.png
[metrics]: ./media/web-sites-monitor/metrics.png
