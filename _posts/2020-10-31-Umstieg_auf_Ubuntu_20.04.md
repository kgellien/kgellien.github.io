# 2020-10-31 Umstieg auf Ubuntu-20.04

Es folgen ein paar Notizen und Anmerkungen zum Update von Ubuntu 18.04 auf 20.04.


## Clipboard

Gibt es nun mehrere Clipboards? `ctrl-v` und `ctrl-shift-insert` können unterschiedliche Ergebnisse liefern!

## Temperaturüberwachung

- [https://linoxide.com/linux-how-to/get-cpu-temperature-on-ubuntu-linux/](https://linoxide.com/linux-how-to/get-cpu-temperature-on-ubuntu-linux/)
- [https://itsubuntu.com/useful-tools-to-monitor-cpu-and-gpu-temperature-in-ubuntu/](https://itsubuntu.com/useful-tools-to-monitor-cpu-and-gpu-temperature-in-ubuntu/)
- [https://www.tecmint.com/monitor-cpu-and-gpu-temperature-in-ubuntu/](https://www.tecmint.com/monitor-cpu-and-gpu-temperature-in-ubuntu/)


## Docker

Achtung: `docker-compose` setzt auf Python auf, d.h. bei Nutzung von Anaconda muss es auch unter Nutzung von Anaconda installiert werden!

Dies geschieht dann, bei userspezifischen Anaconda-Installation per `pip install docker-compose`.


## Ansible vs. Anaconda

Auch hier muss eine klare Entscheidung für ein Python-System getroffen werden.
Hier also wieder Anaconda, was bedeutet, dass Ansible per

```bash
sudo conda install ansible
```

installiert werden muss, da es sonst über PATH nicht erreichbar ist.

**Achtung:** Ohne sudo:

```bash
EnvironmentNotWritableError: The current user does not have write permissions to the target environment.
  environment location: /usr/local/Anaconda3-2020.07
  uid: 1000
  gid: 1000
```,

mit sudo

```bash
sudo: conda: command not found
```

Neuer Versuch:

```bash
/home/gellien/work-git/installer-ubuntu/ansible (master) > sudo /usr/local/Anaconda3-2020.07/bin/conda install ansible
Collecting package metadata (current_repodata.json): done
Solving environment: failed with initial frozen solve. Retrying with flexible solve.
Collecting package metadata (repodata.json): done
Solving environment: failed with initial frozen solve. Retrying with flexible solve.

PackagesNotFoundError: The following packages are not available from current channels:

  - ansible

Current channels:

  - https://repo.anaconda.com/pkgs/main/linux-64
  - https://repo.anaconda.com/pkgs/main/noarch
  - https://repo.anaconda.com/pkgs/r/linux-64
  - https://repo.anaconda.com/pkgs/r/noarch

To search for alternate channels that may provide the conda package you're
looking for, navigate to

    https://anaconda.org

and use the search bar at the top of the page.
```

Dies, obwohl *ohne* sudo (und vollst. Pfad) ansible als downloadbar von conda-forge geführt wird!



Achtung: Es kann passieren, dass manche Packages (noch) nicht über Anaconda installierbar sind.
Auch muss noch genau geschaut werden, was alles Über `PATH` bzw. `PYTHONPATH` zugänglich gemacht werden muss.

**==>** Installiere Anaconda (wieder) lokal für den aktuellen User. Dann funktioniert `conda install ansible` direkt.

Analog wird dann auch `MkDocs` installiert: `conda install mkdocs`. Hier stellen sich nämlich ansonsten auch obige Probleme.
Zunächst hatte ich `mkdocs` über `sudo apt install mkdocs` installiert. Dies ging auf das installierte Default-Python, was nach der Installation von Anaconda erst später im Pfad lag und deshalb nicht mehr angesprochen werden konnte.

Globale Installation von Anaconda ist damit für mich vorläufig gescheitert.


## Jenkins

- [https://linuxhint.com/install_jenkins_docker_ubuntu/](https://linuxhint.com/install_jenkins_docker_ubuntu/)
- [https://github.com/jenkinsci/docker/blob/master/README.md](https://github.com/jenkinsci/docker/blob/master/README.md)


## Libreoffice

Per `apt` ist nur Version 6.4 erhältlich, währen `snap` Version 7 liefert.
Durch Experimente mit verschiedenen Desktops wurde mittelbar auch LibreOffice 6.4 installiert.
`sudo apt remove libreoffice` funktionierte nicht, es musste `sudo apt remove '^libreoffice-.*'` verwendet werden.


## Git-Server

Bisher habe ich meine Repos i.W. auf einem USB-Stick gesammelt und konsolidiert. (Ich möchte meine privaten Repos nicht bei BitBucket oder GitHub lagern.)
Da ich die Repos jeweils auf mehreren Rechnern liegen hatte und ich über den Stick regelmäßig abgeglichen hatte, war Datensicherheit bisher kein Thema.
Da ich i.d.R. auch nicht parallel gearbeitet habe, war es auch kein Verwaltungsproblem.

Trotzdem wollte ich die Gelegenheit nutzen, (wieder) zu einer zentralen Stelle der Wahrheit zu kommen.

Dazu habe ich mir Gitlab und Gitea, jeweils in Docker-Containern, näher angeschaut.

Dabei fiel mir auf, das Gitlab *deutlich* länger zum hochfahren braucht.

Gitlab basiert auf Rails, während Gitea in Go geschrieben ist.
Außerdem bietet Gitlab noch integriertes CI/CD, was für mich aktuell allerdings nicht relevant ist.
Dazu werde ich mich demnächst wieder mit Jenkins beschäftigen.

Beide habe ich mit PostgreSQL als Datenbank angeschaut.

Der wesentliche Grund für mich, zunächst doch noch bei Gitlab zu bleiben bestand darin, dass es Gitlab erlaubt, Repositories direkt zu pushen, *ohne* das sie in Gitlab vorher angelegt wurden.
Da Gitea dieses Feauture inzwischen auch beherrscht, entfiel dieser Grund und ich habe mich aktuell für Gitea entschieden.


### Import

Ich habe lokal meine Git-Repos alle in ein gemeinsames Arbeitsverzeichnis geclont und ausgecheckt.
Damit konnte ich den Massenimport und die passende Konfiguration der Repos einfach per Shell-Skripting vornehmen:

```bash
owner=kgellien
remote=gitea
server=localhost
for repo in $(/usr/bin/ls); do
  cd $repo
  git remote add $remote ssh://git@$server:2221/$owner/$repo.git
  git push $remote
done
```

`localhost`, Port sowie `owner` müssen natürlich an die eigene Konfiguration angepasst werden.
Der explizite Pfad für `ls` liegt darin begründet, dass in meinem System per Default ein Alias für `ls` eingerichtet war, welches neben Farbe auch einen Slash an Verzeichnisnamen anhing.

Für die Verwendung von SSH habe ich einen entsprechenden Schlüssel unter meinem User in Gitea hochgeladen.

Für die meisten Anwender ist es wahrscheinlich sinnvoll, `origin` statt `gitea` als Remote-Name zu wählen.
In meiner aktuellen Umstellungs- und Testphase bleibt aber vorläufig mein Stick noch `origin`.
