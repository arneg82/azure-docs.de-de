---
title: Azure Service Fabric-Ereignisanalyse mit Log Analytics | Microsoft-Dokumentation
description: Erfahren Sie, wie Sie Ereignisse unter Verwendung von Log Analytics zur Überwachung und Diagnose von Azure Service Fabric-Clustern visualisieren und analysieren.
services: service-fabric
documentationcenter: .net
author: srrengar
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/16/2018
ms.author: srrengar
ms.openlocfilehash: b51f7dc43f390152b2b0be223541e381bbddd3c6
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/16/2018
---
# <a name="event-analysis-and-visualization-with-log-analytics"></a>Ereignisanalyse und -visualisierung mit Log Analytics

Log Analytics, auch als OMS (Operations Management Suite) bezeichnet, ist eine Sammlung von Verwaltungsdiensten, die die Überwachung und Diagnose für in der Cloud gehostete Anwendungen und Dienste unterstützen. In diesem Artikel wird beschrieben, wie Sie Abfragen in Log Analytics ausführen, um einen Einblick in den Cluster zu erhalten und Probleme zu beheben. Folgende allgemeine Fragen werden berücksichtigt:

* Wie behebe ich Probleme bei Integritätsereignissen?
* Wie weiß ich, dass ein Knoten ausfällt?
* Wie finde ich heraus, ob die Dienste der Anwendung gestartet oder beendet wurden?

## <a name="log-analytics-workspace"></a>Log Analytics-Arbeitsbereich

Log Analytics sammelt Daten von verwalteten Ressourcen, z.B. von einer Azure Storage-Tabelle oder einem Agent, und verwaltet sie in einem zentralen Repository. Die Daten können dann für Analyse, Warnungen und Visualisierung oder für den weiteren Export verwendet werden. Log Analytics unterstützt Ereignisse, Leistungsdaten oder andere benutzerdefinierten Daten. Überprüfen Sie die [Schritte zum Konfigurieren der Diagnoseerweiterung zum Aggregieren von Ereignissen](service-fabric-diagnostics-event-aggregation-wad.md) und die [Schritte zum Erstellen eines Log Analytics-Arbeitsbereichs zum Lesen der Ereignisse im Speicher](service-fabric-diagnostics-oms-setup.md), um sicherzustellen, dass Daten in Log Analytics übertragen werden.

Nachdem Daten von Log Analytics empfangen wurden, verfügt Azure über mehrere *Verwaltungslösungen*. Dabei handelt es sich um vorkonfigurierte Lösungen zum Überwachen eingehender Daten, die an verschiedene Szenarien angepasst sind. Dazu gehören eine *Service Fabric-Analyselösung* und eine *Containerlösung*. Dies sind die beiden wichtigsten Lösungen für die Diagnose und Überwachung bei Verwendung von Service Fabric-Clustern. In diesem Artikel wird die Verwendung der Service Fabric-Analyse-Lösung beschrieben, die im Arbeitsbereich erstellt wird.

## <a name="access-the-service-fabric-analytics-solution"></a>Zugriff auf die Service Fabric-Analyse-Lösung

1. Navigieren Sie zu der Ressourcengruppe, in der Sie die Lösung der Service Fabric-Analyse erstellt haben. Wählen Sie die Ressource **ServiceFabric\<OMS-Arbeitsbereichsname\>** aus, und wechseln Sie zur zugehörigen Übersichtsseite.

2. Klicken Sie auf der Übersichtsseite auf den Link oben, um zum OMS-Portal zu wechseln.

    ![Link zum OMS-Portal](media/service-fabric-diagnostics-event-analysis-oms/oms-portal-link.png)

3. Im OMS-Portal können Sie nun die Lösungen anzeigen, die Sie aktiviert haben. Klicken Sie auf das Diagramm mit dem Titel „Service Fabric“ (erste Abbildung unten), um zur Service Fabric-Lösung (zweite Abbildung unten) zu wechseln.

    ![OMS – SF-Lösung](media/service-fabric-diagnostics-event-analysis-oms/oms-workspace-all-solutions.png)

    ![OMS – SF-Lösung](media/service-fabric-diagnostics-event-analysis-oms/service-fabric-analytics-new.png)

In der Abbildung oben ist die Startseite der Service Fabric-Analyse-Lösung dargestellt. Es handelt sich um eine Momentaufnahmenansicht der Vorgänge im Cluster. Wenn Sie die Diagnose bei der Clustererstellung aktiviert haben, werden folgende Ereignisse angezeigt: 

* [Betriebskanal:](service-fabric-diagnostics-event-generation-operational.md) Vorgänge einer höheren Ebene, die von der Service Fabric-Plattform (Sammlung von Systemdiensten) durchgeführt werden.
* [Ereignisse des Reliable Actors-Programmiermodells](service-fabric-reliable-actors-diagnostics.md)
* [Ereignisse des Reliable Services-Programmiermodells](service-fabric-reliable-services-diagnostics.md)

>[!NOTE]
>Neben den Ereignissen des Betriebskanals können durch [Aktualisieren der Konfiguration der Diagnoseerweiterung](service-fabric-diagnostics-event-aggregation-wad.md#log-collection-configurations) detailliertere Systemereignisse erfasst werden.

### <a name="view-operational-events-including-actions-on-nodes"></a>Anzeigen von Betriebsereignissen einschließlich Aktionen auf Knoten

1. Klicken Sie auf der Seite „Service Fabric-Analyse“ im OMS-Portal auf das Diagramm für den Betriebskanal.

    ![OMS – SF-Lösung: Betriebskanal](media/service-fabric-diagnostics-event-analysis-oms/service-fabric-analytics-new-operational.png)

2. Klicken Sie auf „Tabelle“, um die Ereignisse in einer Liste anzuzeigen. In der Liste werden alle gesammelten Systemereignisse angezeigt. Zur Referenz: Diese stammen aus WADServiceFabricSystemEventsTable im Azure Storage-Konto. Auch die Reliable Services- und Reliable Actors-Ereignisse, die als Nächstes angezeigt werden, stammen von diesen entsprechenden Tabellen.
    
    ![OMS-Abfrage – Betriebskanal](media/service-fabric-diagnostics-event-analysis-oms/oms-query-operational-channel.png)

Alternativ können Sie auf die Lupe links klicken und mit der Abfragesprache Kusto die gewünschten Ereignisse suchen. Um beispielsweise alle Aktionen zu suchen, die im Cluster für Knoten durchgeführt wurden, können Sie die folgende Abfrage verwenden. Die unten verwendeten Ereignis-IDs finden Sie in der [Referenz der Betriebskanalprotokolle](service-fabric-diagnostics-event-generation-operational.md).

```kusto
ServiceFabricOperationalEvent
| where EventId < 25627 and EventId > 25619 
```

Sie können Abfragen für viele weitere Felder durchführen, z. B. für bestimmte Knoten (Computer), den Systemdienst (TaskName) usw.

### <a name="view-service-fabric-reliable-service-and-actor-events"></a>Anzeigen von Service Fabric Reliable Service- und Reliable Actor-Ereignissen

1. Klicken Sie auf der Seite „Service Fabric-Analyse“ im OMS-Portal auf das Diagramm für Reliable Services.

    ![OMS – SF-Lösung: Reliable Services](media/service-fabric-diagnostics-event-analysis-oms/service-fabric-analytics-reliable-services.png)

2. Klicken Sie auf „Tabelle“, um die Ereignisse in einer Liste anzuzeigen. Hier werden Ereignisse aus den zuverlässigen Diensten angezeigt. Es werden unterschiedliche Ereignisse für den Start und das Ende des runasync-Diensts angezeigt. Dies erfolgt normalerweise bei Bereitstellungen und Upgrades. 

    ![OMS-Abfrage – Reliable Services](media/service-fabric-diagnostics-event-analysis-oms/oms-query-reliable-services.png)

Reliable Actor-Ereignisse können auf ähnliche Weise angezeigt werden. Um ausführlichere Ereignisse für Reliable Actors zu konfigurieren, müssen Sie `scheduledTransferKeywordFilter` in der Konfiguration für die Diagnoseerweiterung ändern (siehe unten). Details zu den entsprechenden Werten finden Sie in der [Referenz der Reliable Actors-Ereignisse](service-fabric-reliable-actors-diagnostics.md#keywords).

```json
"EtwEventSourceProviderConfiguration": [
                {
                    "provider": "Microsoft-ServiceFabric-Actors",
                    "scheduledTransferKeywordFilter": "1",
                    "scheduledTransferPeriod": "PT5M",
                    "DefaultEvents": {
                    "eventDestination": "ServiceFabricReliableActorEventTable"
                    }
                },
```

Die Abfragesprache Kusto ist leistungsstark. Eine weitere nützliche Abfrage ist die Suche der Knoten, die die meisten Ereignisse generieren. In der Abfrage im folgenden Screenshot ist das aggregierte Reliable Services-Ereignis mit dem spezifischen Dienst und Knoten zu sehen.

![OMS-Abfrage – Ereignisse pro Knoten](media/service-fabric-diagnostics-event-analysis-oms/oms-query-events-per-node.png)

## <a name="next-steps"></a>Nächste Schritte

* Um die Infrastrukturüberwachung, d.h. Leistungsindikatoren, zu aktivieren, wechseln Sie zu [Hinzufügen des OMS-Agents](service-fabric-diagnostics-oms-agent.md). Der Agent sammelt Leistungsindikatoren und fügt sie dem vorhandenen Arbeitsbereich zu.
* Für lokale Cluster bietet die OMS ein Gateway (HTTP-Weiterleitungsproxy), über das Daten an die OMS gesendet werden können. Weitere Informationen dazu finden Sie unter [Verbinden von Computern ohne Internetzugriff mit der OMS über das OMS-Gateway](../log-analytics/log-analytics-oms-gateway.md).
* Konfigurieren Sie die OMS für die Einrichtung von [automatisierten Warnungen](../log-analytics/log-analytics-alerts.md) bei der Erkennung und Diagnose.
* Machen Sie sich mit den Features zur [Protokollsuche und -abfrage](../log-analytics/log-analytics-log-searches.md) in Log Analytics vertraut.
* Eine ausführlichere Übersicht über Log Analytics und die zugehörigen Optionen finden Sie unter [Was ist Log Analytics?](../operations-management-suite/operations-management-suite-overview.md)
