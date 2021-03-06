---
title: Upgrade auf Azure SQL Data Warehouse der neuesten Generation | Microsoft-Dokumentation
description: Führen Sie ein Upgrade für Azure SQL Data Warehouse auf die Azure-Hardware- und -Speicherarchitektur der neuesten Generation aus.
services: sql-data-warehouse
author: kevinvngo
manager: craigg-msft
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 04/17/2018
ms.author: kevin
ms.reviewer: igorstan
ms.openlocfilehash: 58d65ef05ed872bb357070de9866253baea5dc70
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/07/2018
ms.locfileid: "33777806"
---
# <a name="optimize-performance-by-upgrading-sql-data-warehouse"></a>Optimieren der Leistung durch ein Upgrade von SQL Data Warehouse
Führen Sie ein Upgrade für Azure SQL Data Warehouse auf die Azure-Hardware- und -Speicherarchitektur der neuesten Generation aus.

## <a name="why-upgrade"></a>Gründe für ein Upgrade
Sie können im Azure-Portal jetzt nahtlos ein Upgrade auf SQL Data Warehouse Gen2 durchführen. Falls Sie ein Data Warehouse der ersten Generation (Gen1) verwenden, wird das Upgrade empfohlen. Durch das Upgrade können Sie die neueste Generation der Azure-Hardware- und erweiterten Azure-Speicherarchitektur verwenden. Auf diese Weise profitieren Sie von höherer Leistung und Skalierbarkeit sowie unbegrenztem spaltenbasiertem Speicher. 

## <a name="applies-to"></a>Anwendungsbereich
Dieses Update gilt für Data Warehouses vom Typ „Gen1“.

## <a name="sign-in-to-the-azure-portal"></a>Melden Sie sich auf dem Azure-Portal an.

Melden Sie sich beim [Azure-Portal](https://portal.azure.com/) an.

## <a name="before-you-begin"></a>Voraussetzungen
> [!NOTE]
> Wenn sich Ihr Data Warehouse der ersten Generation (Gen1) nicht in einer Region befindet, in der Gen2 verfügbar ist, können Sie per PowerShell eine [Geowiederherstellung auf Gen2](https://docs.microsoft.com/en-us/azure/sql-data-warehouse/sql-data-warehouse-restore-database-powershell#restore-from-an-azure-geographical-region) in einer unterstützten Region durchführen.
> 
>

1. Wenn das Data Warehouse vom Typ „Gen1, für das ein Upgrade durchgeführt werden soll, angehalten ist, [müssen Sie das Data Warehouse fortsetzen](pause-and-resume-compute-portal.md).
2. Es tritt eine Ausfallzeit von einigen Minuten auf. 



## <a name="start-the-upgrade"></a>Starten des Upgrades

1. Navigieren Sie im Azure-Portal zu Ihrem Data Warehouse der ersten Generation, und klicken Sie auf **Upgrade auf Gen2**: ![Upgrade_1](./media/sql-data-warehouse-upgrade-to-latest-generation/Upgrade_to_Gen2_1.png)

2. Wählen Sie standardmäßig basierend auf Ihrer aktuellen Leistungsstufe für „Gen 1“ **die vorgeschlagene Leistungsstufe** für das Data Warehouse aus. Verwenden Sie hierfür die folgende Zuordnung:
    
| Gen1 | Gen2 |
| :----------------------: | :-------------------: |
|      DW100 – DW1000      |        DW1000c        |
|          DW1200          |        DW1500c        |
|          DW1500          |        DW1500c        |
|          DW2000          |        DW2000c        |
|          DW3000          |        DW3000c        |
|          DW6000          |        DW6000c        |


3. Stellen Sie sicher, dass die Ausführung Ihrer Workload abgeschlossen ist und die Workload stillgelegt wurde, bevor Sie das Upgrade durchführen. Bevor Ihr Data Warehouse als „Gen2“ wieder online geschaltet wird, tritt eine Ausfallzeit von einigen Minuten auf. Klicken Sie auf **Upgrade durchführen**. Die Leistungsstufe „Gen2“ ist in der Vorschauphase gegenwärtig zum halben Preis erhältlich:
    
    ![Upgrade_2](./media/sql-data-warehouse-upgrade-to-latest-generation/Upgrade_to_Gen2_2.png)

4. **Überwachen Sie Ihr Upgrade**, indem Sie den Status im Azure-Portal überprüfen:

   ![Upgrade3](./media/sql-data-warehouse-upgrade-to-latest-generation/Upgrade_to_Gen2_3.png)
   
   Der erste Schritt des Upgradeprozesses erfolgt im Skalierungsvorgang („Upgrade – Offline“), in dem alle Sitzungen gelöscht und Verbindungen getrennt werden. 
   
   Der zweite Schritt des Upgradeprozesses ist die Datenmigration („Upgrade – Online“). Die Datenmigration ist ein im Hintergrund ausgeführter allmählicher Onlineprozess, der spaltenbasierte Daten langsam aus der alten Speicherarchitektur in die neue Speicherarchitektur verschiebt, indem der lokale SSD-Cache genutzt wird. Während dieser Zeit ist Ihr Data Warehouse für Abfrage- und Ladevorgänge online. All Ihre Daten stehen für Abfragen zur Verfügung, unabhängig davon, ob sie bereits migriert wurden oder nicht. Die Datenmigration erfolgt mit wechselnder Geschwindigkeit je nach Größe Ihrer Daten, Ihrer Leistungsstufe und der Anzahl Ihrer Columnstore-Segmente. 

5. **Suchens Sie nach Ihrem Data Warehouse vom Typ „Gen2“**, indem Sie das SQL-Datenbank-Blatt für die Suche verwenden. 

> [!NOTE]
> Derzeit besteht ein Problem, bei dem Data Warehouses vom Typ „Gen2“ auf dem SQL Data Warehouse-Blatt für die Suche nicht angezeigt werden. Verwenden Sie stattdessen das SQL-Datenbank-Blatt für die Suche, um Ihr auf Gen2 aktualisiertes Data Warehouse zu ermitteln. Wir arbeiten mit Hochdruck an der Lösung dieses Problems.
> 

6. **Optionale Empfehlung**: Um den Hintergrundprozess der Datenmigration zu beschleunigen, empfiehlt es sich, die Datenverschiebung sofort zu erzwingen, indem Sie [ALTER INDEX REBUILD](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-tables-index) in allen Columnstore-Tabellen mit einem größeren SLO-Wert und einer größeren Ressourcenklasse auszuführen. Der Vorgang erfolgt offline, im Gegensatz zum allmählich ausgeführten Hintergrundprozess. Die Datenmigration läuft jedoch wesentlich schneller ab, wenn Sie die neue erweiterte Speicherarchitektur vollständig nutzen können, sobald diese mit hochwertigen Zeilengruppen fertig erstellt wurde. 

Die folgende Abfrage generiert die erforderlichen ALTER INDEX REBUILD-Befehle, um den Datenmigrationsprozess zu beschleunigen:

```sql
SELECT 'ALTER INDEX [' + idx.NAME + '] ON [' 
       + Schema_name(tbl.schema_id) + '].[' 
       + Object_name(idx.object_id) + '] REBUILD ' + ( CASE 
                                                         WHEN ( 
                                                     (SELECT Count(*) 
                                                      FROM   sys.partitions 
                                                             part2 
                                                      WHERE  part2.index_id 
                                                             = idx.index_id 
                                                             AND 
                                                     idx.object_id = 
                                                     part2.object_id) 
                                                     > 1 ) THEN 
              ' PARTITION = ' 
              + Cast(part.partition_number AS NVARCHAR(256)) 
              ELSE '' 
                                                       END ) + '; SELECT ''[' + 
              idx.NAME + '] ON [' + Schema_name(tbl.schema_id) + '].[' + 
              Object_name(idx.object_id) + '] ' + ( 
              CASE 
                WHEN ( (SELECT Count(*) 
                        FROM   sys.partitions 
                               part2 
                        WHERE 
                     part2.index_id = 
                     idx.index_id 
                     AND idx.object_id 
                         = part2.object_id) > 1 ) THEN 
              ' PARTITION = ' 
              + Cast(part.partition_number AS NVARCHAR(256)) 
              + ' completed'';' 
              ELSE ' completed'';' 
                                                    END ) 
FROM   sys.indexes idx 
       INNER JOIN sys.tables tbl 
               ON idx.object_id = tbl.object_id 
       LEFT OUTER JOIN sys.partitions part 
                    ON idx.index_id = part.index_id 
                       AND idx.object_id = part.object_id 
WHERE  idx.type_desc = 'CLUSTERED COLUMNSTORE'; 
```



## <a name="next-steps"></a>Nächste Schritte
Ihr aktualisiertes Data Warehouse ist online. Informationen zur optimalen Nutzung der erweiterten Architektur finden Sie unter [Ressourcenklassen für die Workloadverwaltung](resource-classes-for-workload-management.md).
 
