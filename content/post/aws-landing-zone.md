---
title: "Für euch getestet: Multi Account Setups mit AWS Landing Zone"
author: "Soenke Ruempler"
slug: "fuer-euch-getestet-multi-accounts-setups-mit-aws-landing-zone"
date: 2018-08-03
---

**Update 2018-11-27: Landing Zone 2.0 wurde released und wir haben den Artikel entsprechend angepasst.**

> Was sind AWS Best Practices? Wie kann ich meine Workloads absichern? Wie kann ich Teams größtmögliche Autonomie geben, ohne  dass die Sicherheit/Einheitlichkeit leidet? Welches Grund-Design sollte ich in AWS verwenden?  Wie verwalte ich Benutzer und Zugriffsrechte? Wie geht AWS Multi Account?

Dies sind viele Fragen, die wir regelmäßig hören, gerade von Kunden, die neu auf AWS unterwegs sind, und sich im Zoo der ganzen Konzepte und Services zurechtfinden müssen.

Hier und da existieren Blog Posts zu Best Practices, auch von AWS selbst - manche aktuell, manche outdated - aber wäre es nicht toll, wenn es da etwas Automatisiertes gäbe, so dass man einen Quickstart hat und direkt mit der eigentlichen Arbeit starten kann?

Wir sehen häufig, dass aufgrund verschiedener Umstände das AWS Grund-Setup dann in der AWS Web Console "geklickt" wird, d.h. manuell erstellt und damit wenig nachvollziehbar oder reproduzierbar ist. Weiterhin basiert dann das Einarbeiten von Good/Best Practices häufig auf zufälligen Events, z. B. hat ein\*e Mitarbeiter\*in einen Blog Post gelesen, verlinkt den Artikel in den Dev-Chat nach dem Motto "man müsste mal". Gemacht wird es dann oft nicht, weil "keine Zeit" oder "nicht im Scope" oder der initiale Aufwand zu groß sei. Oder es wird gemacht, aber dann wiederum "geklickt", womit das Wissen dann häufig im Kopf der "Klicker\*innen" verbleibt und somit oft versandet.

Bisher gab es für die Setups keine (uns bekannte) Lösung, doch mit AWS Landing Zone ändert sich dies: Landing Zone eine "AWS Solution", welche aktuelle Best Practices kodifiziert und automatisiert ausrollt. 

## Was verspricht Landing Zone?

 - Automatisiertes AWS Multi Account Setup
 - Grundlegende Sicherheitsrichtlinien
 - Kodifizierte Best Practices (inkl. Updates direkt von AWS) - z.B. automatisiertes CloudTrail Setup, GuardDuty und VPC/Netzwerk-Design
 - DevOps Best Practices: Infrastructure-as-Code durch kodifizierte Templates und Continuous Delivery, wodurch auch eigene Erweiterungen global ausgerollt werden können.
 - hohe Anpassbarkeit durch Templates
 - Modularität
 - Single Sign-On und zentrale Verwaltung von Zugriffsrechten (optional)

## Der erste Eindruck

Während [einige](https://www.contino.io/insights/3-ways-the-new-aws-landing-zone-as-code-accelerates-enterprise-cloud-adoption) [Blog Posts](http://up2v.nl/2018/06/20/aws-introducing-aws-landing-zone-solution/) nur nochmal mehr oder weniger die News von AWS kopieren, haben wir Landing Zone direkt für euch ausprobiert.

### Die Installation

Die Installation erfolgt über ein CloudFormation Template, das sogenannte "Initiation Template". Hier wählt man einige Grundeinstellungen aus.

Die Initialisierung schreibt ein Config-Template in ein S3 Bucket. Dieses dient als Quelle für eine CodePipeline, die ebenfalls automatisch über das Initiation Template angelegt wird.

Diese Config kann später angepasst werden und ist die "Single Source of Truth" für eure Landing Zone. Bei jeder Änderung der Config läuft die CodePipeline los und wendet die Änderungen auf die Infrastruktur an. Neben dem Manifest sind auch CloudFormation Templates, z. B. für VPCs, Teil der Konfiguration (und können somit auch angepasst oder erweitert werden, wenn gewünscht). 

Für den ersten Test empfehlen wir übrigens, den Parameter `BuildLandingZone` auf `false` zu stellen, so dass nicht direkt die Pipeline "losrennt". So könnt ihr erst einmal die generierte Config inspizieren und ggf. noch anpassen, bevor es losgeht. Weiterhin solltet ihr `LockStackSetsExecutionRole` ggf. auch erst auf `false` setzen, denn sonst geht euch ggf. der direkte Zugriff auf die Sub-Accounts verloren. Später ist es allerdings empfehlenswert, den Schalter auf `true` zu setzen, da hierdurch der Administratoren-Rollen-Zugriff in Sub-Accounts auf die Landing Zone Ressourcen beschränkt wird.  

Das Grundsetup legt folgende Accounts an: 

 - Master: Hier liegt die AWS Organization und auch die CodePipeline, welche die Landing Zone ausrollt. Weiterhin liegen hier SSO, und ein Service Catalog für die sogenannte "Account Vending Machine" (AVM), welche das Anlegen von neuen AWS Accounts automatisiert.
 - Security: Hier liegen Rollen, die es erlauben, in andere Accounts zu wechseln sowie Notifications über Security Incidents
 - Shared Services: Active Directory und andere Dienste, die von allen Accounts konsumiert werden, sollen hier Platz finden.
 - Logging: Hier landen zentral die Logs, z. B. CloudTrail Audit Logs.

![AWS Landing Zone Account Struktur](https://d1.awsstatic.com/aws-answers/answers-images/landing-zone-implementation-architecture.6bfa23d88aef1ce97035d0333f476898739697b9.png)

([Quelle: https://aws.amazon.com/answers/aws-landing-zone/](https://aws.amazon.com/answers/aws-landing-zone/))

### Account Baseline

In alle Accounts wird eine sogenannte Baseline provisioniert, diese enthält in der Default-Konfiguration:

 - CloudTrail Setup (Audit Logs)
 - AWS GuardDuty fuer automatisierte Threat Detection.
 - AWS Config und ein grundlegendes Ruleset ("Governance"), z. B. um einen Alert zu senden, wenn CloudTrail deaktiviert wurde
 - IAM Passwort Policy für IAM User
 - Cross Account Access vom Security Account
 - optional eine VPC laut Spezifikation
 - Notifications und Alarme, z. B. bei Stammbenutzer Logins. 

### Einen neuen AWS Account anlegen

Einen neuen AWS Account anlegen könnt ihr nach erfolgreicher Einrichtung der AWS Landing Zone über den AWS Service Catalog, wo ein Produkt `AWS-Landing-Zone-Account-Vending-Machine` angelegt wurde. Über dieses könnt ihr jetzt neue Accounts provisionieren. Die Baseline wird vollautomatisiert mit angelegt.

{{< figure src="/img/aws-landing-zone/new-account.png" title="Auswahl des Produkts im Service Catalog">}}

Nach Auswahl des Produktes können E-Mail-Adresse des Stammbenutzers für den neuen Account sowie ein Name und die Organizational Unit ausgewählt werden. Zusätzlich könnt ihr einstellen, ob und wie eine VPC konfiguriert werden soll.

{{< figure src="/img/aws-landing-zone/new-account-2.png" title="Einstellen der Parameter für den neuen Account">}}

## Was uns gefällt

 - Kodifizierung: Die gesamte Lösung ist durch Code dokumentiert und es gibt keine manuellen Schritte (Ausnahme: Das Umbiegen der CodePipeline auf etwas anderes als S3 als Quelle ist gerade ein geklickter Schritt)
 - Erweiterbarkeit: Landing Zone ist eine fundierte Basis für das Ausrollen z. B. von eigenen CloudFormation Stacks.
 - Best Practises direkt von AWS: Hier hat AWS Erfahrungen gesammelt und kodifiziert.
 - Landing Zone wird laufend von AWS gepflegt und erweitert: So kam kurz nach unserem Test eine neue Version heraus, die 2 Bugs gefixt hat, auf die wir auch gestoßen waren. Allerdings gibt es derzeit noch keine Notifications oder Changelogs.
 - Anwendung auf bestehende Setups: Grundlegend ist es möglich, bestehende Multi-Account Setups auf AWS Landing Zone zu migrieren und somit die Vorteile auch für Bestands Setups zu genießen.
 - Idempotenz: Wir hatten während des Tests mehrere Fehler, wo es nicht weiterging, z. B. in der Pipeline, aber nach Fixen des Codes konnten wir die Pipeline neu anstoßen und es ging weiter. Wir kamen also nie in eine Sackgasse. Dies ist wohl auch dem konsequenten Einsatz von Lambda und Step Functions zu verdanken, die eine gewisse Idempotenz erzwingen.

## Was uns (noch) nicht so gut gefällt

Unter anderen sind uns bei den ersten Tests folgende Dinge aufgefallen:

 - Komplexität durch viele eingesetzte Services: So ist es mitunter etwas mühsam, zu verstehen, was passiert. Ein beispielhafter Aufruf: Service Catalog triggert CloudFormation, welches Custom Resources hat, die Lambda triggern, was eine Step Functions State Machine triggert, welche u.a. CloudFormation StackSets aufrufen. Allerdings ist durch den konsequenten Einsatz von Serverless Komponenten auch ein geringer Wartungsaufwand gewährleistet.
 - Noch nicht so richtig veröffentlicht: Die Solution ist zwar öffentlich abrufbar, aber "versteckt". Sie kann aber trotzdem benutzt werden und unsere ersten Tests haben eine grundlegende "Production readiness" ergeben. Weiterhin unterliegt sie der [Amazon Software License](https://aws.amazon.com/asl/), welche Verwendung und Veränderung im AWS Kontext erlaubt.
 - Die AWS SSO Lösung ist gerade auf die `us-east-1` beschränkt und kann von Haus aus kein Multi-Factor-Auth.

## Und die Kosten?

Landing Zone richtet standardmässig ein paar Ressourcen ein, die Geld kosten. Die größten Kostenverursacher sind AWS Config Rules und GuardDuty je AWS Account und Region. Laut AWS kommen hier ca. $200 pro Monat zusammen.

## Wie könnt ihr es selbst testen?

Ihr könnt AWS Landing Zone selbst testen. Wir empfehlen, erst in einem frisch angelegten Account mit einer neuen Organisation zu testen, damit ihr in der "Sandkiste" Erfahrungen sammeln könnt. Landing Zone selbst kommt als CloudFormation Template und ist somit per "One-Click" installierbar:

[![Launch Stack](https://raw.githubusercontent.com/s0enke/cloudformation-templates/master/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=aws-landing-zone-initiation&templateURL=https://s3.amazonaws.com/solutions-reference/aws-landing-zone/latest/aws-landing-zone-initiation.template)

(ggf. müsst ihr noch die Region in der URL ändern, wenn ihr die Landing Zone Foundation nicht in `us-east-1` provisionieren wollt.)

Hier sind die Dokus von AWS:

 - [AWS Landing Zone Developer Guide](https://s3.amazonaws.com/solutions-reference/aws-landing-zone/latest/aws-landing-zone-developer-guide.pdf)
 - [AWS Landing Zone Implementation Guide](https://s3.amazonaws.com/solutions-reference/aws-landing-zone/latest/aws-landing-zone-implementation-guide.pdf)
 - [AWS Landing Zone  User Guide](https://s3.amazonaws.com/solutions-reference/aws-landing-zone/latest/aws-landing-zone-user-guide.pdf)

superluminar ist AWS Consulting Partner und unterstützt euch gerne, euer neues oder bestehendes Multi-Account Setup mit AWS Landing Zone zu automatisieren oder zu optimieren.

### Zusammenfassung und Ausblick

Mit Landing Zone stellt AWS uns eine solide Grundlage für sichere und automatisierte Multi Account Setups bereit. Weiterhin macht sich die Lösung durch Erweiterbarkeit, konsequenten Einsatz von Infrastructure as Code, Continuous Deployment sowie Updates von AWS positiv bemerkbar.

In folgenden Beiträgen wollen wir unter anderem weitere Aspekte erörtern:

 - [Wie kann ich Landing Zone erweitern, zum Beispiel eigene CloudFormation Stacks global über alle Accounts ausrollen und aktuell halten?]({{< ref "aws-landing-zone-extending-small.md" >}})
 - Wie bekommt ihr ein bestehendes AWS Multi Account Setup in die Landing Zone? Was ist zu beachten?
 - Wie könnt ihr die Baseline anpassen, um z.B. ohne Active Directory / SSO zu starten?
 - Wie updatet ihr Landing Zone bei neuen Releases? Was passiert bei einem Update?
 - Wie kann ich unterschiedliche Account-Settings für unterschiedliche Umgebungen (z. B. dev/prod) oder Teams/OUs erreichen?
 - Wie funktioniert die eingebaute SSO Lösung? Was ist zu beachten?
