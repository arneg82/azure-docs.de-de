---
title: Erstellen von Warnungen für Azure-Dienste – Azure-Portal | Microsoft-Dokumentation
description: E-Mails, Benachrichtigungen oder Automatisierung werden ausgelöst und URLs (Webhooks) von Websites aufgerufen, wenn die von Ihnen angegebenen Bedingungen erfüllt sind.
author: rboucher
manager: carmonm
editor: ''
services: monitoring-and-diagnostics
documentationcenter: monitoring-and-diagnostics
ms.assetid: f7457655-ced6-4102-a9dd-7ddf2265c0e2
ms.service: monitoring-and-diagnostics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/23/2016
ms.author: robb
ms.openlocfilehash: b0d938112aaea4d86dd539b53a1749cc800607a7
ms.sourcegitcommit: 20d103fb8658b29b48115782fe01f76239b240aa
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/03/2018
---
# <a name="create-classic-metric-alerts-in-azure-monitor-for-azure-services---azure-portal"></a>Erstellen klassischer Metrikwarnungen in Azure Monitor für Azure-Dienste – Azure-Portal
> [!div class="op_single_selector"]
> * [Portal](insights-alerts-portal.md)
> * [PowerShell](insights-alerts-powershell.md)
> * [BEFEHLSZEILENSCHNITTSTELLE (CLI)](insights-alerts-command-line-interface.md)
>
>

## <a name="overview"></a>Übersicht

> [!NOTE]
> In diesem Artikel erfahren Sie, wie Sie ältere klassische Metrikwarnungen erstellen. Azure Monitor unterstützt jetzt [neuere Metrikwarnungen](monitoring-near-real-time-metric-alerts.md). 
>
>

In diesem Artikel erfahren Sie, wie Sie klassische Azure-Metrikwarnungen über das Azure-Portal einrichten können. 

Sie können auf der Grundlage von Überwachungsmetriken für Ihre Azure-Services oder von Ereignissen, die bei diesen auftreten, eine Warnung empfangen.

* **Metrikwerte** : Die Warnung wird ausgelöst, wenn der Wert einer angegebenen Metrik einen von Ihnen festgelegten Schwellenwert in beliebiger Richtung überschreitet. Das Auslösen erfolgt sowohl, wenn die Bedingung erstmals erfüllt wird, als auch danach, wenn diese Bedingung nicht mehr erfüllt wird.    
* **Aktivitätsprotokollereignisse**: Eine Warnung kann für *jedes* Ereignis oder nur dann ausgelöst werden, wenn ein bestimmtes Ereignis auftritt. Erfahren Sie mehr über [Aktivitätsprotokollwarnungen](monitoring-activity-log-alerts.md).

Sie können konfigurieren, dass bei einer klassischen Metrikwarnung Folgendes erfolgt, wenn sie ausgelöst wird:

* Senden von E-Mail-Benachrichtigungen an den Dienstadministrator und Co-Administratoren
* Senden von E-Mal an weitere von Ihnen angegebene Adressen
* Aufrufen eines Webhooks
* Starten der Ausführung eines Azure-Runbooks (nur über das Azure-Portal)

Sie haben folgende Möglichkeiten zum Konfigurieren von klassischen Metrikwarnregeln und Abrufen zugehöriger Informationen:

* [Azure-Portal](insights-alerts-portal.md)
* [PowerShell](insights-alerts-powershell.md)
* [Befehlszeilenschnittstelle (CLI)](insights-alerts-command-line-interface.md)
* [Azure Monitor-REST-API](https://msdn.microsoft.com/library/azure/dn931945.aspx)

## <a name="create-an-alert-rule-on-a-metric-with-the-azure-portal"></a>Erstellen einer Warnungsregel anhand einer Metrik mit dem Azure-Portal
1. Suchen Sie im [Portal](https://portal.azure.com/)die Ressource, die Sie überwachen möchten, und wählen Sie sie aus.

2. Wählen Sie im Abschnitt „ÜBERWACHUNG“ die Option **Warnungen (klassisch)** aus. Text und Symbol können je nach Ressource geringfügig variieren. **Warnungen (klassisch)** ist möglicherweise auch unter **Warnungen** oder **Warnungsregeln** zu finden.

    ![Überwachung](./media/insights-alerts-portal/AlertRulesButton.png)

3. Wählen Sie den Befehl **Metrikwarnung hinzufügen (klassisch)** aus, und füllen Sie die Felder aus.

    ![Warnung hinzufügen](./media/insights-alerts-portal/AddAlertOnlyParamsPage.png)

4. **Benennen** Sie Ihre Warnungsregel, und wählen Sie eine **Beschreibung** aus, die auch in Benachrichtigungs-E-Mails angezeigt wird.

5. Wählen Sie die **Metrik** aus, die Sie überwachen möchten, und dann je einen Wert für **Bedingung** und **Schwellenwert** für die Metrik aus. Wählen Sie auch den **Zeitraum** der Metrikregel aus, der erfüllt sein muss, ehe die Warnung ausgelöst wird. Wenn Sie z.B. den Zeitraum „In den letzten 5 Minuten“ auswählen und die Warnung nach einer CPU-Auslastung von über 80 % sucht, wird die Warnung ausgelöst, wenn die CPU-Auslastung 5 Minuten durchgängig über 80 % lag. Nachdem der erste Trigger ausgelöst wurde, erfolgt ein erneutes Auslösen, wenn die CPU-Auslastung 5 Minuten unter 80% bleibt. Die Messung der CPU-Metrik erfolgt minütlich.

6. Aktivieren Sie **E-Mail-Besitzer...** , wenn Sie möchten, dass Administratoren und Co-Administratoren per E-Mail benachrichtigt werden, wenn die Warnung ausgelöst wird.

7. Wenn Sie möchten, dass bei Auslösen der Warnung eine Benachrichtigung an weitere E-Mail-Adressen gesendet wird, fügen Sie diese dem Feld **Zusätzliche Administrator-E-Mail-Adresse** hinzu. Trennen Sie mehrere E-Mail-Adressen durch Semikolons: *email@contoso.com;email2@contoso.com*

8. Fügen Sie in einen gültigen URI in das Feld **Webhook** ein, wenn dieser bei Auslösen der Warnung aufgerufen werden soll.

9. Wenn Sie Azure Automation verwenden, können Sie ein Runbook auswählen, das ausgeführt werden soll, sobald die Warnung ausgelöst wird.

10. Wählen Sie **OK** aus, wenn das Erstellen der Warnung abgeschlossen ist.   

Innerhalb weniger Minuten wird die Warnung aktiv und wie oben beschrieben ausgelöst.

## <a name="managing-your-alerts"></a>Verwalten von Warnungen
Nachdem Sie eine Warnung erstellt haben, können Sie sie auswählen und:

* ein Diagramm einblenden, das den Schwellenwert der Metrik und die tatsächlichen Werte vom Vortag zeigt.
* bearbeiten oder löschen.
* sie **deaktivieren** oder **aktivieren**, wenn Sie den Empfang von Benachrichtigungen zu dieser Warnung vorübergehend beenden oder fortsetzen möchten.

## <a name="next-steps"></a>Nächste Schritte
* [Übersicht über die Azure-Überwachung](monitoring-overview.md) , einschließlich der Typen von Informationen, die Sie sammeln und überwachen können.
* Informieren Sie sich ausführlicher über die [neueren Metrikwarnungen](monitoring-near-real-time-metric-alerts.md).
* Erfahren Sie mehr über das [Konfigurieren von Webhooks in Warnungen](insights-webhooks-alerts.md).
* Erfahren Sie mehr über das [Konfigurieren von Warnungen zu Aktivitätsprotokollereignissen](monitoring-activity-log-alerts.md).
* Erfahren Sie mehr zu [Azure Automation-Runbooks](../automation/automation-starting-a-runbook.md).
* Verschaffen Sie sich einen [Überblick über Diagnoseprotokolle](monitoring-overview-of-diagnostic-logs.md) , um detaillierte Hochfrequenzmetriken für Ihren Dienst zu erfassen.
* Verschaffen Sie sich einen Überblick über das [Sammeln von Dienstmetriken](insights-how-to-customize-monitoring.md) , um sicherzustellen, dass Ihr Dienst verfügbar und reaktionsfähig ist.
