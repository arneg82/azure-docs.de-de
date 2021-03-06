---
title: Verwaltung der Benutzerbereitstellung für Unternehmens-Apps in Azure Active Directory| Microsoft-Dokumentation
description: Erfahren Sie, wie Sie mithilfe von Azure Active Directory die Benutzerkontobereitstellung für Unternehmens-Apps verwalten.
services: active-directory
documentationcenter: ''
author: asmalser
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: app-mgmt
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/26/2017
ms.author: asmalser
ms.reviewer: asmalser
ms.openlocfilehash: a6f9f35931ff13eb3f0f35748b3a040af37df672
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/20/2018
ms.locfileid: "34337893"
---
# <a name="managing-user-account-provisioning-for-enterprise-apps-in-the-azure-portal"></a>Verwalten der Benutzerkontobereitstellung für Unternehmens-Apps im Azure-Portal
Dieser Artikel beschreibt die Verwendung des [Azure-Portals](https://portal.azure.com) zur Verwaltung der automatischen Bereitstellung und der Aufhebung der Bereitstellung von Benutzerkonten für Anwendungen, die dies unterstützen – insbesondere für Anwendungen, die aus der Kategorie „Vorgestellt“ des [Azure Active Directory-Anwendungskatalogs](manage-apps/what-is-single-sign-on.md#get-started-with-the-azure-ad-application-gallery) hinzugefügt wurden. Weitere Informationen zur automatisierten Benutzerbereitstellung sowie zu deren Funktionsweise finden Sie unter [Automatisieren der Bereitstellung und Bereitstellungsaufhebung von Benutzern für SaaS-Anwendungen mit Azure Active Directory](active-directory-saas-app-provisioning.md).

## <a name="finding-your-apps-in-the-portal"></a>Suchen Ihrer Apps im Portal
Alle Anwendungen, die von einem Verzeichnisadministrator mithilfe des [Azure Active Directory-Anwendungskatalogs](manage-apps/what-is-single-sign-on.md#get-started-with-the-azure-ad-application-gallery) konfiguriert werden, können im [Azure-Portal](https://portal.azure.com) angezeigt und verwaltet werden. Sie finden die Anwendungen im Portal im Abschnitt **Alle Dienste** &gt; **Unternehmensanwendungen**. Unternehmens-Apps sind Apps, die innerhalb Ihrer Organisation bereitgestellt und verwendet werden.

![Bereich für Unternehmensanwendungen](./media/active-directory-enterprise-apps-manage-provisioning/enterprise-apps-pane.png)

Wählen Sie auf der linken Seite den Link **All applications** (Alle Anwendungen), um eine Liste aller konfigurierten Apps anzuzeigen, einschließlich Apps, die aus dem Katalog hinzugefügt wurden. Beim Auswählen einer App wird der Ressourcenbereich für die App geladen, in dem Berichte für die App angezeigt und eine Reihe von Einstellungen verwaltet werden können.

Einstellungen für die Bereitstellung von Benutzerkonten können durch Auswählen der Option **Bereitstellung** auf der linken Seite verwaltet werden.

![Bereich für Anwendungsressourcen](./media/active-directory-enterprise-apps-manage-provisioning/enterprise-apps-provisioning.png)

## <a name="provisioning-modes"></a>Bereitstellungsmodi
Der Bereich **Bereitstellung** beginnt mit dem Menü **Modus**, das anzeigt, welche Bereitstellungsmodi für eine Unternehmensanwendung unterstützt werden, und in dem Sie die Modi konfigurieren können. Die verfügbaren Optionen umfassen:

* **Automatisch**: Diese Option wird angezeigt, wenn Azure AD die automatische API-basierte Bereitstellung bzw. das Aufheben der Bereitstellung von Benutzerkonten für diese Anwendung unterstützt. Bei Auswahl dieses Modus wird eine Schnittstelle angezeigt, die Administratoren durch folgende Aufgaben führt: Konfiguration von Azure AD zum Herstellen einer Verbindung mit der Benutzerverwaltungs-API der Anwendung, Erstellen von Kontozuordnungen und -workflows, die den Fluss von Benutzerkontodaten zwischen Azure AD und der App definieren und Verwaltung des Azure AD-Bereitstellungsdiensts.
* **Manuell** : Diese Option wird angezeigt, wenn Azure AD die automatische Bereitstellung von Benutzerkonten für diese Anwendung nicht unterstützt. Diese Option bedeutet, dass in der Anwendung gespeicherte Benutzerkontodatensätze basierend auf den Benutzerverwaltungs- und Bereitstellungsfunktionen dieser Anwendung (einschließlich SAML Just-in-Time-Bereitstellung) mit einem externen Prozess verwaltet werden müssen.

## <a name="configuring-automatic-user-account-provisioning"></a>Konfigurieren der automatischen Bereitstellung von Benutzerkonten
Bei Auswahl der Option **Automatisch** wird ein Bildschirm angezeigt, der in vier Abschnitte unterteilt ist:

### <a name="admin-credentials"></a>Administratoranmeldeinformationen
In diesem Abschnitt müssen die Anmeldeinformationen für Azure AD eingegeben werden, um eine Verbindung mit der Benutzerverwaltungs-API der Anwendung herzustellen. Die erforderliche Eingabe variiert je nach Anwendung. Informationen zu den Anmeldeinformationstypen und den Anforderungen für bestimmte Anwendungen finden Sie im [Konfigurationstutorial der jeweiligen Anwendung](active-directory-saas-app-provisioning.md).

Durch Auswählen der Schaltfläche **Verbindung testen** können Sie die Anmeldeinformationen testen, indem Azure AD versucht, mit den angegebenen Anmeldeinformationen eine Verbindung mit der Bereitstellungs-App der App herzustellen.

### <a name="mappings"></a>Zuordnungen
In diesem Abschnitt können Administratoren anzeigen und bearbeiten, welche Benutzerattribute zwischen Azure AD und der Zielanwendung übertragen werden, wenn Benutzerkonten bereitgestellt oder aktualisiert werden.

Zwischen Azure AD-Benutzerobjekten und den Benutzerobjekten der einzelnen SaaS-Apps ist eine vorkonfigurierte Sammlung von Zuordnungen vorhanden. Einige Apps verwalten andere Objekttypen, z. B. Gruppen oder Kontakte. Bei Auswahl einer dieser Zuordnungen in die Tabelle wird rechts der Mapping-Editor geöffnet, in dem Zuordnungen angezeigt und angepasst werden können.

![Bereich für Anwendungsressourcen](./media/active-directory-enterprise-apps-manage-provisioning/enterprise-apps-provisioning-mapping.png)

Unterstützte Anpassungen umfassen:

* Aktivieren und Deaktivieren von Zuordnungen für bestimmte Objekte, z. B. das Azure AD-Benutzerobjekt an das Benutzerobjekt der SaaS-App.
* Bearbeiten der Attribute, die vom Azure AD-Benutzerobjekt an das Benutzerobjekt der App übermittelt werden Weitere Informationen zur Attributzuordnung finden Sie unter [Grundlegendes zu Attributzuordnungstypen](active-directory-saas-customizing-attribute-mappings.md#understanding-attribute-mapping-types).
* Filtern Sie die Bereitstellungsaktionen, die Azure AD für die Zielanwendung ausführt. Anstatt eine vollständige Synchronisierung von Objekten durch Azure AD zuzulassen, können Sie die ausgeführten Aktionen beschränken. Beispiel: Wenn Sie nur **Aktualisieren**auswählen, aktualisiert Azure AD nur vorhandene Benutzerkonten in einer Anwendung und erstellt keine neuen. Wird nur **Erstellen**ausgewählt, erstellt Azure nur neue Benutzerkonten, aktualisiert jedoch keine vorhandenen Konten. Dieses Feature ermöglicht Administratoren das Erstellen verschiedener Zuordnungen für die Kontoerstellung und das Aktualisieren von Workflows.

### <a name="settings"></a>Einstellungen
In diesem Abschnitt können Administratoren den Azure AD-Bereitstellungsdienst für die ausgewählte Anwendung starten und beenden. Außerdem haben sie die Möglichkeit, den Bereitstellungscache zu leeren und den Dienst neu zu starten.

Wenn die Bereitstellung für eine Anwendung zum ersten Mal aktiviert wird, legen Sie den **Bereitstellungsstatus** auf **Ein** fest, um den Dienst zu aktivieren. Aufgrund dieser Änderung führt der Azure AD-Bereitstellungsdienst eine anfängliche Synchronisierung durch. Hierzu liest er die im Abschnitt **Benutzer und Gruppen** zugewiesenen Benutzer, fragt die Zielanwendung für diese Benutzer ab und führt dann die im Azure AD-Abschnitt **Zuordnungen** definierten Bereitstellungsaktionen durch. Während dieses Vorgangs speichert der Bereitstellungsdienst Daten im Cache dazu, welche Benutzerkonten er verwaltet, damit die Bereitstellung nicht verwalteter Konten innerhalb der Zielanwendungen, die nicht im Zuweisungsumfang enthalten waren, nicht aufgehoben wird. Nach der anfänglichen Synchronisierung synchronisiert der Bereitstellungsdienst automatisch Benutzer- und Gruppenobjekte in einem Intervall von 10 Minuten.

Durch Ändern des **Bereitstellungsstatus** in **Aus** wird der Bereitstellungsdienst lediglich angehalten. In diesem Status werden von Azure keine Benutzer- oder Gruppenobjekte in der App erstellt, aktualisiert oder entfernt. Wird der Status wieder zu „Ein“ geändert, fährt der Dienst fort, wo er unterbrochen wurde.

Wenn das Kontrollkästchen zum **Deaktivieren des aktuellen Status und Starten der Synchronisierung** ausgewählt und dann gespeichert wird, wird der Bereitstellungsdienst beendet, verwirft die zwischengespeicherten Daten zu den von Azure AD verwalteten Konten, startet die Dienste neu und führt die anfängliche Synchronisierung erneut aus. Diese Option ermöglicht Administratoren das erneute Starten des Bereitstellungsprozesses.

### <a name="synchronization-details"></a>Synchronisierungsdetails
Dieser Abschnitt enthält weitere Details zum Vorgang des Bereitstellungsdiensts, einschließlich der ersten und letzten Ausführung des Bereitstellungsdiensts für die Anwendung, sowie die Anzahl der verwalteten Benutzer- und Gruppenkonten.

Es wird ein Link zum **Bericht über die Bereitstellungsaktivität** bereitgestellt. Dieser Bericht enthält ein Protokoll zu allen Benutzern und Gruppen, die zwischen Azure AD und der Zielanwendung erstellt, aktualisiert und entfernt wurden. Darüber hinaus wird ein Link zum **Fehlerbericht der Bereitstellung** bereitgestellt. Dieser enthält detaillierte Fehlermeldungen für Benutzer- und Gruppenobjekte, die nicht gelesen, erstellt, aktualisiert oder entfernt werden konnten. 

## <a name="feedback"></a>Feedback

Es wäre schön, wenn Sie uns weiter Feedback senden würden! Feedback und Verbesserungsvorschläge können Sie uns im [Feedbackforum](https://feedback.azure.com/forums/169401-azure-active-directory/category/162510-admin-portal) über den Abschnitt **Verwaltungsportal** zukommen lassen.  Das Technikerteam hat großen Spaß daran, jeden Tag neue Dinge zu entwickeln, und Ihr Feedback ist dabei eine große Hilfe für das Team, die nächsten Ziele anzugehen und zu definieren.

