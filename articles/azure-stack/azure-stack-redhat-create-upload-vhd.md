---
title: Erstellen und Hochladen einer VHD-Datei mit Red Hat Enterprise Linux zur Verwendung in Azure Stack | Microsoft-Dokumentation
description: Erfahren Sie, wie Sie eine virtuelle Azure-Festplatte (Virtual Hard Disk, VHD) erstellen und hochladen, die ein Red Hat-Linux-Betriebssystem enthält.
services: azure-stack
documentationcenter: ''
author: JeffGoldner
manager: BradleyB
editor: ''
tags: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/11/2018
ms.author: jeffgo
ms.openlocfilehash: 524c3b3b86e43b56e1d1f2a1653bdf7b82061022
ms.sourcegitcommit: fc64acba9d9b9784e3662327414e5fe7bd3e972e
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/12/2018
ms.locfileid: "34076614"
---
# <a name="prepare-a-red-hat-based-virtual-machine-for-azure-stack"></a>Vorbereiten eines auf Red Hat basierenden virtuellen Computers für Azure Stack
In diesem Artikel erfahren Sie, wie Sie einen auf Red Hat Enterprise Linux (RHEL) basierenden virtuellen Computer für die Verwendung in Azure Stack vorbereiten. In diesem Artikel werden die RHEL-Versionen 7.1 und höher behandelt. Darüber hinaus werden in diesem Artikel die Hypervisoren Hyper-V, KVM und VMware für die Vorbereitung vorgestellt.

Informationen zur Unterstützung von Red Hat Enterprise Linux finden Sie unter [Red Hat and Azure Stack: Frequently Asked Questions](https://access.redhat.com/articles/3413531) (Red Hat und Azure Stack: Häufig gestellte Fragen).

## <a name="prepare-a-red-hat-based-virtual-machine-from-hyper-v-manager"></a>Vorbereiten eines auf Red Hat basierenden virtuellen Computers über Hyper-V-Manager

### <a name="prerequisites"></a>Voraussetzungen
In diesem Abschnitt wird davon ausgegangen, dass Sie bereits eine ISO-Datei von der Red Hat-Website beschafft und das RHEL-Image auf einer virtuellen Festplatte (VHD) installiert haben. Weitere Informationen zum Installieren eines Betriebssystemimage mit dem Hyper-V-Manager finden Sie unter [Installieren der Hyper-V-Rolle und Konfigurieren eines virtuellen Computers](http://technet.microsoft.com/library/hh846766.aspx).

**Installationshinweise zu RHEL**

* Das VHDX-Format wird von Azure Stack nicht unterstützt. Azure unterstützt nur feste virtuelle Festplatten. Sie können Hyper-V Manager verwenden, um den Datenträger in das VHD-Format zu konvertieren, oder Sie können das convert-vhd-Cmdlet verwenden. Wählen Sie bei Verwendung von VirtualBox die Option **Feste Größe** und nicht die standardmäßig dynamisch zugeordnete Option, wenn Sie den Datenträger erstellen.
* Azure Stack unterstützt nur virtuelle Computer der ersten Generation. Sie können virtuelle Computer der 1. Generation vom VHDX- in das VHD-Dateiformat und von einem dynamisch erweiterbaren Datenträger in einen Datenträger mit fester Größe konvertieren. Die Generation eines virtuellen Computers können Sie nicht ändern. Weitere Informationen finden Sie unter [Should I create a generation 1 or 2 virtual machine in Hyper-V?](https://technet.microsoft.com/windows-server-docs/compute/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v) (Sollte ich einen virtuellen Computer der 1. oder 2. Generation in Hyper-V erstellen?).
* Die maximal zulässige Größe für die virtuelle Festplatte beträgt 1.023 GB.
* Beim Installieren des Linux-Betriebssystems wird empfohlen, anstelle von LVM (Logical Volume Manager) – bei vielen Installationen oftmals voreingestellt – die Standardpartitionen zu verwenden. Bei diesem Verfahren werden LVM-Namenskonflikte mit geklonten virtuellen Computern verhindert. Dies gilt besonders, falls Sie einen Betriebssystem-Datenträger zur Problembehandlung mit einem anderen virtuellen Computer verbinden müssen, der identisch ist.
* Kernel-Unterstützung für die Bereitstellung von UDF-Dateisystemen (Universal Disk Format) ist erforderlich. Beim ersten Starten unter Azure übergibt das Medium mit UDF-Formatierung, das an den Gast angefügt ist, die Bereitstellungskonfiguration an den virtuellen Linux-Computer. Der Azure-Linux-Agent muss das UDF-Dateisystem bereitstellen können, um dessen Konfiguration zu lesen und den virtuellen Computer bereitzustellen.
* Versionen des Linux-Kernels, die älter als 2.6.37 sind, weisen keine NUMA-Unterstützung (Non-Uniform Memory Access, nicht einheitlicher Speicherzugriff) unter Hyper-V mit höheren VM-Größen auf. Dieses Problem betrifft in erster Linie ältere Verteilungen, die den vorgeschalteten Red Hat 2.6.32 Kernel verwenden, und es wurde in RHEL 6.6 (Kernel 2.6.32-504) behoben. Für Systeme, auf denen benutzerdefinierte Kernel ausgeführt werden, die älter als 2.6.37 sind, bzw. RHEL-basierte Kernel, die älter als 2.6.32-504 sind, muss der Startparameter `numa=off` in der Kernel-Befehlszeile auf „grub.conf“ festgelegt werden. Weitere Informationen finden Sie unter Red Hat [KB 436883](https://access.redhat.com/solutions/436883).
* Konfigurieren Sie auf dem Betriebssystem-Datenträger keine Swap-Partition. Der Linux-Agent kann so konfiguriert werden, dass auf dem temporären Ressourcendatenträger eine Auslagerungsdatei erstellt wird.  Weitere Informationen hierzu finden Sie unter den folgenden Schritten.
* Alle VHDs in Azure benötigen eine virtuelle Größe, die auf 1 MB ausgerichtet ist. Beim Konvertieren von einem unformatierten Datenträger in VHD müssen Sie sicherstellen, dass die Größe des unformatierten Datenträgers ein Vielfaches von 1 MB vor der Konvertierung beträgt. Einzelheiten erfahren Sie im folgenden Schritt.
* cloud-init wird von Azure Stack nicht unterstützt. Ihre VM muss mit einer unterstützten Version des Microsoft Azure Linux-Agents konfiguriert sein.

### <a name="prepare-a-rhel-7-virtual-machine-from-hyper-v-manager"></a>Vorbereiten eines virtuellen RHEL 7-Computers über Hyper-V-Manager

1. Wählen Sie im Hyper-V-Manager den virtuellen Computer aus.

2. Klicken Sie auf **Verbinden** , um ein Konsolenfenster für den virtuellen Computer zu öffnen.

3. Erstellen oder bearbeiten Sie die Datei `/etc/sysconfig/network`, und fügen Sie ihr den folgenden Text hinzu:
   
        NETWORKING=yes
        HOSTNAME=localhost.localdomain

4. Erstellen oder bearbeiten Sie die Datei `/etc/sysconfig/network-scripts/ifcfg-eth0`, und fügen Sie ihr den folgenden Text hinzu:
   
        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no
    NM_CONTROLLED=no

5. Stellen Sie sicher, dass der Netzwerkdienst beim Booten startet, indem Sie den folgenden Befehl ausführen:

        # sudo systemctl enable network

6. Registrieren Sie mit dem folgenden Befehl das Red Hat-Abonnement, um die Installation von Paketen aus dem RHEL-Repository zu ermöglichen:

        # sudo subscription-manager register --auto-attach --username=XXX --password=XXX

7. Modifizieren Sie die Boot-Zeile des Kernels in Ihrer Grub-Konfiguration, um zusätzliche Kernel-Parameter für Azure einzubinden. Öffnen Sie für diese Änderung `/etc/default/grub` in einem Text-Editor, und bearbeiten Sie den Parameter `GRUB_CMDLINE_LINUX`. Beispiel: 
   
        GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
   
   Dadurch wird zudem sichergestellt, dass alle Konsolennachrichten zum ersten seriellen Port gesendet werden. Dieser kann Azure bei der Behebung von Fehlern unterstützen. Außerdem werden bei dieser Konfiguration die neuen RHEL 7-Benennungskonventionen für Netzwerkkarten deaktiviert. Neben den oben erwähnten Punkten ist es ratsam, die folgenden Parameter zu entfernen:
   
        rhgb quiet crashkernel=auto
   
    Weder der Graphical Boot noch der Quiet Boot sind in einer Cloudumgebung nützlich, in der alle Protokolle an den seriellen Port gesendet werden sollen. Sie können die Option `crashkernel` bei Bedarf konfiguriert lassen. Beachten Sie, dass die Menge an verfügbarem Arbeitsspeicher auf dem virtuellen Computer mit diesem Parameter um mindestens 128 MB reduziert wird. Dies kann bei kleineren virtuellen Computern problematisch sein.

8. Nachdem Sie die Bearbeitung von `/etc/default/grub`abgeschlossen haben, führen Sie den folgenden Befehl zum erneuten Erstellen der GRUB-Konfiguration aus:

        # sudo grub2-mkconfig -o /boot/grub2/grub.cfg

9. Stellen Sie sicher, dass der SSH-Server installiert und so konfiguriert ist, dass er beim Booten hochfährt. Ergänzen Sie `/etc/ssh/sshd_config` um die folgende Zeile:

        ClientAliveInterval 180

10. Das WALinuxAgent-Paket `WALinuxAgent-<version>` wurde in das Red Hat Extras-Repository übertragen. Aktivieren Sie das Extras-Repository, indem Sie den folgenden Befehl ausführen:

        # subscription-manager repos --enable=rhel-7-server-extras-rpms

11. Installieren Sie den Azure Linux-Agent, indem Sie den folgenden Befehl ausführen:

        # sudo yum install WALinuxAgent

        # sudo systemctl enable waagent.service

12. Erstellen Sie auf dem Betriebssystem-Datenträger keinen Auslagerungsbereich.

    Der Azure Linux-Agent kann den Auslagerungsbereich automatisch mit dem lokalen Ressourcendatenträger konfigurieren, der nach der Bereitstellung des virtuellen Computers in Azure mit dem virtuellen Computer verknüpft ist. Beachten Sie, dass der lokale Ressourcendatenträger ein temporärer Datenträger ist und geleert werden kann, wenn die Bereitstellung des virtuellen Computers aufgehoben wird. Passen Sie nach dem Installieren des Azure Linux-Agents im vorherigen Schritt die folgenden Parameter in `/etc/waagent.conf` entsprechend an:

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

13. Wenn Sie die Registrierung  des Abonnements aufheben möchten, führen Sie den folgenden Befehl aus:

        # sudo subscription-manager unregister

14. Führen Sie die folgenden Befehle aus, um den virtuellen Computer zurückzusetzen und ihn für die Bereitstellung in Azure vorzubereiten:

        # sudo waagent -force -deprovision

        # export HISTSIZE=0

        # logout

15. Klicken Sie im Hyper-V-Manager auf **Aktion** > **Herunterfahren**. Ihre Linux-VHD kann nun in Azure hochgeladen werden.


## <a name="prepare-a-red-hat-based-virtual-machine-from-kvm"></a>Vorbereiten eines auf Red Hat basierenden virtuellen Computers über KVM

### <a name="prepare-a-rhel-7-virtual-machine-from-kvm"></a>Vorbereiten eines virtuellen RHEL 7-Computers über KVM

1. Laden Sie das KVM-Image von RHEL 7 von der Red Hat-Website herunter. Bei diesem Verfahren wird RHEL 7 als Beispiel verwendet.

2. Legen Sie ein Stammkennwort fest.

    Generieren Sie ein verschlüsseltes Kennwort, und kopieren Sie die Ausgabe des Befehls:

        # openssl passwd -1 changeme

    Legen Sie ein Stammkennwort mit Guestfish fest:

        # guestfish --rw -a <image-name>
        > <fs> run
        > <fs> list-filesystems
        > <fs> mount /dev/sda1 /
        > <fs> vi /etc/shadow
        > <fs> exit

   Ändern Sie im zweiten Feld des Stammbenutzers "!!" in das verschlüsselte Kennwort.

3. Erstellen Sie einen virtuellen Computer in KVM über das qcow2-Image. Legen Sie den Datenträgertyp auf **qcow2** und als Gerätemodell der virtuellen Netzwerkschnittstelle **virtio** fest. Starten Sie anschließend den virtuellen Computer, und melden Sie sich als Stammbenutzer an.

4. Erstellen oder bearbeiten Sie die Datei `/etc/sysconfig/network`, und fügen Sie ihr den folgenden Text hinzu:
   
        NETWORKING=yes
        HOSTNAME=localhost.localdomain

5. Erstellen oder bearbeiten Sie die Datei `/etc/sysconfig/network-scripts/ifcfg-eth0`, und fügen Sie ihr den folgenden Text hinzu:
   
        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no
        NM_CONTROLLED=no

6. Stellen Sie sicher, dass der Netzwerkdienst beim Booten startet, indem Sie den folgenden Befehl ausführen:

        # sudo systemctl enable network

7. Registrieren Sie mit dem folgenden Befehl das Red Hat-Abonnement, um die Installation von Paketen aus dem RHEL-Repository zu ermöglichen:

        # subscription-manager register --auto-attach --username=XXX --password=XXX

8. Modifizieren Sie die Boot-Zeile des Kernels in Ihrer Grub-Konfiguration, um zusätzliche Kernel-Parameter für Azure einzubinden. Öffnen Sie für diese Konfiguration `/etc/default/grub` in einem Text-Editor, und bearbeiten Sie den Parameter `GRUB_CMDLINE_LINUX`. Beispiel: 
   
        GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
   
   Mit diesem Befehl wird zudem sichergestellt, dass alle Konsolennachrichten zum ersten seriellen Port gesendet werden. Dieser kann Azure bei der Behebung von Fehlern unterstützen. Außerdem werden mit dem Befehl die neuen RHEL 7-Benennungskonventionen für Netzwerkkarten deaktiviert. Neben den oben erwähnten Punkten ist es ratsam, die folgenden Parameter zu entfernen:
   
        rhgb quiet crashkernel=auto
   
    Weder der Graphical Boot noch der Quiet Boot sind in einer Cloudumgebung nützlich, in der alle Protokolle an den seriellen Port gesendet werden sollen. Sie können die Option `crashkernel` bei Bedarf konfiguriert lassen. Beachten Sie, dass die Menge an verfügbarem Arbeitsspeicher auf dem virtuellen Computer mit diesem Parameter um mindestens 128 MB reduziert wird. Dies kann bei kleineren virtuellen Computern problematisch sein.

9. Nachdem Sie die Bearbeitung von `/etc/default/grub`abgeschlossen haben, führen Sie den folgenden Befehl zum erneuten Erstellen der GRUB-Konfiguration aus:

        # grub2-mkconfig -o /boot/grub2/grub.cfg

10. Fügen Sie „initramfs“ Hyper-V-Module hinzu:

    Bearbeiten Sie `/etc/dracut.conf` , und fügen Sie Inhalt hinzu:

        add_drivers+="hv_vmbus hv_netvsc hv_storvsc"

    Erstellen Sie „initramfs“ neu:

        # dracut -f -v

11. Deinstallieren Sie die Cloud-Initialisierung:

        # yum remove cloud-init

12. Stellen Sie sicher, dass der SSH-Server installiert und konfiguriert ist, damit er beim Starten hochfährt:

        # systemctl enable sshd

    Ändern Sie die Datei „/etc/ssh/sshd_config“, sodass sie die folgenden Zeilen enthält:

        PasswordAuthentication yes
        ClientAliveInterval 180

13. Das WALinuxAgent-Paket `WALinuxAgent-<version>` wurde in das Red Hat Extras-Repository übertragen. Aktivieren Sie das Extras-Repository, indem Sie den folgenden Befehl ausführen:

        # subscription-manager repos --enable=rhel-7-server-extras-rpms

14. Installieren Sie den Azure Linux-Agent, indem Sie den folgenden Befehl ausführen:

        # yum install WALinuxAgent

    Aktivieren Sie den Waagent-Dienst:

        # systemctl enable waagent.service

15. Erstellen Sie auf dem Betriebssystem-Datenträger keinen Auslagerungsbereich.

    Der Azure Linux-Agent kann den Auslagerungsbereich automatisch mit dem lokalen Ressourcendatenträger konfigurieren, der nach der Bereitstellung des virtuellen Computers in Azure mit dem virtuellen Computer verknüpft ist. Beachten Sie, dass der lokale Ressourcendatenträger ein temporärer Datenträger ist und geleert werden kann, wenn die Bereitstellung des virtuellen Computers aufgehoben wird. Passen Sie nach dem Installieren des Azure Linux-Agents im vorherigen Schritt die folgenden Parameter in `/etc/waagent.conf` entsprechend an:

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

16. Heben Sie das Abonnement (falls erforderlich) auf, indem Sie den folgenden Befehl ausführen:

        # subscription-manager unregister

17. Führen Sie die folgenden Befehle aus, um den virtuellen Computer zurückzusetzen und ihn für die Bereitstellung in Azure vorzubereiten:

        # sudo waagent -force -deprovision

        # export HISTSIZE=0

        # logout

18. Fahren Sie den virtuellen Computer in KVM herunter.

19. Konvertieren Sie das qcow2-Image in das VHD-Format.

> [!NOTE]
> In den Versionen qemu-img-Versionen > oder = 2.2.1 taucht ein bekannter Bug auf, der zu Fehlern bei der Formatierung der VHD-führt. Dieses Problem wurde in QEMU 2.6 behoben. Sie sollten entweder qemu-img 2.2.0 oder niedriger verwenden oder auf 2.6 oder höher aktualisieren. Referenz: https://bugs.launchpad.net/qemu/+bug/1490611.
>


    First convert the image to raw format:

        # qemu-img convert -f qcow2 -O raw rhel-7.4.qcow2 rhel-7.4.raw

    Make sure that the size of the raw image is aligned with 1 MB. Otherwise, round up the size to align with 1 MB:

        # MB=$((1024*1024))
        # size=$(qemu-img info -f raw --output json "rhel-7.4.raw" | \
          gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')

        # rounded_size=$((($size/$MB + 1)*$MB))
        # qemu-img resize rhel-7.4.raw $rounded_size

    Convert the raw disk to a fixed-sized VHD:

        # qemu-img convert -f raw -o subformat=fixed -O vpc rhel-7.4.raw rhel-7.4.vhd

    Or, with qemu version **2.6+** include the `force_size` option:

        # qemu-img convert -f raw -o subformat=fixed,force_size -O vpc rhel-7.4.raw rhel-7.4.vhd


## <a name="prepare-a-red-hat-based-virtual-machine-from-vmware"></a>Vorbereiten eines auf Red Hat basierenden virtuellen Computers über VMware
### <a name="prerequisites"></a>Voraussetzungen
In diesem Abschnitt wird davon ausgegangen, dass Sie bereits einen virtuellen Computer von RHEL in VMware installiert haben. Weitere Informationen zum Installieren eines Betriebssystems in VMware finden Sie im [VMware-Installationshandbuch für Gastbetriebssysteme](http://partnerweb.vmware.com/GOSIG/home.html).

* Beim Installieren des Linux-Betriebssystems wird empfohlen, anstelle von LVM – bei vielen Installationen oftmals voreingestellt – die Standardpartitionen zu verwenden. Bei diesem Verfahren werden LVM-Namenskonflikte mit geklonten virtuellen Computern vermieden. Dies gilt besonders, falls Sie einen Betriebssystem-Datenträger zur Problembehandlung mit einem anderen virtuellen Computer verbinden müssen. LVM oder RAID können bei Bedarf auf Datenträgern verwendet werden.
* Konfigurieren Sie auf dem Betriebssystem-Datenträger keine Swap-Partition. Sie können den Linux-Agent zur Erstellung einer Auslagerungsdatei auf dem temporären Ressourcendatenträger konfigurieren. Weitere Informationen hierzu finden Sie in den folgenden Schritten.
* Wählen Sie beim Erstellen der virtuellen Festplatte **Virtuellen Datenträger als einzelne Datei speichern**aus.


### <a name="prepare-a-rhel-7-virtual-machine-from-vmware"></a>Vorbereiten eines virtuellen RHEL 7-Computers über VMWare
1. Erstellen oder bearbeiten Sie die Datei `/etc/sysconfig/network`, und fügen Sie ihr den folgenden Text hinzu:
   
        NETWORKING=yes
        HOSTNAME=localhost.localdomain

2. Erstellen oder bearbeiten Sie die Datei `/etc/sysconfig/network-scripts/ifcfg-eth0`, und fügen Sie ihr den folgenden Text hinzu:
   
        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no
        NM_CONTROLLED=no

3. Stellen Sie sicher, dass der Netzwerkdienst beim Booten startet, indem Sie den folgenden Befehl ausführen:

        # sudo chkconfig network on

4. Registrieren Sie mit dem folgenden Befehl das Red Hat-Abonnement, um die Installation von Paketen aus dem RHEL-Repository zu ermöglichen:

        # sudo subscription-manager register --auto-attach --username=XXX --password=XXX

5. Modifizieren Sie die Boot-Zeile des Kernels in Ihrer Grub-Konfiguration, um zusätzliche Kernel-Parameter für Azure einzubinden. Öffnen Sie für diese Änderung `/etc/default/grub` in einem Text-Editor, und bearbeiten Sie den Parameter `GRUB_CMDLINE_LINUX`. Beispiel: 
   
        GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
   
   Durch diese Konfiguration wird zudem sichergestellt, dass alle Konsolennachrichten zum ersten seriellen Port gesendet werden. Dieser kann Azure bei der Behebung von Fehlern unterstützen. Außerdem werden die neuen RHEL 7-Benennungskonventionen für Netzwerkkarten deaktiviert. Neben den oben erwähnten Punkten ist es ratsam, die folgenden Parameter zu entfernen:
   
        rhgb quiet crashkernel=auto
   
    Weder der Graphical Boot noch der Quiet Boot sind in einer Cloudumgebung nützlich, in der alle Protokolle an den seriellen Port gesendet werden sollen. Sie können die Option `crashkernel` bei Bedarf konfiguriert lassen. Beachten Sie, dass die Menge an verfügbarem Arbeitsspeicher auf dem virtuellen Computer mit diesem Parameter um mindestens 128 MB reduziert wird. Dies kann bei kleineren virtuellen Computern problematisch sein.

6. Nachdem Sie die Bearbeitung von `/etc/default/grub`abgeschlossen haben, führen Sie den folgenden Befehl zum erneuten Erstellen der GRUB-Konfiguration aus:

        # sudo grub2-mkconfig -o /boot/grub2/grub.cfg

7. Fügen Sie „initramfs“ Hyper-V-Module hinzu.

    Bearbeiten Sie `/etc/dracut.conf`und fügen Sie Inhalt hinzu:

        add_drivers+="hv_vmbus hv_netvsc hv_storvsc"

    Erstellen Sie „initramfs“ neu:

        # dracut -f -v

8. Stellen Sie sicher, dass der SSH-Server installiert und konfiguriert ist, damit er beim Booten hochfährt. Dies ist normalerweise die Standardeinstellung. Ergänzen Sie `/etc/ssh/sshd_config` um die folgende Zeile:

        ClientAliveInterval 180

9. Das WALinuxAgent-Paket `WALinuxAgent-<version>` wurde in das Red Hat Extras-Repository übertragen. Aktivieren Sie das Extras-Repository, indem Sie den folgenden Befehl ausführen:

        # subscription-manager repos --enable=rhel-7-server-extras-rpms

10. Installieren Sie den Azure Linux-Agent, indem Sie den folgenden Befehl ausführen:

        # sudo yum install WALinuxAgent

        # sudo systemctl enable waagent.service

11. Erstellen Sie auf dem Betriebssystem-Datenträger keinen Auslagerungsbereich.

    Der Azure Linux-Agent kann den Auslagerungsbereich automatisch mit dem lokalen Ressourcendatenträger konfigurieren, der nach der Bereitstellung des virtuellen Computers in Azure mit dem virtuellen Computer verknüpft ist. Beachten Sie, dass der lokale Ressourcendatenträger ein temporärer Datenträger ist und geleert werden kann, wenn die Bereitstellung des virtuellen Computers aufgehoben wird. Passen Sie nach dem Installieren des Azure Linux-Agents im vorherigen Schritt die folgenden Parameter in `/etc/waagent.conf` entsprechend an:

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

12. Wenn Sie die Registrierung  des Abonnements aufheben möchten, führen Sie den folgenden Befehl aus:

        # sudo subscription-manager unregister

13. Führen Sie die folgenden Befehle aus, um den virtuellen Computer zurückzusetzen und ihn für die Bereitstellung in Azure vorzubereiten:

        # sudo waagent -force -deprovision

        # export HISTSIZE=0

        # logout

14. Fahren Sie den virtuellen Computer herunter, und konvertieren Sie die VMDK-Datei in das VHD-Format.

> [!NOTE]
> In den Versionen qemu-img-Versionen > oder = 2.2.1 taucht ein bekannter Bug auf, der zu Fehlern bei der Formatierung der VHD-führt. Dieses Problem wurde in QEMU 2.6 behoben. Sie sollten entweder qemu-img 2.2.0 oder niedriger verwenden oder auf 2.6 oder höher aktualisieren. Referenz: https://bugs.launchpad.net/qemu/+bug/1490611.
>


    First convert the image to raw format:

        # qemu-img convert -f vmdk -O raw rhel-7.4.vmdk rhel-7.4.raw

    Make sure that the size of the raw image is aligned with 1 MB. Otherwise, round up the size to align with 1 MB:

        # MB=$((1024*1024))
        # size=$(qemu-img info -f raw --output json "rhel-7.4.raw" | \
          gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')

        # rounded_size=$((($size/$MB + 1)*$MB))
        # qemu-img resize rhel-7.4.raw $rounded_size

    Convert the raw disk to a fixed-sized VHD:

        # qemu-img convert -f raw -o subformat=fixed -O vpc rhel-7.4.raw rhel-7.4.vhd

    Or, with qemu version **2.6+** include the `force_size` option:

        # qemu-img convert -f raw -o subformat=fixed,force_size -O vpc rhel-7.4.raw rhel-7.4.vhd


## <a name="prepare-a-red-hat-based-virtual-machine-from-an-iso-by-using-a-kickstart-file-automatically"></a>Automatisches Vorbereiten eines auf Red Hat basierenden virtuellen Computers aus einer ISO-Datei mit einer Kickstart-Datei
### <a name="prepare-a-rhel-7-virtual-machine-from-a-kickstart-file"></a>Vorbereiten eines virtuellen RHEL 7-Computers über eine Kickstart-Datei

1.  Erstellen Sie eine Kickstart-Datei mit dem folgenden Inhalt, und speichern Sie die Datei. Weitere Informationen zur Kickstart-Installation finden Sie im [Kickstart-Installationshandbuch](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/chap-kickstart-installations.html).

        # Kickstart for provisioning a RHEL 7 Azure VM

        # System authorization information
          auth --enableshadow --passalgo=sha512

        # Use graphical install
        text

        # Do not run the Setup Agent on first boot
        firstboot --disable

        # Keyboard layouts
        keyboard --vckeymap=us --xlayouts='us'

        # System language
        lang en_US.UTF-8

        # Network information
        network  --bootproto=dhcp

        # Root password
        rootpw --plaintext "to_be_disabled"

        # System services
        services --enabled="sshd,waagent,NetworkManager"

        # System timezone
        timezone Etc/UTC --isUtc --ntpservers 0.rhel.pool.ntp.org,1.rhel.pool.ntp.org,2.rhel.pool.ntp.org,3.rhel.pool.ntp.org

        # Partition clearing information
        clearpart --all --initlabel

        # Clear the MBR
        zerombr

        # Disk partitioning information
        part /boot --fstype="xfs" --size=500
        part / --fstyp="xfs" --size=1 --grow --asprimary

        # System bootloader configuration
        bootloader --location=mbr

        # Firewall configuration
        firewall --disabled

        # Enable SELinux
        selinux --enforcing

        # Don't configure X
        skipx

        # Power down the machine after install
        poweroff

        %packages
        @base
        @console-internet
        chrony
        sudo
        parted
        -dracut-config-rescue

        %end

        %post --log=/var/log/anaconda/post-install.log

        #!/bin/bash

        # Register Red Hat Subscription
        subscription-manager register --username=XXX --password=XXX --auto-attach --force

        # Install latest repo update
        yum update -y

        # Enable extras repo
        subscription-manager repos --enable=rhel-7-server-extras-rpms

        # Install WALinuxAgent
        yum install -y WALinuxAgent

        # Unregister Red Hat subscription
        subscription-manager unregister

        # Enable waaagent at boot-up
        systemctl enable waagent

        # Disable the root account
        usermod root -p '!!'

        # Configure swap in WALinuxAgent
        sed -i 's/^\(ResourceDisk\.EnableSwap\)=[Nn]$/\1=y/g' /etc/waagent.conf
        sed -i 's/^\(ResourceDisk\.SwapSizeMB\)=[0-9]*$/\1=2048/g' /etc/waagent.conf

        # Set the cmdline
        sed -i 's/^\(GRUB_CMDLINE_LINUX\)=".*"$/\1="console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300"/g' /etc/default/grub

        # Enable SSH keepalive
        sed -i 's/^#\(ClientAliveInterval\).*$/\1 180/g' /etc/ssh/sshd_config

        # Build the grub cfg
        grub2-mkconfig -o /boot/grub2/grub.cfg

        # Configure network
        cat << EOF > /etc/sysconfig/network-scripts/ifcfg-eth0
        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no
        NM_CONTROLLED=no
        EOF

        # Deprovision and prepare for Azure
        waagent -force -deprovision

        %end

2. Legen Sie die Kickstart-Datei an einem Speicherort ab, an dem das Installationssystem darauf zugreifen kann.

3. Erstellen Sie in Hyper-V Manager einen neuen virtuellen Computer. Wählen Sie auf der Seite **Virtuelle Festplatte verbinden** die Option **Virtuelle Festplatte später zuordnen**, und schließen Sie den Assistenten für neue virtuelle Computer ab.

4. Öffnen Sie die VM-Einstellungen:

    a.  Ordnen Sie dem virtuellen Computer eine neue virtuelle Festplatte zu. Wählen Sie **VHD-Format** und **Feste Größe** aus.

    b.  Ordnen Sie die ISO-Installationsdatei dem DVD-Laufwerk zu.

    c.  Legen Sie fest, dass BIOS von der CD starten soll.

5. Starten Sie den virtuellen Computer. Wenn die Installationsanweisungen angezeigt werden, drücken Sie die **TAB-TASTE** , um die Startoptionen zu konfigurieren.

6. Geben Sie am Ende der Startoptionen `inst.ks=<the location of the kickstart file>` ein und drücken Sie die **EINGABETASTE**.

7. Warten Sie, bis die Installation abgeschlossen ist. Wenn sie abgeschlossen ist, wird der virtuelle Computer automatisch heruntergefahren. Ihre Linux-VHD kann nun in Azure hochgeladen werden.

## <a name="known-issues"></a>Bekannte Probleme
### <a name="the-hyper-v-driver-could-not-be-included-in-the-initial-ram-disk-when-using-a-non-hyper-v-hypervisor"></a>Der Hyper-V-Treiber konnte dem anfänglichen RAM-Datenträger nicht hinzugefügt werden, wenn ein anderer Hypervisor als ein Hyper-V-Hypervisor verwendet wurde.

In einigen Fällen enthalten Linux-Installationsprogramme unter Umständen keine Treiber für Hyper-V auf dem anfänglichen RAM-Datenträger („initrd“ oder „initramfs“), sofern von Linux nicht die Ausführung in einer Hyper-V-Umgebung erkannt wird.

Wenn ein anderes Virtualisierungssystem (z.B. Virtualbox, Xen usw.) zur Vorbereitung des Linux-Image verwendet wird, müssen Sie unter Umständen „initrd“ neu erstellen, um sicherzustellen, dass mindestens die Kernelmodule „hv_vmbus“ und „hv_storvsc“ auf dem anfänglichen RAM-Datenträger verfügbar sind. Dies ist zumindest auf Systemen, die auf der Red Hat-Upstream-Verteilung basieren, ein bekanntes Problem.

Um dieses Problem zu beheben, müssen Sie „initramfs“ Hyper-V-Module hinzufügen und das Archiv neu erstellen:

Bearbeiten Sie `/etc/dracut.conf`, und fügen Sie den folgenden Inhalt hinzu:

        add_drivers+="hv_vmbus hv_netvsc hv_storvsc"

Erstellen Sie „initramfs“ neu:

        # dracut -f -v

Weitere Informationen finden Sie unter [Neuerstellung von „initramfs“](https://access.redhat.com/solutions/1958).

## <a name="next-steps"></a>Nächste Schritte
Sie können jetzt mit Ihrer Red Hat Enterprise Linux-VHD-Datei neue virtuelle Azure-Computer in Azure Stack erstellen. Ist dies das erste Mal, dass Sie die VHD-Datei in Azure Stack hochladen, lesen Sie [Erstellen und Veröffentlichen von Marketplace-Elementen mithilfe des Marketplace-Toolkits](azure-stack-marketplace-publisher.md).

Weitere Informationen zu den Hypervisoren, die zum Ausführen von Red Hat Enterprise Linux zertifiziert sind, finden Sie auf der [Red Hat-Website](https://access.redhat.com/certified-hypervisors).
