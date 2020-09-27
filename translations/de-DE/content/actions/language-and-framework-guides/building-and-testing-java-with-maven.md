---
title: Java bauen und testen mit Maven
intro: 'Du kannst einen Workflow für kontinuierliche Integration (CI) in GitHub-Aktionen erstellen, um Dein Java-Projekt mit Maven zu bauen und zu testen.'
product: '{{ site.data.reusables.gated-features.actions }}'
versions:
  free-pro-team: '*'
  enterprise-server: '>=2.22'
---

{{ site.data.reusables.actions.enterprise-beta }}
{{ site.data.reusables.actions.enterprise-github-hosted-runners }}

### Einführung

Dieser Leitfaden zeigt Dir, wie Du einen Workflow erstellen kannst, der eine kontinuierliche Integration (CI) für Dein Java-Projekt mit Hilfe des Software-Projektmanagement-Tools Maven durchführt. Der Workflow, den Du erstellst, zeigt Dir, wenn Commits zu einem Pull-Request zu Build- oder Testfehlern für deinen Standard-Zweig führen. Dieser Ansatz kann dazu beitragen, dass Dein Code immer brauchbar ist. Du kannst Deinen CI-Workflow so erweitern, dass er Dateien im Cache zwischenspeichert und Artefakte von einem Workflow-Lauf hochlädt.

{{ site.data.variables.product.prodname_dotcom }}-gehostete Runnner haben einen Tools-Cache mit vorinstallierter Software, einschließlich Java Development Kits (JDKs) und Maven. Eine Liste der Software und der vorinstallierten Versionen für JDK und Maven findest Du unter „[Software auf {{ site.data.variables.product.prodname_dotcom }}-gehosteten Runnern](/actions/automating-your-workflow-with-github-actions/software-installed-on-github-hosted-runners)“.

### Vorrausetzungen

Du solltest mit YAML und der Syntax für {{ site.data.variables.product.prodname_actions }} vertraut sein. Weitere Informationen findest Du unter:
- „[Workflow-Syntax für {{ site.data.variables.product.prodname_actions }}](/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions)“
- "[Einen Workflow konfigurieren](/actions/automating-your-workflow-with-github-actions/configuring-a-workflow)"

Du solltest ein grundlegendes Verständnis von Java und dem Framework Maven haben. Weitere Informationen findest Du in der [Anleitung für erste Schritte mit Maven](http://maven.apache.org/guides/getting-started/index.html) in der Maven-Dokumentation.

{{ site.data.reusables.actions.enterprise-setup-prereq }}

### Einstieg mit einer Maven-Workflow-Vorlage

{{ site.data.variables.product.prodname_dotcom }} bietet eine Maven-Workflow-Vorlage, die für die meisten Maven-basierten Java-Projekte funktionieren wird. Weitere Informationen findest Du im [Workflow-Template für Maven](https://github.com/actions/starter-workflows/blob/master/ci/maven.yml).

Um schnell loszulegen, kannst Du beim Erstellen eines neuen Workflows die vorkonfigurierte Maven-Vorlage auswählen. Weitere Informationen findest Du unter „[Mit vorkonfigurierten Workflow-Vorlagen beginnen](/actions/automating-your-workflow-with-github-actions/starting-with-preconfigured-workflow-templates)“.

Du kannst auch manuell diesen Workflow hinzufügen, indem Du eine neue Datei im Verzeichnis `.github/workflows` Deines Reporitorys erstellst.

{% raw %}
```yaml
name: Java CI

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - name: Set up JDK 1.8
        uses: actions/setup-java@v1
        with:
          java-version: 1.8
      - name: Build with Maven
        run: mvn -B package --file pom.xml
```
{% endraw %}

Dieser Workflow führt die folgenden Schritte aus:

1. Der Schritt `checkout` lädt eine Kopie Deines Repositorys auf den Runner herunter.
2. Der Schritt `setup-java` konfiguriert das Java 1.8 JDK.
3. Der Schritt "Build with Maven" führt das Maven-„Target“ (Ziel) `package` im nicht-interaktiven Modus aus, um sicherzustellen, dass der Code gebaut, Tests bestanden und ein Paket erstellt werden kann.

Die Standard-Workflow-Vorlagen sind ausgezeichnete Ausgangspunkte beim Erstellen des Build- und Testworkflows, und Du kannst die Vorlage an die Anforderungen Deines Projekts anpassen.

{{ site.data.reusables.github-actions.example-github-runner }}

{{ site.data.reusables.github-actions.java-jvm-architecture }}

### Deinen Code bauen und testen

Du kannst die gleichen Befehle verwenden, die Du auch lokal verwendest, um Deinen Code zu erstellen und zu testen.

Der Starter-Workflow führt standardmäßig das „target“ (Ziel) `package` aus. In der Standard-Maven-Konfiguration lädt dieser Befehl Abhängigkeiten herunter, baut Klassen, führt Tests durch und paketiert Klassen in ihr verteilbares Format, zum Beispiel eine JAR-Datei.

Wenn Du zum Bauen Deines Projekts andere Befehle verwenden oder ein anderes Ziel auszuführen möchtest, kannst Du dies angeben. Vielleicht möchtest Du beispielsweise das Ziel `verify` ausführen, das in Deiner Datei _pom-ci.xml_ konfiguriert ist.

{% raw %}
```yaml
steps:
  - uses: actions/checkout@v2
  - uses: actions/setup-java@v1
    with:
      java-version: 1.8
  - name: Run the Maven verify phase
    run: mvn -B verify --file pom-ci.xml
```
{% endraw %}

### Abhängigkeiten „cachen“ (zwischenspeichern)

Du kannst Deine Abhängigkeiten zwischenspeichern, um die Workflow-Ausführungen zu beschleunigen. Nach einem erfolgreichen Lauf wird Dein lokales Maven-Repository in der Aktions-Infrastruktur auf GitHub gespeichert. Bei zukünftigen Workflow-Ausführungen wird der Cache wiederhergestellt, so dass Abhängigkeiten nicht aus entfernten Maven-Repositories heruntergeladen werden müssen. Weitere Informationen findest Du unter „[Caching-Abhängigkeiten zur Beschleunigung von Workflows](/actions/automating-your-workflow-with-github-actions/caching-dependencies-to-speed-up-workflows)“ und der [Aktion `cache`](https://github.com/marketplace/actions/cache).

{% raw %}
```yaml
steps:
  - uses: actions/checkout@v2
  - name: Set up JDK 1.8
    uses: actions/setup-java@v1
    with:
      java-version: 1.8
  - name: Cache Maven packages
    uses: actions/cache@v2
    with:
      path: ~/.m2
      key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
      restore-keys: ${{ runner.os }}-m2
  - name: Build with Maven
    run: mvn -B package --file pom.xml
```
{% endraw %}

Dieser Workflow speichert den Inhalt Deines lokalen Maven-Repositiorys im Verzeichnis `.m2` des Home-Verzeichnisses auf dem Runner. Der Cache-Schlüssel wird der gehashte Inhalt von _pom.xml_sein, so dass Änderungen an _pom.xml_ den Cache ungültig machen.

### Workflow-Daten als Artefakte paketieren

Nachdem sowohl Build erfolgreich war und Deine Tests bestanden hat, wirst Du die resultierenden Java-Pakete als Build-Artefakt hochladen wollen. Dies speichert die gebauten Pakete als Teil der Workflow-Ausführung und ermöglicht Dir, sie herunterzuladen. Artefakte können Dir helfen, Pull-Requests in Deiner lokalen Umgebung zu testen und zu debuggen, bevor sie zusammengeführt werden („merge“). Weitere Informationen findest Du unter „[Workflow-Daten mittels Artefakten persistieren](/actions/automating-your-workflow-with-github-actions/persisting-workflow-data-using-artifacts)“.

Maven erstellt normalerweise Ausgabedateien wie JARs, EARs oder WARs im Verzeichnis `target`. Um diese als Artefakte hochzuladen, kannst du sie in ein neues Verzeichnis kopieren, welches Artefakte zum Hochladen enthält. Zum Beispiel kannst Du ein Verzeichnis namens `staging` erstellen. Dann kannst Du den Inhalt dieses Verzeichnisses mit der Aktion `upload-artifact` hochladen.

{% raw %}
```yaml
steps:
  - uses: actions/checkout@v2
  - uses: actions/setup-java@v1
  - run: mvn -B package --file pom.xml
  - run: mkdir staging && cp target/*.jar staging
  - uses: actions/upload-artifact@v2
    with:
      name: Package
      path: staging
```
{% endraw %}