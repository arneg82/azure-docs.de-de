---
title: Erstellen von Umgebungen mit mehreren virtuellen Computern und PaaS-Ressourcen mit Azure Resource Manager-Vorlagen | Microsoft-Dokumentation
description: Hier erfahren Sie, wie Sie in Azure DevTest Labs auf der Grundlage einer Azure Resource Manager-Vorlage Umgebungen mit mehreren virtuellen Computern und PaaS-Ressourcen erstellen.
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/05/2018
ms.author: spelluru
ms.openlocfilehash: 397ab30b252fbfa121b763b005907764d2b15f20
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/07/2018
---
# <a name="create-multi-vm-environments-and-paas-resources-with-azure-resource-manager-templates"></a>Erstellen von Umgebungen mit mehreren virtuellen Computern und PaaS-Ressourcen mit Azure Resource Manager-Vorlagen

Über das [Azure-Portal](http://go.microsoft.com/fwlink/p/?LinkID=525040) können Sie ganz einfach [einen virtuellen Computer erstellen und einem Lab hinzufügen](https://docs.microsoft.com/azure/devtest-lab/devtest-lab-add-vm). Für einzelne virtuelle Computer funktioniert das sehr gut. Wenn die Umgebung allerdings mehrere virtuelle Computer enthält, muss jeder virtuelle Computer einzeln erstellt werden. Für Szenarien wie etwa eine Web-App mit mehreren Ebenen oder eine SharePoint-Farm ist ein Mechanismus erforderlich, der die gleichzeitige Erstellung mehrerer virtueller Computer ermöglicht. Mit Azure Resource Manager-Vorlagen können Sie jetzt die Infrastruktur und Konfiguration Ihrer Azure-Lösung definieren und wiederholt mehrere virtuelle Computer in einem konsistenten Zustand bereitstellen. Dieses Feature hat folgende Vorteile:

- Azure Resource Manager-Vorlagen werden direkt über Ihr Quellcodeverwaltungs-Repository (GitHub oder Team Services Git) geladen.
- Nach der Konfiguration können Ihre Benutzer eine Umgebung erstellen, indem sie genau wie bei anderen Arten von [VM-Grundlagen](./devtest-lab-comparing-vm-base-image-types.md) eine Azure Resource Manager-Vorlage über das Azure-Portal auswählen.
- Azure PaaS-Ressourcen können in einer Umgebung zusätzlich zu virtuellen IaaS-Computern über eine Azure Resource Manager-Vorlage bereitgestellt werden.
- Die Umgebungskosten können im Lab zusätzlich zu den einzelnen virtuellen Computern nachverfolgt werden, die unter Verwendung anderer Grundlagen erstellt wurden.
- PaaS-Ressourcen werden erstellt und in der Kostenverfolgung aufgeführt. Allerdings gilt das automatische Herunterfahren von VMs nicht für PaaS-Ressourcen.
- Den Benutzern steht für Umgebungen die gleiche VM-Richtliniensteuerung zur Verfügung wie für virtuelle Computer mit einem einzelnen Lab.

Erfahren Sie mehr über die zahlreichen [Vorteile der Verwendung von Resource Manager-Vorlagen](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#the-benefits-of-using-resource-manager) zum Bereitstellen, Aktualisieren oder Löschen aller Ihrer Labressourcen in einem einzigen Vorgang.

> [!NOTE]
> Wenn Sie eine Resource Manager-Vorlage als Grundlage zum Erstellen weiterer Lab-VMs verwenden, sind jedoch einige Unterschiede dahingehend zu berücksichtigen, ob Sie einzelne oder mehrere VMs erstellen. Unter [Verwenden der Azure Resource Manager-Vorlage eines virtuellen Computers](devtest-lab-use-resource-manager-template.md) werden diese Unterschiede detaillierter veranschaulicht.
>
>

## <a name="configure-azure-resource-manager-template-repositories"></a>Konfigurieren von Azure Resource Manager-Vorlagenrepositorys

Bei Infrastruktur als Code und Konfiguration als Code empfiehlt es sich, Umgebungsvorlagen in der Quellcodeverwaltung zu verwalten. Azure DevTest Labs wendet diese Vorgehensweise an und lädt alle Azure Resource Manager-Vorlagen direkt aus Ihrem GitHub- oder VSTS Git-Repository. Folglich können Resource Manager-Vorlagen im gesamten Freigabezyklus (von der Test- bis zur Produktionsumgebung) verwendet werden.

Zum Organisieren von Azure Resource Manager-Vorlagen in einem Repository sind folgende Regeln zu beachten:

- Der Name der Mastervorlagendatei muss `azuredeploy.json` lauten. 

    ![Zentrale Azure Resource Manager-Vorlagendateien](./media/devtest-lab-create-environment-from-arm/master-template.png)

- Wenn Sie in einer Parameterdatei definierte Parameterwerte verwenden möchten, muss der Name der Parameterdatei `azuredeploy.parameters.json` lauten.
- Sie können mithilfe der Parameter `_artifactsLocation` und `_artifactsLocationSasToken` den URI-Wert „parametersLink“ erstellen, der DevTest Labs das automatische Verwalten geschachtelter Vorlagen ermöglicht. Weitere Informationen finden Sie im Blogbeitrag [How Azure DevTest Labs makes nested Resource Manager template deployments easier for testing environments](https://blogs.msdn.microsoft.com/devtestlab/2017/05/23/how-azure-devtest-labs-makes-nested-arm-template-deployments-easier-for-testing-environments/) (Wie Azure DevTest Labs Bereitstellungen geschachtelter Resource Manager-Vorlagen für Testumgebungen vereinfacht).
- Sie können Metadaten definieren, um den Anzeigenamen und die Beschreibung der Vorlage anzugeben. Diese Metadaten müssen sich in einer Datei namens `metadata.json` befinden. Die folgende Metadaten-Beispieldatei veranschaulicht das Angeben von Anzeigename und Beschreibung: 

```json
{
 
"itemDisplayName": "<your template name>",
 
"description": "<description of the template>"
 
}
```

Mit den folgenden Schritten können Sie Ihrem Lab über das Azure-Portal ein Repository hinzufügen: 

1. Melden Sie sich auf dem [Azure-Portal](http://go.microsoft.com/fwlink/p/?LinkID=525040)an.
1. Wählen Sie **Alle Dienste** und dann in der Liste die Option **DevTest Labs**.
1. Wählen Sie in der Liste der Labs das gewünschte Lab aus.   
1. Wählen Sie im Bereich **Übersicht** des Labs die Option **Konfiguration und Richtlinien** aus.

    ![Konfiguration und Richtlinien](./media/devtest-lab-create-environment-from-arm/configuration-and-policies-menu.png)

1. Wählen Sie in der Einstellungsliste **Configuration and Policies** (Konfiguration und Richtlinien) die Option **Repositorys** aus. Auf der Seite **Repositorys** werden die Repositorys aufgelistet, die dem Lab hinzugefügt wurden. Für alle Labs wird automatisch ein Repository namens `Public Repo` generiert und mit dem [DevTest Labs-GitHub-Repository](https://github.com/Azure/azure-devtestlab) verbunden, das mehrere verwendbare VM-Artefakte enthält.

    ![Öffentliches Repository](./media/devtest-lab-create-environment-from-arm/public-repo.png)

1. Wählen Sie **Hinzufügen+** aus, um Ihr Azure Resource Manager-Vorlagenrepository hinzuzufügen.
1. Wenn sich der zweite Bereich des Typs **Repositorys** öffnet, geben Sie die erforderlichen Informationen ein:
    - **Name**: Geben Sie den im Lab verwendeten Repositorynamen ein.
    - **Git-Klon-URL:** Geben Sie die Git-HTTPS-Klon-URL aus GitHub oder Visual Studio Team Services ein.  
    - **Verzweigung**: Geben Sie den Verzweigungsnamen für den Zugriff auf Ihre Azure Resource Manager-Vorlagendefinitionen ein. 
    - **Persönliches Zugriffstoken**: Das persönliche Zugriffstoken ermöglicht den sicheren Zugriff auf Ihr Repository. Wählen Sie  **&lt;Ihr Name> > Mein Profil > Sicherheit > Öffentliches Zugriffstoken** aus, um Ihr Token aus Visual Studio Team Services abzurufen. Um Ihr Token aus GitHub abzurufen, wählen Sie Ihren Avatar und anschließend **Einstellungen > Öffentliches Zugriffstoken** aus. 
    - **Ordnerpfade**: Geben Sie über eines der beiden Eingabefelder den Ordnerpfad ein – entweder zu Ihren Artefaktdefinitionen (erstes Eingabefeld) oder zu Ihren Azure Resource Manager-Vorlagendefinitionen. Der Pfad beginnt mit einem Schrägstrich (/) und wird relativ zu Ihrem Git-Klon URI angegeben.   
    
        ![Öffentliches Repository](./media/devtest-lab-create-environment-from-arm/repo-values.png)


1. Wenn alle erforderlichen Felder ausgefüllt wurden und bei der Überprüfung der Angaben keine Fehler aufgetreten sind, wählen Sie **Speichern** aus.

Im nächsten Abschnitt erfahren Sie, wie Sie Umgebungen auf der Grundlage einer Azure Resource Manager-Vorlage erstellen.

## <a name="create-an-environment-from-a-resource-manager-template-using-the-azure-portal"></a>Erstellen einer Umgebung auf der Grundlage einer Azure Resource Manager-Vorlage im Azure-Portal

Nachdem im Lab ein Azure Resource Manager-Vorlagenrepository konfiguriert wurde, können Ihre Labbenutzer mit folgenden Schritten eine Umgebung über das Azure-Portal erstellen:

1. Melden Sie sich auf dem [Azure-Portal](http://go.microsoft.com/fwlink/p/?LinkID=525040)an.
1. Wählen Sie **Alle Dienste** und dann in der Liste die Option **DevTest Labs**.
1. Wählen Sie in der Liste der Labs das gewünschte Lab aus.   
1. Wählen Sie die Bereich des Labs die Option **Hinzufügen+** aus.
1. Im Bereich **Basis auswählen** werden die Basisimages, die Sie mit Ihren Azure Resource Manager-Vorlagen verwenden können, zuerst aufgeführt. Wählen Sie die gewünschte Azure Resource Manager-Vorlage aus.

    ![Auswählen einer Grundlage](./media/devtest-lab-create-environment-from-arm/choose-a-base.png)
  
1. Geben Sie im Bereich **Hinzufügen** den Wert für **Umgebungsname** ein. Der Umgebungsname ist der Name, der Ihren Benutzern im Lab angezeigt wird. Die restlichen Eingabefelder sind in der Azure Resource Manager-Vorlage definiert. Falls in der Vorlage Standardwerte definiert sind oder die Datei `azuredeploy.parameter.json` vorhanden ist, werden in diesen Eingabefeldern Standardwerte angezeigt. Für Parameter vom Typ *Sichere Zeichenfolge* können Sie die Geheimnisse verwenden, die im [persönlichen geheimen Speicher](https://azure.microsoft.com/updates/azure-devtest-labs-keep-your-secrets-safe-and-easy-to-use-with-the-new-personal-secret-store) des Labs gespeichert sind.

    ![Hinzufügen eines Bereichs](./media/devtest-lab-create-environment-from-arm/add.png)

    > [!NOTE]
    > Einige Parameterwerte werden als leere Werte angezeigt (auch wenn sie angegeben wurden). Wenn Benutzer diese Werte also Parametern in einer Azure Resource Manager-Vorlage zuweisen, zeigt DevTest Labs die Werte nicht an. Stattdessen werden leere Eingabefelder angezeigt, in die Lab-Benutzer beim Erstellen der Umgebung einen Wert eingeben müssen.
    > 
    > - GEN-UNIQUE
    > - GEN-UNIQUE-[N]
    > - GEN-SSH-PUB-KEY
    > - GEN-PASSWORD 
 
1. Wählen Sie **Hinzufügen** aus, um die Umgebung zu erstellen. Die Bereitstellung der Umgebung beginnt umgehend, und der Status wird in der Liste **My virtual machines** (Meine virtuellen Computer) angezeigt. Vom Lab wird automatisch eine neue Ressourcengruppe erstellt, um alle in der Azure Resource Manager-Vorlage definierten Ressourcen bereitzustellen.
1. Wählen Sie die erstellte Umgebung in der Liste **My virtual machines** (Meine virtuellen Computer) aus, um den Ressourcengruppenbereich zu öffnen, und erkunden Sie alle in der Umgebung bereitgestellten Ressourcen.
    
    ![Liste mit Ihren virtuellen Computern](./media/devtest-lab-create-environment-from-arm/all-environment-resources.png)
   
   Sie können die Umgebung auch erweitern, um lediglich die Liste der VMs anzuzeigen, die in der Umgebung bereitgestellt sind.
    
    ![Liste mit Ihren virtuellen Computern](./media/devtest-lab-create-environment-from-arm/my-vm-list.png)

1. Klicken Sie auf eine der Umgebungen, um die verfügbaren Aktionen anzuzeigen. Hierzu zählt beispielsweise das Anwenden von Artefakten, das Anfügen von Datenträgern oder das Anpassen des Zeitpunkts für das automatische Herunterfahren.

    ![Umgebungsaktionen](./media/devtest-lab-create-environment-from-arm/environment-actions.png)

## <a name="deploy-a-resource-manager-template-to-create-a-vm"></a>Bereitstellen einer Resource Manager-Vorlage zum Erstellen einer VM
Nachdem Sie eine Resource Manager-Vorlage gespeichert und an Ihre Anforderungen angepasst haben, können Sie sie zum Automatisieren der VM-Erstellung verwenden. 
- Unter [Bereitstellen von Ressourcen mit Azure Resource Manager-Vorlagen und Azure PowerShell](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy) wird beschrieben, wie Sie Azure PowerShell mit Resource Manager-Vorlagen verwenden, um Ihre Ressourcen in Azure bereitzustellen. 
- Unter [Bereitstellen von Ressourcen mit Azure Resource Manager-Vorlagen und Azure-CLI](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy-cli) wird beschrieben, wie Sie die Azure CLI mit Resource Manager-Vorlagen verwenden, um Ihre Ressourcen in Azure bereitzustellen.

> [!NOTE]
> Nur Benutzer mit einer Berechtigung der Art „Labbesitzer“ können VMs aus einer Resource Manager-Vorlage erstellen, indem sie Azure PowerShell nutzen. Wenn Sie die VM-Erstellung per Resource Manager-Vorlage automatisieren möchten und nur über Benutzerberechtigungen verfügen, können Sie den [Befehl **az lab vm create** in der CLI](https://docs.microsoft.com/cli/azure/lab/vm#az_lab_vm_create) verwenden.

### <a name="general-limitations"></a>Allgemeine Einschränkungen 

Berücksichtigen Sie diese Einschränkungen bei der Verwendung einer Resource Manager-Vorlage in DevTest Labs:

- Jede von Ihnen erstellte Resource Manager-Vorlage kann sich nicht auf vorhandene Ressourcen beziehen, sondern nur auf neue Ressourcen. Wenn Sie eine vorhandene Resource Manager-Vorlage haben, die Sie normalerweise außerhalb von DevTest Labs bereitstellen und Verweise auf vorhandene Ressourcen enthält, kann sie nicht im Lab verwendet werden.

   Die einzige Ausnahme ist, dass Sie auf ein bestehendes virtuelles Netzwerk verweisen **können**. 

- Formeln können nicht mithilfe von Lab-VMs erstellt werden, die anhand einer Resource Manager-Vorlage erstellt wurden. 

- Benutzerdefinierte Images können nicht mithilfe von Lab-VMs erstellt werden, die anhand einer Resource Manager-Vorlage erstellt wurden. 

- Die meisten Richtlinien werden nicht beachtet, wenn Sie Resource Manager-Vorlagen bereitstellen.

   Beispielsweise kann eine Labrichtlinie festgelegt sein, laut der ein Benutzer nur fünf VMs erstellen kann. Wenn der Benutzer jedoch eine Resource Manager-Vorlage bereitstellt, die Dutzende von VMs erstellt, wird dies zugelassen. Es folgen Richtlinien, die nicht beachtet werden:

   - Anzahl der VMs pro Benutzer
   - Anzahl der Premium-VMs pro Lab-Benutzer
   - Anzahl der Premium-Datenträger pro Lab-Benutzer


### <a name="configure-environment-resource-group-access-rights-for-lab-users"></a>Konfigurieren von Zugriffsrechten für Lab-Benutzer in der Umgebungsressourcengruppe

Lab-Benutzer können eine Resource Manager-Vorlage bereitstellen. Doch standardmäßig verfügen sie über die Berechtigung „Leser“, was bedeutet, dass sie die Ressourcen in einer Umgebungsressourcengruppe nicht ändern können. Beispielsweise können sie ihre Ressourcen nicht anhalten oder starten.

Führen Sie diese Schritte aus, um Ihren Lab-Benutzern das Zugriffsrecht „Mitwirkender“ zu erteilen. Wenn sie anschließend eine Resource Manager-Vorlage bereitstellen, können sie die Ressourcen in dieser Umgebung bearbeiten. 


1. Wählen Sie im Bereich **Übersicht** des Labs die Option **Konfiguration und Richtlinien** aus.
1. Wählen Sie **Labeinstellungen** aus.
1. Wählen Sie im Bereich „Labeinstellungen“ die Option **Mitwirkender** aus, um Lab-Benutzern Schreibberechtigungen zu erteilen.

    ![Konfigurieren von Zugriffsrechten für Lab-Benutzer](./media/devtest-lab-create-environment-from-arm/configure-access-rights.png)

1. Wählen Sie **Speichern**aus.

## <a name="next-steps"></a>Nächste Schritte
* Nach der Erstellung des virtuellen Computers können Sie im Verwaltungsbereich des virtuellen Computers die Option **Verbinden** auswählen, um eine Verbindung mit dem virtuellen Computer herzustellen.
* Anzeigen und Verwalten von Ressourcen in einer Umgebung durch Auswählen der Umgebung in der Liste **My virtual machines** (Meine virtuellen Computer) in Ihrem Lab. 
* Sehen Sie sich die [Azure Resource Manager-Vorlagen aus dem Katalog mit Azure-Schnellstartvorlagen](https://github.com/Azure/azure-quickstart-templates) an.
