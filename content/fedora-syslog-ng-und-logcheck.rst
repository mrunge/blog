syslog-ng und logcheck bei Fedora
#################################
:date: 2011-12-30 20:11
:author: mrunge
:category: Linux
:tags: logcheck, syslog-ng
:slug: fedora-syslog-ng-und-logcheck

Ich benutze syslog-ng als Syslog-Daemon und logcheck, um mich über
Auffälligkeiten in den Logs informieren zu lassen. Bei Fedora gehören
die Logfiles normalerweise dem Benutzer root und der Gruppe root.
Logcheck soll aus Sicherheitsgründen nicht als Benutzer root laufen. Bei
Fedora gibt es einen logcheck-Nutzer, der in der Gruppe adm ist.
logcheck darf in der Standardkonfig jedoch keine Logfiles lesen.

Zum einen kann man dem beikommen, indem man in der Datei
/etc/logrotate.d/syslog den folgenden Eintrag zufügt:

::

    create 0640 root adm

Syslog-ng lässt sich damit jedoch noch nicht zur Mitarbeit überreden, es
überprüft die Rechte an den Log-Dateien und korrigiert diese notfallt.
Daher benötigt man für syslog-ng noch die folgende kleine Änderung im
Config-File:

::

    destination d_mesg {
        file("/var/log/messages"
             owner("root")
             group("adm")
             perm(0640)
            );
     };
     destination d_auth {
         file("/var/log/secure"
             owner("root")
             group("adm")
             perm(0640)
         );
     };

