# Projekt: LFS-Ansible

LFS-Ansible wurde erstellt um die Verwaltung und Konfiguration von
Linux-Systemen[^1] in der [LFS](https://www.liebfrauenschule.de/) zu
automatisieren. [Ansible](https://docs.ansible.com/ansible/latest/index.html)
ist ein 
leistungsstarkes Open-Source-Tool, das es ermöglicht, Systeme zu
konfigurieren, Software zu installieren und Aufgaben zu
automatisieren.

## Projektstruktur
Das Projekt ist in verschiedene Verzeichnisse und Dateien unterteilt,
die jeweils spezifische Funktionen und Aufgaben erfüllen. In einer
künftigen Überarbeitung sollen die Playbooks auf Basis von
[roles](https://docs.ansible.com/ansible/latest/getting_started/basic_concepts.html#roles)
stärker modularisiert
werden.   
Hier ist
eine Übersicht über die aktuelle Struktur:

### Hauptverzeichnis
- **.git**: Dieses Verzeichnis enthält die Versionskontrollinformationen für das Projekt.
- **.gitignore**: Diese Datei spezifiziert, welche Dateien und Verzeichnisse von Git ignoriert werden sollen.
- **LICENSE**: Die Lizenzdatei für das Projekt.
- **README.md**: Diese Datei, die eine Einführung und Übersicht über das Projekt bietet.

### Verzeichnisse
- **data/**: Dieses Verzeichnis enthält Konfigurationsdateien und andere notwendige Daten, die von den Playbooks verwendet werden.
  - **etc/**: Enthält Systemkonfigurationsdateien.
  - **klausur/**: Enthält Dateien, die für Prüfungen oder Tests verwendet werden.
  - **udev/**, **winbind/**, **nbc/**: Weitere Unterverzeichnisse mit spezifischen Konfigurationsdateien.

- **packages/**: Dieses Verzeichnis enthält Paketdateien, die für die Installation erforderlich sind, z.B. `TigerJython.tar.gz`.

- **vorlagen/**: Enthält Vorlagen, die ich von einem anderen, [ähnlich
  gelagerten Projekt](https://github.com/feschoppe/ansible-manage-ubuntu-clients) geklaut habe.

### Wichtige Dateien
- **ansible.cfg**: Die Konfigurationsdatei für Ansible, die globale Einstellungen definiert.
- **.yml-Dateien**: Ansible Playbooks, die verschiedene Automatisierungsaufgaben definieren.
- **.sh-Dateien**: Shell-Skripte, die für bestimmte Aufgaben verwendet werden, wie z.B. das Überprüfen von Dateien oder das Senden von Wake-on-LAN-Paketen.

## Kategorien von Playbooks
Die Playbooks können in verschiedene Kategorien unterteilt werden, je nach ihrer Funktion:

### Systemkonfiguration
- **check-config.yml**: Überprüft grundlegende Systemkonfigurationen wie die Aktion beim Schließen des Laptopdeckels und die WLAN-Verbindung.
- **config-power-off.yml**: Konfiguriert die Energieeinstellungen, um den Laptop beim Schließen des Deckels auszuschalten.

### Paketverwaltung
- **apt-install-filius.yml**: Installiert das Programm Filius und stellt sicher, dass es korrekt installiert ist.
- **apt_installiere_programm.yml**: Installiert eine Liste von Programmen über den Paketmanager APT.
- **check-pip-venv.yml**: Prüft, ob `python3 -m venv` pip korrekt bootstrappt, und repariert
  bei defektem `ensurepip` die venv-/pip-Pakete (Reinstall). Mit `-e repair=false` nur Diagnose.

### Netzwerkverwaltung
- **check-wol.yml**: Überprüft und konfiguriert die Wake-on-LAN-Einstellungen für Netzwerkgeräte.
- **wake-devices.sh**: Ein Skript, das Wake-on-LAN-Pakete an Geräte im Netzwerk sendet.
- **check-proxy.yml**: Proxy-Diagnose auf Schülergeräten. Verteilt `files/proxy_test.py`,
  führt es aus und sammelt Ergebnisse als CSV unter `results/`. Prüft u.a. ob „Proxy erzwingen"
  aktiv ist und ob `kurswahl.lfs-ol.de` über den Squid-Proxy erreichbar ist.
  Hintergrund: [iserv-proxy-kurswahl.md](../../../../../admin/iserv-proxy-kurswahl.md)

### Speicherplatzverwaltung
- **disk-usage.yml**: Überwacht den Speicherplatz auf der `/var`-Partition und bereinigt bei Bedarf alte Dateien.
- **check_var_part.yml**: Überprüft den Speicherplatz auf der `/var`-Partition und führt bei Bedarf eine Bereinigung durch.

### Benutzerverwaltung
- **make_sudo.yml**: Fügt den Benutzer `mlo` zur `sudoers`-Datei hinzu, um ihm Administratorrechte zu gewähren.

### Dateien und Verzeichnisse
- **fetch_desktop-files.yml**: Kopiert Dateien vom Desktop eines Benutzers auf den lokalen Rechner.
- **transfer_file_to_tmp-user.yml**: Überträgt Dateien auf den Desktop eines Benutzers.

## Verwendung
Um das Projekt zu nutzen, benötigst du Ansible auf deinem System. Die
Playbooks können mit dem Befehl `ansible-playbook <playbook-name>.yml`
ausgeführt werden. Stelle sicher, dass alle benötigten Dateien und
Konfigurationen vorhanden sind, bevor du ein Playbook ausführst. 

Ein spezifischer Mechanismus, der im Rahmen dieses Projekts verwendet
wird, ist die Möglichkeit, Playbooks gezielt für bestimmte Hostgruppen
auszuführen und die Parallelisierung zu steuern. Ein Beispielaufruf
sieht wie folgt aus: 

```bash
ansible-playbook -f 33 <playbook-name>.yml -e "target_group=INFacerLAN"
```

#### Analyse des Befehls:
- **`-f 33`**: Diese Option legt die Anzahl der Forks fest, die
  Ansible verwenden soll. Forks bestimmen, wie viele Hosts
  gleichzeitig konfiguriert werden können. In diesem Fall werden 33
  Hosts parallel verarbeitet, was die Geschwindigkeit der
  Bereitstellung auf mehreren Maschinen erhöht. 

- **`<playbook-name>.yml`**: Dies ist das Playbook, das ausgeführt
  wird. Jedes Playbook hat eine spezifische Aufgabe, wie z.B. das
  Konfigurieren von Energieeinstellungen oder das Installieren von
  Software. 

- **`-e "target_group=INFacerLAN"`**: Diese Option übergibt eine extra
  Variable an das Playbook. In diesem Fall wird die Variable
  `target_group` auf `INFacerLAN` gesetzt, was bedeutet, dass das
  Playbook auf die Hostgruppe `INFacerLAN` angewendet wird. Dies
  ermöglicht es, spezifische Gruppen von Hosts gezielt anzusprechen
  und zu konfigurieren. Diese Flexibilität ist besonders nützlich,
  wenn du mehrere Gruppen von Maschinen mit unterschiedlichen
  Konfigurationen verwalten möchtest.   
  Die möglichen `target_group`s sind in `data/hosts`
  definiert. (s. auch `ansible.cfg`: `inventory = data/hosts`)

Der Mechanismus zur Angabe von Hostgruppen und zur Parallelisierung
ist für alle Playbooks anwendbar und hilft dabei, die Ausführung
effizient und zielgerichtet zu gestalten. Einige Playbooks könnten
diesen Mechanismus noch nicht vollständig implementiert haben.    
Suche nach 
```YAML
  vars:
    target_group: "{{ target_group | default('INFacerWLAN') }}"
  hosts: "{{ target_group }}"
```

## Verwendete ansible Mechanismen

### Reboot or shutdown

Die meisten Playbooks enthalten einen `post_tasks` Abschnitt, der den
Zielrechner durchbootet, wenn Änderungen durchgeführt wurden oder
herunterfährt, wenn keine Änderungen notwendig waren.  
Dies ist vor allem für unsere Schüler-Laptops sinnvoll.

```YAML
  post_tasks:
    - name: Neustart des Systems, wenn Änderungen vorgenommen wurden
      reboot:
      when: wol_status.changed

    - name: Herunterfahren des Systems, wenn keine Änderungen erforderlich sind
      command: shutdown now
      when: not wol_status.changed
```

[^1]: Es ist mit ansible auch möglich [Windows](https://docs.ansible.com/ansible/latest/os_guide/intro_windows.html) zu steuern. Das wird vermutlich ein Thema für die Zukunft, wenn wir unsere IT [virtualisieren](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_module.html).
