---
title: Verwaltete Azure SQL-Datenbank-Instanz – Herstellen einer Verbindung mit einer Anwendung | Microsoft-Dokumentation
description: In diesem Artikel wird erläutert, wie Sie eine Verbindung zwischen einer Anwendung und einer verwalteten Azure SQL-Datenbank-Instanz herstellen.
ms.service: sql-database
author: srdjan-bozovic
manager: craigg
ms.custom: managed instance
ms.topic: article
ms.date: 04/10/2018
ms.author: srbozovi
ms.reviewer: bonova, carlrab
ms.openlocfilehash: 1eecd28d5e7043acae5cfd52edf93e8d301bd31e
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/16/2018
---
# <a name="connect-your-application-to-azure-sql-database-managed-instance"></a>Herstellen einer Verbindung zwischen einer Anwendung und einer verwalteten Azure SQL-Datenbank-Instanz

Heutzutage haben Sie mehrere Möglichkeiten bei der Wahl, wie und wo Sie Anwendungen hosten. 
 
Beispielsweise können Sie eine Anwendung unter Verwendung von Azure App Service oder einiger der integrierten Optionen des virtuellen Netzwerks (VNET) von Azure (z.B. Azure App Service-Umgebung, virtueller Computer, VM-Skalierungsgruppe) in der Cloud hosten. Alternativ können Sie auch den Hybrid Cloud-Ansatz wählen und Ihre Anwendungen lokal hosten. 
 
Unabhängig von Ihrer Wahl können Sie die Anwendungen mit einer verwalteten Instanz (Vorschauversion) verbinden.  

![Hochverfügbarkeit](./media/sql-database-managed-instance/application-deployment-topologies.png)  

## <a name="connect-an-application-inside-the-same-vnet"></a>Herstellen einer Verbindung mit einer Anwendung im selben VNET 

Dies ist das einfachste Szenario. Zwischen virtuellen Computern innerhalb des VNET kann eine direkte Verbindung hergestellt werden, auch wenn sie sich in unterschiedlichen Subnetzen befinden. Zum Herstellen einer Verbindung mit einer Anwendung in einer Azure-Anwendungsumgebung oder einem virtuellen Computer müssen Sie lediglich die Verbindungszeichenfolge entsprechend festlegen.  
 
Falls Sie keine Verbindung herstellen können, sollten Sie überprüfen, ob eine Netzwerksicherheitsgruppe für das Anwendungssubnetz festgelegt ist. In diesem Fall müssen Sie die ausgehende Verbindung am SQL-Port 1433 sowie den Portbereich 11000-12000 für die Umleitung öffnen. 

## <a name="connect-an-application-inside-a-different-vnet"></a>Herstellen einer Verbindung mit einer Anwendung in einem anderen VNET 

Dieses Szenario ist etwas komplexer, da die verwaltete Instanz über eine private IP-Adresse in ihrem eigenen VNET verfügt. Für die Verbindung ist für eine Anwendung der Zugriff auf das VNET erforderlich, in dem die verwaltete Instanz bereitgestellt wird. Daher müssen Sie zunächst eine Verbindung zwischen der Anwendung und dem VNET der verwalteten Instanz herstellen. Die VNETs müssen für dieses Szenario nicht zum selben Abonnement gehören. 
 
Es gibt zwei Optionen zum Herstellen einer Verbindung mit VNETs: 
- [Peering in virtuellen Azure-Netzwerken](../virtual-network/virtual-network-peering-overview.md) 
- VNET-zu-VNET-VPN-Gateway ([Azure-Portal](../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md), [PowerShell](../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md), [Azure CLI](../vpn-gateway/vpn-gateway-howto-vnet-vnet-cli.md)) 
 
Dabei ist die Option des Peerings vorzuziehen, da beim Peering das Microsoft-Backbonenetzwerk verwendet wird, sodass es im Hinblick auf die Konnektivität keinen erkennbaren Unterschied der Latenz zwischen virtuellen Computern im Peering-VNET und im selben VNET gibt. Das VNET-Peering beschränkt sich auf die Netzwerke in derselben Region. Dennoch ist das regionsübergreifende Peering in einigen Regionen als Vorschauversion aktiviert.  
 
> [!IMPORTANT]
> Regionsübergreifend erstellte VNET-Peerings bieten gegebenenfalls nicht die gleiche Verfügbarkeit und Zuverlässigkeit wie Peerings in einer allgemein verfügbaren Version. VNET-Peerings weisen unter Umständen eingeschränkte Funktionen auf und stehen gegebenenfalls nicht in allen Azure-Regionen zur Verfügung. Aktuelle Hinweise zur Verfügbarkeit und zum Status dieses Features finden Sie auf der Seite  [Azure-Updates](https://azure.microsoft.com/updates/?product=virtual-network). 

## <a name="connect-an-on-premises-application"></a>Herstellen einer Verbindung mit einer lokalen Anwendung 

Auf eine verwaltete Instanz kann nur über eine private IP-Adresse zugegriffen werden. Für den Zugriff auf die verwaltete Instanz über die lokale Umgebung müssen Sie eine Standort-zu-Standort-Verbindung zwischen der Anwendung und dem VNET der verwalteten Instanz herstellen. 
 
Es gibt zwei Optionen für die lokale Verbindung mit dem Azure-VNET: 
- Site-to-Site-VPN-Verbindung ([Azure-Portal](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md), [PowerShell](../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md), [Azure CLI](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-cli.md)) 
- [ExpressRoute](../expressroute/expressroute-introduction.md)-Verbindung  
 
Wenn Sie eine lokale Verbindung mit Azure hergestellt haben und keine Verbindung mit der verwalteten Instanz herstellen können, sollten Sie überprüfen, ob für die Firewall eine geöffnete ausgehende Verbindung am SQL-Port 1433 und ein geöffneter Portbereich 11000-12000 für die Umleitung festgelegt sind. 

## <a name="connect-an-azure-app-service-hosted-application"></a>Herstellen einer Verbindung mit einer über Azure App Service gehosteten Anwendung 

Auf eine verwaltete Instanz kann nur über eine private IP-Adresse zugegriffen werden. Für den Zugriff auf die verwaltete Instanz über Azure App Service müssen Sie daher zunächst eine Verbindung zwischen der Anwendung und dem VNET der verwalteten Instanz herstellen. Siehe [Integrieren Ihrer App in ein Azure Virtual Network](../app-service/web-sites-integrate-with-vnet.md).  
 
Informationen zur Problembehandlung finden Sie unter [Problembehandlung bei VNETs und Anwendungen](../app-service/web-sites-integrate-with-vnet.md#troubleshooting). Wenn keine Verbindung hergestellt werden kann, versuchen Sie, [die Netzwerkkonfiguration zu synchronisieren](sql-database-managed-instance-sync-network-configuration.md). 
 
Ein spezieller Fall beim Herstellen einer Verbindung zwischen Azure App Service mit einer verwalteten Instanz besteht, wenn Sie Azure App Service in ein Netzwerk integriert haben, das mittels Peering mit dem VNET der verwalteten Instanz verbunden ist. In diesem Fall muss die folgende Konfiguration eingerichtet werden: 

- Für das VNET der verwalteten Instanz darf KEIN Gateway eingerichtet sein.  
- Für das VNET der verwalteten Instanz muss die Option „Remotegateways verwenden“ aktiviert sein. 
- Für das mittels Peering verbundene VNET muss die Option „Gatewaytransit zulassen“ aktiviert sein. 
 
Dieses Szenario ist in der folgenden Abbildung dargestellt:

![Peering integrierte App](./media/sql-database-managed-instance/integrated-app-peering.png)
 
## <a name="connect-an-application-on-the-developers-box"></a>Herstellen einer Verbindung mit einer Anwendung in der Entwicklerbox 

Auf eine verwaltete Instanz kann nur über eine private IP-Adresse zugegriffen werden. Für den Zugriff auf die verwaltete Instanz über die Entwicklerbox müssen Sie daher zunächst eine Verbindung zwischen der Entwicklerbox und dem VNET der verwalteten Instanz herstellen.  
 
Konfigurieren Sie eine Point-to-Site-Verbindung mit einem VNET unter Verwendung einer nativen Azure-Zertifikatauthentifizierung ([Azure-Portal](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md), [PowerShell](../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md) oder [Azure CLI](../vpn-gateway/vpn-gateway-howto-point-to-site-classic-azure-portal.md)). In diesen Artikeln finden Sie diesbezügliche Informationen.  

## <a name="next-steps"></a>Nächste Schritte

- Informationen zur verwalteten Instanz finden Sie unter [Was ist eine verwaltete Instanz?](sql-database-managed-instance.md)
- Ein Tutorial, in dem die Erstellung einer neuen verwalteten Instanz gezeigt wird, finden Sie unter [Create an Azure SQL Database Managed Instance in the Azure portal](sql-database-managed-instance-create-tutorial-portal.md) (Erstellen einer verwalteten Azure SQL-Datenbank-Instanz im Azure-Portal).
