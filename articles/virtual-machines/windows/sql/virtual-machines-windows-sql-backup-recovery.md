---
title: Sichern und Wiederherstellen von SQL Server | Microsoft Docs
description: "Erläutert Überlegungen zur Sicherung und Wiederherstellung für SQL Server-Datenbanken auf virtuellen Azure-Computern."
services: virtual-machines-windows
documentationcenter: na
author: MikeRayMSFT
manager: craigg
editor: 
tags: azure-resource-management
ms.assetid: 95a89072-0edf-49b5-88ed-584891c0e066
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 11/15/2016
ms.author: mikeray
ms.openlocfilehash: 16fef048e7c795f3d21fbc4185f6ba31bbc885fb
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/21/2018
---
# <a name="backup-and-restore-for-sql-server-in-azure-virtual-machines"></a>Sicherung und Wiederherstellung für SQL Server auf virtuellen Azure-Computern
## <a name="overview"></a>Übersicht
Von Azure Storage werden drei Kopien jedes Azure-VM-Datenträgers verwaltet, um den Schutz vor Datenverlust oder physischer Datenbeschädigung zu gewährleisten. Anders als in einem lokalen Szenario müssen Sie sich also keine Sorgen in dieser Hinsicht machen. Sie sollten jedoch weiterhin Ihre SQL Server-Datenbanken sichern, um sie gegen Anwendungs- oder Benutzerfehler (z.B. Einfügen falscher Daten oder Löschen einer Tabelle) zu schützen und die Point-in-Time-Wiederherstellung zu ermöglichen.

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-both-include.md)]

Für SQL Server auf Azure-VMs können Sie systemeigene Sicherungs- und Wiederherstellungsverfahren mit angefügten Datenträgern als Ziel der Sicherungsdateien verwenden. Die Anzahl von Datenträgern, die an einen virtuellen Azure-Computer angefügt werden können, ist allerdings je nach [Größe des virtuellen Computers](../sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) begrenzt. Zudem muss der Mehraufwand für die Datenträgerverwaltung berücksichtigt werden.

Ab SQL Server 2014 können Sie Microsoft Azure Blob Storage für die Sicherung und Wiederherstellung verwenden. SQL Server 2016 bietet auch Erweiterungen für diese Option. Darüber hinaus bietet SQL Server 2016 für in Microsoft Azure Blob Storage gespeicherte Datenbankdateien eine Option für nahezu sofortige Sicherungen und schnelle Wiederherstellungen mithilfe von Azure-Momentaufnahmen. Dieser Artikel bietet eine Übersicht über diese Optionen. Weitere Informationen finden Sie unter [SQL Server-Sicherung und -Wiederherstellung mit dem Microsoft Azure Blob Storage Service](https://msdn.microsoft.com/library/jj919148.aspx).

> [!NOTE]
> Eine Beschreibung der Optionen zum Sichern sehr großer Datenbanken finden Sie im Blogbeitrag zu den [Sicherungsstrategien für mehrere Terabyte große SQL Server-Datenbanken für virtuelle Azure-Computer](http://blogs.msdn.com/b/igorpag/archive/2015/07/28/multi-terabyte-sql-server-database-backup-strategies-for-azure-virtual-machines.aspx).
> 
> 

Die folgenden Abschnitte enthalten Informationen zu den verschiedenen Versionen von SQL Server, die auf einem virtuellen Azure-Computer unterstützt werden.

## <a name="sql-server-virtual-machines"></a>Virtuelle SQL Server-Computer
Wenn die SQL Server-Instanz auf einem virtuellen Azure-Computer ausgeführt wird, befinden sich die Datenbankdateien bereits auf Datenträgern in Azure. Diese Datenträger befinden sich in Azure Blob Storage. Die Gründe für das Sichern der Datenbank und der von Ihnen verwendete Ansatz ändern sich also leicht. Stellen Sie sich einmal Folgendes vor: 

* Sie müssen keine Datenbanksicherungen mehr durchführen, um Ihre Daten vor Hardwarefehlern oder Fehlern auf Speichermedien zu schützen, da Microsoft Azure diesen Schutz im Rahmen des Microsoft Azure-Diensts bereitstellt.
* Sie müssen jedoch weiterhin Datenbanksicherungen zum Schutz vor Benutzerfehlern oder zu Archivierungszwecken, aus gesetzlichen Gründen oder zu administrativen Zwecken durchführen.
* Sie können die Sicherungsdatei direkt in Azure speichern. Weitere Informationen finden Sie in den folgenden Abschnitten mit Hinweisen für die verschiedenen Versionen von SQL Server.

## <a name="sql-server-2016"></a>SQL Server 2016
Microsoft SQL Server 2016 unterstützt die [Sichern und Wiederherstellen mit Azure-Blobs](https://msdn.microsoft.com/library/jj919148.aspx) -Funktionen aus SQL Server 2014. Die Version enthält aber auch die folgenden Erweiterungen:

| 2016-Erweiterung | Details |
| --- | --- |
| **Striping** |Für Sicherungen in Microsoft Azure Blob Storage unterstützt SQL Server 2016 das Sichern in mehreren Blobs, um die Sicherung großer Datenbanken mit bis zu 12,8 TB zu ermöglichen. |
| **Momentaufnahmesicherung** |Durch die Verwendung von Azure-Momentaufnahmen ermöglicht die Dateimomentaufnahme-Sicherung von SQL Server nahezu sofortige Backups und schnelle Wiederherstellungen für Datenbankdateien, die mit dem Azure Blob Storage-Dienst gespeichert werden. Sicherungs- und Wiederherstellungsrichtlinien lassen sich dank dieser Funktion vereinfachen. Die Dateimomentaufnahme-Sicherung unterstützt auch Point-in-Time-Wiederherstellungen. Weitere Informationen finden Sie im Artikel zu [Momentaufnahmesicherungen für Datenbankdateien in Azure](https://msdn.microsoft.com/library/mt169363%28v=sql.130%29.aspx). |
| **Verwaltete Sicherungsplanung** |SQL Server Managed Backup für Azure unterstützt jetzt benutzerdefinierte Zeitpläne. Weitere Informationen finden Sie unter [SQL Server Managed Backup für Microsoft Azure](https://msdn.microsoft.com/library/dn449496.aspx). |

Eine praxisnahe Einführung in die Funktionen von SQL Server 2016 für Azure Blob Storage bietet das [Tutorial zur Verwendung des Microsoft Azure Blob Storage-Diensts mit SQL Server 2016-Datenbanken](https://msdn.microsoft.com/library/dn466438.aspx).

## <a name="sql-server-2014"></a>SQL Server 2014
SQL Server 2014 enthält die folgenden Erweiterungen:

1. **Sichern und Wiederherstellen in Azure**:
   
   * *SQL Server-URL-Sicherung* wird jetzt in SQL Server Management Studio unterstützt. Die Option zum Sichern in Azure ist jetzt bei der Verwendung der Aufgabe „Sichern oder Wiederherstellen“ oder des Wartungsplanungs-Assistenten in SQL Server Management Studio verfügbar. Weitere Informationen finden Sie unter [SQL Server-URL-Sicherung](https://msdn.microsoft.com/library/jj919148%28v=sql.120%29.aspx).
   * *SQL Server Managed Backup für Azure* verfügt über neue Funktionen, die eine automatisierte Sicherungsverwaltung ermöglichen. Diese Funktionen erleichtern insbesondere die Automatisierung der Sicherungsverwaltung für SQL Server 2014-Instanzen auf einem Azure-Computer. Weitere Informationen finden Sie unter [SQL Server Managed Backup für Microsoft Azure](https://msdn.microsoft.com/library/dn449496%28v=sql.120%29.aspx).
   * *Automatisierte Sicherung* bietet zusätzliche Automatisierungsfunktionen, mit denen die *verwaltete SQL Server-Sicherung in Azure* automatisch für alle vorhandenen und neuen Datenbanken für einen virtuellen SQL Server-Computer in Azure aktiviert werden kann. Weitere Informationen finden Sie unter [Automatisierte Sicherung für SQL Server in Azure Virtual Machines](virtual-machines-windows-sql-automated-backup.md).
   * Eine Übersicht über alle Optionen für die SQL Server 2014-Sicherung in Azure finden Sie unter [SQL Server-Sicherung und -Wiederherstellung mit dem Microsoft Azure Blob Storage-Dienst](https://msdn.microsoft.com/library/jj919148%28v=sql.120%29.aspx).
2. **Verschlüsselung**: SQL Server 2014 unterstützt das Verschlüsseln von Daten beim Erstellen einer Sicherung. Es werden mehrere Verschlüsselungsalgorithmen und die Verwendung eines Zertifikats oder asymmetrischen Schlüssels unterstützt. Weitere Informationen finden Sie unter [Sicherungsverschlüsselung](https://msdn.microsoft.com/library/dn449489%28v=sql.120%29.aspx).

## <a name="sql-server-2012"></a>SQL Server 2012
Ausführliche Informationen zur SQL Server-Sicherung und -Wiederherstellung in SQL Server 2012 finden Sie unter [Sichern und Wiederherstellen von SQL Server-Datenbanken (SQL Server 2012)](https://msdn.microsoft.com/library/ms187048%28v=sql.110%29.aspx).

Ab SQL Server 2012 SP1, kumulatives Update 2, ist die Sicherung und Wiederherstellung im bzw. aus dem Azure Blob Storage-Dienst möglich. Diese Erweiterung kann verwendet werden, um SQL Server-Datenbanken in einer SQL Server-Instanz auf einem virtuellen Azure-Computer oder einer lokalen Instanz zu sichern. Weitere Informationen finden Sie im Artikel zu [SQL Server-Sicherung und -Wiederherstellung mit dem Azure Blob Storage-Dienst](https://msdn.microsoft.com/library/jj919148%28v=sql.110%29.aspx).

Zu den Vorteilen des Azure Blob Storage-Diensts zählen die Möglichkeit, die Beschränkung auf 16 angefügte Datenträger zu umgehen, die einfache Verwaltung und die direkte Verfügbarkeit der Sicherungsdatei für eine andere SQL Server-Instanz auf einem virtuellen Azure-Computer oder eine lokale Instanz zur Migration oder Notfallwiederherstellung. Eine vollständige Liste der Vorteile, die die Verwendung des Azure Blob Storage-Diensts für SQL Server-Sicherungen bietet, finden Sie im Abschnitt *Vorteile* unter [SQL Server-Sicherung und -Wiederherstellung mit dem Azure Blob Storage-Dienst](https://msdn.microsoft.com/library/jj919148%28v=sql.110%29.aspx).

Bewährte Methoden, Empfehlungen und Informationen zur Problembehandlung finden Sie unter [Bewährte Methoden für die Sicherung und Wiederherstellung (Azure Blob Storage-Dienst)](https://msdn.microsoft.com/library/jj919149%28v=sql.110%29.aspx).

## <a name="sql-server-2008"></a>SQL Server 2008
Informationen zur SQL Server-Sicherung und -Wiederherstellung in SQL Server 2008 R2 finden Sie unter [Sichern und Wiederherstellen von Datenbanken in SQL Server (SQL Server 2008 R2)](https://msdn.microsoft.com/library/ms187048%28v=sql.105%29.aspx).

Informationen zur SQL Server-Sicherung und -Wiederherstellung in SQL Server 2008 finden Sie unter [Sichern und Wiederherstellen von Datenbanken in SQL Server (SQL Server 2008)](https://msdn.microsoft.com/library/ms187048%28v=sql.100%29.aspx).

## <a name="next-steps"></a>Nächste Schritte
Wenn Sie an der Planung Ihrer Bereitstellung von SQL Server auf einem virtuellen Azure-Computer arbeiten, finden Sie im folgenden Tutorial Informationen zur Bereitstellung: [Bereitstellen eines virtuellen SQL Server-Computers mit Azure Resource Manager](virtual-machines-windows-portal-sql-server-provision.md).

Obwohl Sie Ihre Daten durch Sicherung und Wiederherstellung migrieren können, sind möglicherweise einfachere Migrationspfade für SQL Server auf einer Azure-VM verfügbar. Eine vollständige Erläuterung der Migrationsoptionen und Empfehlungen finden Sie unter [Migrieren einer Datenbank zu SQL Server auf einem virtuellen Azure-Computer](virtual-machines-windows-migrate-sql.md).

Lesen Sie auch die weiteren [Ressourcen für die Ausführung von SQL Server in Azure Virtual Machines](virtual-machines-windows-sql-server-iaas-overview.md).

