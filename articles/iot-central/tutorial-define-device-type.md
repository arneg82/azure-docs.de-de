---
title: Definieren eines neuen Gerätetyps in Azure IoT Central | Microsoft-Dokumentation
description: In diesem Tutorial für Ersteller erfahren Sie, wie Sie in Ihrer Azure IoT Central-Anwendung einen neuen Gerätetyp definieren. Sie definieren die Telemetriedaten, den Zustand, die Eigenschaften und die Einstellungen für den Typ.
services: iot-central
author: tanmaybhagwat
ms.author: tanmayb
ms.date: 04/16/2018
ms.topic: tutorial
ms.prod: microsoft-iot-central
manager: timlt
ms.openlocfilehash: e1488b708bbbee67362d834a9a703520d37bef37
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/16/2018
ms.locfileid: "34201671"
---
# <a name="1---define-a-new-device-type-in-your-azure-iot-central-application"></a>1: Definieren eines neuen Gerätetyps in Ihrer Azure IoT Central-Anwendung

In diesem Tutorial für Ersteller erfahren Sie, wie Sie in Ihrer Microsoft Azure IoT Central-Anwendung mithilfe einer Gerätevorlage eine neue Art von Gerät definieren. Eine Gerätevorlage definiert die Telemetriedaten, den Zustand, die Eigenschaften und die Einstellungen für Ihren Gerätetyp.

Damit Sie Ihre Anwendung testen können, bevor Sie eine Verbindung mit einem echten Gerät herstellen, generiert Azure IoT Central auf der Grundlage der erstellten Vorlage ein simuliertes Gerät.

In diesem Tutorial erstellen Sie die Gerätevorlage **Connected Air Conditioner** (Verbundene Klimaanlage). Für eine verbundene Klimaanlage gilt Folgendes:

* Sie sendet Telemetriedaten wie Temperatur und Luftfeuchtigkeit.
* Sie meldet den Zustand (etwa, ob sie eingeschaltet ist).
* Sie verfügt über Eigenschaften wie Firmwareversion und Seriennummer.
* Sie verfügt über Einstellungen wie Zieltemperatur und Lüftergeschwindigkeit.

In diesem Tutorial lernen Sie Folgendes:

> [!div class="checklist"]
> * Erstellen einer neuen Gerätevorlage
> * Hinzufügen von Telemetriedaten zu Ihrem Gerät
> * Anzeigen simulierter Telemetriedaten
> * Definieren der Ereignismessung
> * Anzeigen simulierter Ereignisse
> * Definieren der Zustandsmessung
> * Anzeigen des simulierten Zustands
> * Verwenden von Geräteeigenschaften
> * Verwenden von Geräteeinstellungen

## <a name="prerequisites"></a>Voraussetzungen

Für diese Schnellstartanleitung benötigen Sie eine Azure IoT Central-Anwendung. Wenn Sie die Schritte der Schnellstartanleitung [Create an Azure IoT Central application](quick-deploy-iot-central.md) (Erstellen einer Azure IoT Central-Anwendung) ausgeführt haben, können Sie die so erstellte Anwendung verwenden. Führen Sie andernfalls die folgenden Schritte aus, um eine leere Azure IoT Central-Anwendung zu erstellen:

1. Navigieren Sie zur Azure IoT Central-Seite [Application Manager](https://aka.ms/iotcentral) (Anwendungs-Manager).

1. Geben Sie die E-Mail-Adresse und das Kennwort für den Zugriff auf Ihr Azure-Abonnement ein:

   ![Eingeben Ihres Organisationskontos](media/tutorial-define-device-type/sign-in.png)

1. Klicken Sie auf **Neue Anwendung**, um mit der Erstellung einer neuen Azure IoT Central-Anwendung zu beginnen:

    ![Azure IoT Central-Seite „Application Manager“ (Anwendungs-Manager)](media/tutorial-define-device-type/iotcentralhome.png)

1. So erstellen Sie eine neue Azure IoT Central-Anwendung:

    1. Wählen Sie einen Anzeigenamen für die Anwendung (beispielsweise **Contoso Air Conditioners**). Azure IoT Central generiert automatisch ein eindeutiges URL-Präfix. Dieses URL-Präfix kann in einen einprägsameren Wert geändert werden.
    1. Wählen Sie eine Azure Active Directory-Instanz und ein Azure-Abonnement aus. Weitere Informationen zu Verzeichnissen und Abonnements finden Sie unter [Create your Azure IoT Central Application](howto-create-application.md) (Erstellen Ihrer Azure IoT Central-Anwendung).
    1. Verwenden Sie entweder eine bereits vorhandene Ressourcengruppe, oder erstellen Sie eine Ressourcengruppe mit einem beliebigen Namen. Beispiel: **contoso-rg**.
    1. Wählen Sie die Region aus, die Ihnen geografisch am nächsten liegt.
    1. Wählen Sie die Anwendungsvorlage **Benutzerdefinierte Anwendung** aus.
    1. Wählen Sie den Zahlungsplan **Free 30 Day Trial Application** (Kostenlose 30-tägige Testanwendung) aus.
    1. Wählen Sie dann **Erstellen** aus.

    ![Azure IoT Central-Seite „Anwendung erstellen“](media/tutorial-define-device-type/iotcentralcreate.png)

Weitere Informationen finden Sie unter [Create your Azure IoT Central Application](howto-create-application.md) (Erstellen Ihrer Azure IoT Central-Anwendung).

## <a name="create-a-new-custom-device-template"></a>Erstellen einer neuen benutzerdefinierten Gerätevorlage

Als Ersteller können Sie die Gerätevorlagen in Ihrer Anwendung erstellen und bearbeiten. Wenn Sie eine Gerätevorlage erstellen, generiert Azure IoT Central auf der Grundlage der Vorlage ein simuliertes Gerät. Das simulierte Gerät generiert Telemetriedaten, mit denen Sie das Verhalten Ihrer Anwendung testen können, bevor Sie eine Verbindung mit einem physischen Gerät herstellen.

Wenn Sie Ihrer Anwendung eine neue Gerätevorlage hinzufügen möchten, navigieren Sie zur Seite **Application Builder** (Anwendungs-Generator). Klicken Sie hierzu im linken Navigationsmenü auf **Application Builder** (Anwendungs-Generator):

    ![Application Builder page](media/tutorial-define-device-type/builderhome.png)

## <a name="add-a-device-and-define-telemetry"></a>Hinzufügen eines Geräts und Definieren von Telemetriedaten

In diesem Abschnitt erfahren Sie Schritt für Schritt, wie Sie für Geräte, die temperaturbezogene Telemetriedaten an Ihre Anwendung senden, eine neue Gerätevorlage namens **Connected Air Conditioner** erstellen:

1. Klicken Sie auf der Seite **Application Builder** (Anwendungs-Generator) auf **Create Device Template** (Gerätevorlage erstellen):

    ![Seite „Application Builder“ (Anwendungs-Generator), Option „Create Device Template“ (Gerätevorlage erstellen)](media/tutorial-define-device-type/builderhomedevices.png)

1. Klicken Sie auf der Seite **Device Templates** (Gerätevorlagen) auf **Benutzerdefiniert**. Mit einer Gerätevorlage vom Typ **Benutzerdefiniert** können Sie sämtliche Eigenschaften und Verhaltensweisen Ihrer verbundenen Klimaanlage definieren:

    ![Geräte](media/tutorial-define-device-type/builderhomedevicescustom.png)

1. Geben Sie auf der Seite **New Device Template** (Neue Gerätevorlage) den Gerätenamen **Connected Air Conditioner** ein, und klicken Sie anschließend auf **Erstellen**. Sie können auch ein Bild Ihres Geräts hochladen, das Bedienern im Device Explorer angezeigt wird:

    ![Benutzerdefiniertes Gerät](media/tutorial-define-device-type/createcustomdevice.png)

1. Vergewissern Sie sich in der Gerätevorlage **Connected Air Conditioner**, dass Sie sich auf der Seite **Measurements** (Messungen) befinden, um die Telemetriedaten zu definieren. Jede Gerätevorlage, die Sie definieren, bietet individuelle Seiten für Folgendes:

    * Angeben der Messungen, die vom Gerät gesendet werden (etwa Telemetriedaten, Ereignisse und Zustände)
    * Definieren der Einstellungen zum Steuern des Geräts
    * Definieren der Eigenschaften zum Erfassen von Informationen zum Gerät
    * Definieren der Regeln für das Gerät
    * Anpassen des Gerätedashboards für die Bediener

    ![Messungen für die Klimaanlage](media/tutorial-define-device-type/airconmeasurements.png)

    > [!NOTE]
    > Wenn Sie den Namen des Geräts oder der Gerätevorlage ändern möchten, klicken Sie auf den Text im oberen Seitenbereich.

1. Klicken Sie auf **New Measurement** (Neue Messung), um die Messung für die Temperaturtelemetrie hinzuzufügen. Wählen Sie **Telemetrie** als Messungstyp aus:

    ![Messungen für die verbundene Klimaanlage](media/tutorial-define-device-type/airconmeasurementsnew.png)

1. Jede Art von Telemetrie, die Sie für eine Gerätevorlage definieren, beinhaltet [Konfigurationsoptionen](howto-set-up-template.md) wie etwa:

    * Anzeigeoptionen
    * Telemetriedetails
    * Simulationsparameter

    Verwenden Sie zum Konfigurieren der Telemetrie **Temperature** (Temperatur) die Informationen aus der folgenden Tabelle:

    | Einstellung              | Wert         |
    | -------------------- | -----------   |
    | Anzeigename         | Temperature   |
    | Feldname           | temperature   |
    | Units                | F             |
    | Min                  | 60            |
    | max                  | 110           |
    | Dezimalstellen       | 0             |

    Sie können auch eine Farbe für die Telemetrieanzeige auswählen. Klicken Sie auf **Speichern**, um die Telemetriedefinition zu speichern:

    ![Konfigurieren der Temperatursimulation](media/tutorial-define-device-type/temperaturesimulation.png)

1. Nach kurzer Zeit wird auf der Seite **Measurements** (Messungen) ein Diagramm der Temperaturtelemetriedaten Ihrer simulierten verbundenen Klimaanlage angezeigt. Mithilfe der Steuerelemente können Sie die Sichtbarkeit sowie die Aggregation verwalten und die Telemetriedefinition bearbeiten:

    ![Anzeigen der Temperatursimulation](media/tutorial-define-device-type/viewsimulation.png)

1. Darüber hinaus können Sie das Diagramm mithilfe der Steuerelemente **Linie**, **Gestapelt** und **Zeitbereich bearbeiten** anpassen:

    ![Anpassen des Diagramms](media/tutorial-define-device-type/customizechart.png)

## <a name="define-event-measurement"></a>Definieren der Ereignismessung
Mithilfe der Ereignisoption können Sie Zeitpunktdaten definieren, die vom Gerät gesendet werden, um ein bedeutsames Ereignis wie etwa einen Fehler oder den Ausfall einer Komponente anzugeben. Geräteereignisse können ähnlich wie Telemetriemessungen von Azure IoT Central simuliert werden, sodass Sie das Verhalten Ihrer Anwendung testen können, bevor Sie eine Verbindung mit einem physischen Gerät herstellen. Ereignismessungen für den Gerätetyp werden in der Ansicht **Measurements** (Messungen) definiert.

1. Klicken Sie auf **New Measurement** (Neue Messung), um die Ereignismessung **Fan Motor Error** (Lüftermotorfehler) hinzuzufügen. Wählen Sie dann **Ereignis** als Messungstyp aus:

    ![Messungen für die verbundene Klimaanlage](media/tutorial-define-device-type/eventnew.png)

1. Jede Art von Ereignis, das Sie für eine Gerätevorlage definieren, beinhaltet [Konfigurationsoptionen](howto-set-up-template.md) wie etwa:

    * Anzeigename
    * Feldname
    * Schweregrad

    Verwenden Sie zum Konfigurieren des Ereignisses **Fan Motor Error** die Informationen aus der folgenden Tabelle:

    | Einstellung              | Wert             |
    | -------------------- | -----------       |
    | Anzeigename         | Fan Motor Error   |
    | Feldname           | fanmotorerr       |
    | Schweregrad             | Fehler             |

    Klicken Sie auf **Speichern**, um die Ereignisdefinition zu speichern:

    ![Konfigurieren der Ereignismessung](media/tutorial-define-device-type/eventconfiguration.png)

1. Nach kurzer Zeit wird auf der Seite **Measurements** (Messungen) ein Diagramm der Ereignisse angezeigt, die nach dem Zufallsprinzip auf der Grundlage Ihrer simulierten verbundenen Klimaanlage generiert wurden. Mithilfe der Steuerelemente können Sie die Sichtbarkeit verwalten und die Ereignisdefinition bearbeiten:

    ![Anzeigen der Ereignissimulation](media/tutorial-define-device-type/eventview.png)

1. Klicken Sie im Diagramm auf das Ereignis, um zusätzliche Ereignisdetails anzuzeigen:

    ![Anzeigen von Ereignisdetails](media/tutorial-define-device-type/eventviewdetail.png)


## <a name="define-state-measurement"></a>Definieren der Zustandsmessung
Mithilfe der Zustandsoption können Sie den Zustand des Geräts oder der dazugehörigen Komponenten über einen Zeitraum definieren und visualisieren. Der Gerätezustand kann ähnlich wie Telemetriemessungen von Azure IoT Central simuliert werden, sodass Sie das Verhalten Ihrer Anwendung testen können, bevor Sie eine Verbindung mit einem physischen Gerät herstellen. Zustandsmessungen für den Gerätetyp werden in der Ansicht **Measurements** (Messungen) definiert.

1. Klicken Sie auf **New Measurement** (Neue Messung), um die Messung **Fan Mode** (Lüftermodus) hinzuzufügen. Wählen Sie dann **Zustand** als Messungstyp aus:

    ![Zustandsmessungen für die verbundene Klimaanlage](media/tutorial-define-device-type/statenew.png)

1. Jede Art von Zustand, den Sie für eine Gerätevorlage definieren, beinhaltet [Konfigurationsoptionen](howto-set-up-template.md) wie etwa:

    * Anzeigename
    * Feldname
    * Werte mit optionalen Anzeigebeschriftungen
    * Wertspezifische Farben

    Verwenden Sie zum Konfigurieren des Zustands **Fan Mode** (Lüftermodus) die Informationen aus der folgenden Tabelle:

    | Einstellung              | Wert             |
    | -------------------- | -----------       |
    | Anzeigename         | Fan Mode          |
    | Feldname           | fanmode           |
    | Wert                | 1                 |
    | Anzeigebeschriftung        | Operating         |
    | Wert                | 0                 |
    | Anzeigebeschriftung        | Stopped           |

    Klicken Sie auf **Speichern**, um die Definition der Zustandsmessung zu speichern:

    ![Konfigurieren der Zustandsmessung](media/tutorial-define-device-type/stateconfiguration.png)

1. Nach kurzer Zeit wird auf der Seite **Measurements** (Messungen) ein Diagramm der Zustände angezeigt, die nach dem Zufallsprinzip auf der Grundlage Ihrer simulierten verbundenen Klimaanlage generiert wurden. Mithilfe der Steuerelemente können Sie die Sichtbarkeit verwalten und die Zustandsdefinition bearbeiten:

    ![Anzeigen der Zustandssimulation](media/tutorial-define-device-type/stateview.png)

1. Sollten vom Gerät innerhalb kurzer Zeit zu viele Datenpunkte gesendet werden, wird die Zustandsmessung anders dargestellt, wie in der folgenden Abbildung zu sehen. Wenn Sie auf das Diagramm klicken, werden alle Datenpunkte innerhalb dieses Zeitraums in chronologischer Reihenfolge angezeigt. Sie können den Zeitbereich auch einschränken, um die Messung im Diagramm anzuzeigen.

    ![Anzeigen von Zustandsdetails](media/tutorial-define-device-type/stateviewdetail.png)

## <a name="properties-device-properties-and-settings"></a>Eigenschaften, Geräteeigenschaften und Einstellungen

Eigenschaften, Geräteeigenschaften und Einstellungen sind unterschiedliche Werte, die in einer Gerätevorlage definiert und jedem einzelnen Gerät zugeordnet werden:

* _Einstellungen_ dienen dazu, Konfigurationsdaten aus Ihrer Anwendung an ein Gerät zu senden. Mithilfe einer Einstellung kann ein Bediener beispielsweise das Telemetrieintervall des Geräts von zwei Sekunden in fünf Sekunden ändern. Wenn ein Bediener eine Einstellung ändert, wird diese auf der Benutzeroberfläche als ausstehend markiert, bis das Gerät bestätigt, dass die Einstellungsänderung durchgeführt wurde.
* _Eigenschaften_ dienen zur Erfassung von Geräteinformationen in Ihrer Anwendung. Mithilfe von Eigenschaften können Sie beispielsweise die Seriennummer eines Geräts oder die Telefonnummer des Geräteherstellers erfassen. Eigenschaften werden in der Anwendung gespeichert und nicht mit dem Gerät synchronisiert. Ein Bediener kann Eigenschaften Werte zuweisen.
* _Geräteeigenschaften_ dienen dazu, einem Gerät das Senden von Eigenschaftswerten an Ihre Anwendung zu ermöglichen. Diese Eigenschaften können nur durch das Gerät geändert werden. Für Bediener sind Geräteeigenschaften schreibgeschützt.

## <a name="use-settings"></a>Verwenden von Einstellungen

_Einstellungen_ ermöglichen es einem Bediener, Konfigurationsdaten an ein Gerät zu senden. In diesem Abschnitt fügen Sie der Gerätevorlage **Connected Air Conditioner** eine Einstellung hinzu, die es Bedienern ermöglicht, die Zieltemperatur der verbundenen Klimaanlage festzulegen.

1. Navigieren Sie zur Seite **Einstellungen** für die Gerätevorlage **Connected Air Conditioner**:

    ![Vorbereiten des Hinzufügens einer Einstellung](media/tutorial-define-device-type/deviceaddsetting.png)

    Sie können Einstellungen unterschiedlicher Art erstellen – beispielsweise Zahlen oder Text.

1. Klicken Sie auf **Zahl**, um Ihrem Gerät eine Zahleneinstellung hinzuzufügen.

1. Verwenden Sie zum Konfigurieren der Einstellung **Set Temperature** (Sollwerttemperatur) die Informationen aus der folgenden Tabelle:

    | Feld                | Wert           |
    | -------------------- | -----------     |
    | Anzeigename         | Set Temperature |
    | Feldname           | setTemperature  |
    | Unit of measure (Maßeinheit)  | F               |
    | Dezimalstellen       | 1               |
    | Mindestwert        | 20              |
    | Maximalwert        | 200             |
    | Anfangswert        | 80              |
    | Beschreibung          | Set the target temperature for the air conditioner |

    Klicken Sie anschließend auf **Speichern**:

    ![Konfigurieren der Einstellung für die Sollwerttemperatur](media/tutorial-define-device-type/configuresetting.png)

    > [!NOTE]
    > Wenn das Gerät eine Einstellungsänderung bestätigt, ändert sich der Status der Einstellungsänderung in **Synchronisiert**.

1. Sie können die Einstellungskacheln verschieben und ihre Größe ändern, um das Layout der Seite **Einstellungen** anzupassen:

    ![Anpassen des Layouts der Einstellungen](media/tutorial-define-device-type/settingslayout.png)

## <a name="use-properties"></a>Verwenden von Eigenschaften

_Eigenschaften_ dienen dazu, Geräteinformationen in der Anwendung zu speichern. In diesem Abschnitt fügen Sie der Gerätevorlage **Connected Air Conditioner** Eigenschaften zum Speichern der Seriennummer und der Firmwareversion des jeweiligen Geräts hinzu.

1. Navigieren Sie zur Seite **Eigenschaften** für die Gerätevorlage **Connected Air Conditioner**:

    ![Vorbereiten des Hinzufügens einer Eigenschaft](media/tutorial-define-device-type/deviceaddproperty.png)

    Sie können Eigenschaften unterschiedlicher Art erstellen – beispielsweise Zahlen oder Text. Klicken Sie auf **Text**, um Ihrer Gerätevorlage eine Seriennummerneigenschaft hinzuzufügen.

1. Verwenden Sie zum Konfigurieren der Seriennummerneigenschaft die Informationen aus der folgenden Tabelle:

    | Feld                | Wert                |
    | -------------------- | -------------------- |
    | Anzeigename         | Serial number        |
    | Feldname           | serialNumber         |
    | Anfangswert        | cac00001             |
    | Beschreibung          | Device serial number |

    Behalten Sie für die anderen Felder die Standardwerte bei.

    ![Konfigurieren der Geräteeigenschaften](media/tutorial-define-device-type/configureproperties.png)

    Klicken Sie auf **Speichern**.

1. Klicken Sie auf **Text**, um Ihrer Gerätevorlage eine Firmwareversionseigenschaft hinzuzufügen.

1. Verwenden Sie zum Konfigurieren der Firmwareversionseigenschaft die Informationen aus der folgenden Tabelle:

    | Feld                | Wert                   |
    | -------------------- | ----------------------- |
    | Anzeigename         | Firmware version        |
    | Feldname           | firmwareVersion         |
    | Anfangswert        | 0.1                     |
    | Beschreibung          | Device firmware version |

    ![Konfigurieren der Geräteeigenschaften](media/tutorial-define-device-type/configureproperties2.png)

    Klicken Sie auf **Speichern**.

1. Sie können die Eigenschaftenkacheln verschieben und ihre Größe ändern, um das Layout der Seite **Eigenschaften** anzupassen:

    ![Anpassen des Layouts der Eigenschaften](media/tutorial-define-device-type/propertieslayout.png)

## <a name="view-your-simulated-device"></a>Anzeigen Ihres simulierten Geräts

Nachdem Sie Ihre Gerätevorlage **Connected Air Conditioner** definiert haben, können Sie das dazugehörige **Dashboard** anpassen, um ihm die definierten Messungen, Einstellungen und Eigenschaften hinzuzufügen. Anschließend können Sie eine Dashboardvorschau als Bediener anzeigen:

1. Navigieren Sie zur Seite **Dashboard** für die Gerätevorlage **Connected Air Conditioner**:

    ![Dashboards für die verbundene Klimaanlage](media/tutorial-define-device-type/aircondashboards.png)

1. Klicken Sie auf **Liniendiagramm**, um die Komponente dem **Dashboard** hinzuzufügen:

    ![Dashboardkomponenten](media/tutorial-define-device-type/dashboardcomponents1.png)

1. Konfigurieren Sie die Komponente **Liniendiagramm** mit den Informationen aus der folgenden Tabelle:

    | Einstellung      | Wert       |
    | ------------ | ----------- |
    | Titel        | Temperature |
    | Zeitbereich   | Letzte 30 Minuten |
    | Measurements (Messungen) | temperature (Klicken Sie neben **temperature** auf **Sichtbarkeit**.) |

    ![Einstellungen für Liniendiagramm](media/tutorial-define-device-type/linechartsettings.png)

    Klicken Sie auf **Speichern**.

1. Konfigurieren Sie die Komponente **Event Chart** (Ereignisdiagramm) mit den Informationen aus der folgenden Tabelle:

    | Einstellung      | Wert       |
    | ------------ | ----------- |
    | Titel        | Events |
    | Zeitbereich   | Letzte 30 Minuten |
    | Measurements (Messungen) | Fan Motor Error (Klicken Sie neben **Fan Motor Error** auf **Sichtbarkeit**.) |

    ![Einstellungen für Liniendiagramm](media/tutorial-define-device-type/dashboardeventchartsetting.png)

    Klicken Sie auf **Speichern**.

1. Konfigurieren Sie die Komponente **State Chart** (Zustandsdiagramm) mit den Informationen aus der folgenden Tabelle:

    | Einstellung      | Wert       |
    | ------------ | ----------- |
    | Titel        | Fan Mode |
    | Zeitbereich   | Letzte 30 Minuten |
    | Measurements (Messungen) | Fan Mode (Klicken Sie neben **Fan Mode** auf **Sichtbarkeit**.) |

    ![Einstellungen für Liniendiagramm](media/tutorial-define-device-type/dashboardstatechartsetting.png)

    Klicken Sie auf **Speichern**.

1. Klicken Sie auf **Settings and Properties** (Einstellungen und Eigenschaften), um dem Dashboard die Einstellung für die Sollwerttemperatur hinzuzufügen:

    ![Dashboardkomponenten](media/tutorial-define-device-type/dashboardcomponents4.png)

1. Konfigurieren Sie die Komponente **Settings and Properties** (Einstellungen und Eigenschaften) mit den Informationen aus der folgenden Tabelle:

    | Einstellung                 | Wert         |
    | ----------------------- | ------------- |
    | Titel                   | Set target temperature |
    | Settings and Properties (Einstellungen und Eigenschaften) | Set Temperature |

    ![Einstellungen für die Seriennummerneigenschaft](media/tutorial-define-device-type/propertysettings3.png)

    Klicken Sie auf **Speichern**.

1. Klicken Sie auf **Settings and Properties** (Einstellungen und Eigenschaften), um dem Dashboard die Seriennummer des Geräts hinzuzufügen:

    ![Dashboardkomponenten](media/tutorial-define-device-type/dashboardcomponents3.png)

1. Konfigurieren Sie die Komponente **Settings and Properties** (Einstellungen und Eigenschaften) mit den Informationen aus der folgenden Tabelle:

    | Einstellung                 | Wert         |
    | ----------------------- | ------------- |
    | Titel                   | Serial number |
    | Settings and Properties (Einstellungen und Eigenschaften) | Seriennummer |

    ![Einstellungen für die Seriennummerneigenschaft](media/tutorial-define-device-type/propertysettings1.png)

    Klicken Sie auf **Speichern**.

1. Klicken Sie auf **Settings and Properties** (Einstellungen und Eigenschaften), um dem Dashboard die Firmwareversion des Geräts hinzuzufügen:

    ![Dashboardkomponenten](media/tutorial-define-device-type/dashboardcomponents4.png)

1. Konfigurieren Sie die Komponente **Settings and Properties** (Einstellungen und Eigenschaften) mit den Informationen aus der folgenden Tabelle:

    | Einstellung                 | Wert            |
    | ----------------------- | ---------------- |
    | Titel                   | Firmware version |
    | Settings and Properties (Einstellungen und Eigenschaften) | Firmware Version |

    ![Einstellungen für die Seriennummerneigenschaft](media/tutorial-define-device-type/propertysettings2.png)

    Klicken Sie auf **Speichern**.

1. Deaktivieren Sie rechts oben auf der Seite den **Entwurfsmodus**, um das Dashboard als Bediener anzuzeigen.

## <a name="next-steps"></a>Nächste Schritte

In diesem Tutorial haben Sie Folgendes gelernt:

<!-- Repeat task list from intro -->
> [!div class="nextstepaction"]
> * Erstellen einer neuen Gerätevorlage
> * Hinzufügen von Telemetriedaten zu Ihrem Gerät
> * Anzeigen simulierter Telemetriedaten
> * Definieren von Geräteereignissen
> * Anzeigen simulierter Ereignisse
> * Definieren des Zustands
> * Anzeigen des simulierten Zustands
> * Verwenden von Geräteeigenschaften
> * Verwenden von Geräteeinstellungen

Nachdem Sie nun eine Gerätevorlage in Ihrer Azure IoT Central-Anwendung definiert haben, können Sie mit folgenden Schritten fortfahren:

* [Konfigurieren von Regeln und Aktionen für Ihr Gerät in Azure IoT Central](tutorial-configure-rules.md)
* [Anpassen der Azure IoT Central-Ansicht für Bediener](tutorial-customize-operator.md)
