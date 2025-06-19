---
theme: dracula
title: Kubernetes für bridgeCon 2025
drawings:
  persist: false
transition: slide-left
mdc: true

layout: image
image: /cover.png
---

---
class: text-center
---

# Kubernetes – Ein grober Überblick

<!--
Zielgruppe Entwickler mit Docker-Erfahrung
-->

---

# Agenda

- Was ist Kubernetes und warum?
- Einstieg & Installation
- Deklarative Konfiguration & Tools
- Basis-Ressourcen im Cluster
  - Pod, Deployment, StatefulSet, Service
- Konfigurations- & Speicherelemente
  - ConfigMap, Secret, Volume, PVC
- Erweiterbarkeit mit CRDs & Operatoren
- GitOps mit Flux (Live-Demo)
- Fragen & Diskussion

---

# Was ist Kubernetes?

<v-clicks>

- Kubernetes ist eine **Plattform zur Orchestrierung von Containern**
- Es verwaltet **Bereitstellung, Skalierung und Ausfallsicherheit** von Anwendungen
- Ziel: *"gewünschten Zustand herstellen und erhalten"*
- Inspiriert von Google’s interner Infrastruktur (Borg)
- Kernidee: Infrastruktur als Code – deklarativ & automatisiert

</v-clicks>


---

# Warum Kubernetes?

<v-clicks>

- 🤖 **Automatisierung & Selbstheilung**
- 🚀 **Skalierung auf Knopfdruck**
- 🔄 **Zero-Downtime Deployments**
- 🌍 **Portabilität (Cloud, On-Prem, Hybrid)**
- 📝 **Deklarative Konfiguration (YAML)**
- 🛡️ **Rollen & Rechte (RBAC)**
- 💡 **Großes Ökosystem & Community**

</v-clicks>

<!--

- Automatisierung: Kubernetes erkennt, wenn Container oder Pods ausfallen und startet sie neu.  
- Skalierung: Ihr könnt Lastspitzen einfach abfangen, Kubernetes erstellt oder entfernt automatisch Instanzen.
- Zero-Downtime: Updates und Deployments laufen ohne Ausfälle, Rollbacks sind einfach möglich.
- Portabilität: Gleicher Stack auf Azure, AWS, GCP, on-prem oder lokal – keine Cloud-Bindung!
- Deklarativ: Infrastruktur als Code, alles nachvollziehbar und versionierbar.
- RBAC: Feingranulare Rechte, mehrere Teams können sicher im selben Cluster arbeiten.
- Ökosystem: Riesige Community, viele Tools und Tutorials – siehe https://github.com/tomhuang12/awesome-k8s-resources
-->

---

# Installation

<v-clicks depth=2>

- **Installation & Betrieb von Kubernetes**
  - In der Praxis ist eine eigene Installation komplex
  - Für den Start besser eine **fertige lokale Lösung** verwenden  
    K3s, RKE2, ... oder Managed Cloud Provider (AWS, Azure, GCP)
- Im **Produktivbetrieb** ist die Konfiguration aufwändig:
  - **Netzwerk-Layer** (CNI-Plugin) muss ausgewählt und konfiguriert werden (z.B. Calico, Flannel, Cilium)
  - Storage, Monitoring, Logging, RBAC, Security etc. müssen individuell angepasst werden
  - Multi node, Ressource Limits, Skalierung, High Availability (HA)
  - Kubernetes Update nicht so einfach wie `apt upgrade -y`
  - (n + 1) Administratoren notwendig
  - heute nicht

</v-clicks>

---

# Einstieg

<v-clicks depth=2>

- **Docker Desktop** (Windows/Mac)  
  Startet schnell ein lokalen Kubernetes-Cluster
- **Rancher Desktop** (Windows/Mac/Linux)  
  Alternative zu Docker Desktop, ebenfalls mit integriertem Kubernetes
- minikube
- kind

</v-clicks>

---

# Deklarative Konfiguration

<v-clicks depth=2>

- Ressourcen als YAML definieren (Manifest)
  - beschreiben Soll-Zustand
- Cluster-Zustand wird automatisch angepasst
- kann versioniert werden (git)
- Grundvoraussetzung für GitOps
- Helm Charts für
  - Templating der Manifeste
  - Paketmanger für Anwendungen

</v-clicks>

---

# Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: whoami-pod
spec:
  containers:
    - name: whoami
      image: docker.io/traefik/whoami
      ports:
        - containerPort: 80
```

- Kleinste deploybare Einheit in Kubernetes (für Container)
- Mehrere Container pro Pod möglich (z. B. Sidecars)
- Container teilen sich Netzwerk, IP & Volumes
- Unterstützt Init-Container für vorbereitende Aufgaben
- Pods sind kurzlebig
- In der Praxis werden sie fast immer durch Controller gemanaged

---

# Tools: kubectl & Lens

<v-clicks depth=2>

- **kubectl** ist das wichtigste Kubernetes-Werkzeug
  - Mit kubectl werden Ressourcen erstellt, geändert, gelöscht, inspiziert
  - Direkte Kommunikation mit dem Kubernetes API-Server
  - Beispiel: `kubectl apply -f pod.yaml`
  - Cluster-Zustand wird automatisch angepasst
  - Löschen mit `kubectl delete -f <datei.yaml>`  
    oder `kubectl delete pods my-pod-1`
  - Autocomplete  
    `kubectl completion powershell | Out-String | Invoke-Expression`
- **Lens** als grafische IDE für die Demo
  - Erlaubt visuelle Darstellung von Ressourcen und Logs
  - Einfaches Anwenden von Manifesten per Klick
  - unter $10M Umsatz free tier

</v-clicks>

---

# Demo

in Lens

**Pod**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: whoami-pod
spec:
  containers:
    - name: whoami
      image: docker.io/traefik/whoami
      ports:
        - containerPort: 80
```

---
layout: two-cols-header
---

# Deployment

::left::

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: app
          image: docker.io/traefik/whoami
          ports:
            - name: http
              containerPort: 80
```

::right::

- Verwalten stateless Anwendungen (Webserver, API, Worker)
- Sorgt für Skalierung, Updates und Selbstheilung

<!-- 
Deployments sind der Standard für stateless Anwendungen. Sie beschreiben, wie viele Pods laufen sollen („replicas“), welches Image verwendet wird, und kümmern sich um Updates und Skalierung. Bei Fehlern sorgt das Deployment dafür, dass die gewünschte Anzahl Pods immer verfügbar ist. 
-->

---
layout: two-cols-header
---

# StatefulSet

::left::

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
spec:
  serviceName: "db"
  replicas: 1
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
        - name: db
          image: postgres
          ports:
            - containerPort: 5432
          volumeMounts:
            - name: postgres
              mountPath: /data
```

::right::

```yaml
          env:
          - name: POSTGRES_PASSWORD
            value: admin
  volumeClaimTemplates:
  - metadata:
      name: postgres
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 2Gi
```

- Für zustandsbehaftete Anwendungen (z.B. Datenbanken)
- Erhält feste Namen & stabilen Speicher für jeden Pod

<!-- 
StatefulSets sind für Anwendungen gedacht, die stabilen Speicher und eine feste Identität pro Instanz brauchen – z.B. Datenbanken oder Message Queues. Jeder Pod bekommt einen stabilen Namen und ein eigenes Volume. Updates und Rollouts erfolgen hier geordnet (nacheinander).
-->

---

# Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: http
  type: LoadBalancer
```

- Macht Pods im Cluster auffindbar und erreichbar
- nutzt Label-Selector-Prinzip
- Verschiedene Typen: ClusterIP, NodePort, LoadBalancer
- `curl http://localhost/`

---

# ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-config
data:
  APP_ENV: production
  LOG_LEVEL: debug
```

- Speichert Konfiguration als Key/Value-Paare
- Trennung von Code und Konfiguration
- Kann als Umgebungsvariable oder Datei im Pod verwendet werden
- Der Value kann auch eine ganze Datei sein. (< 1MB)

<!-- 
Mit ConfigMaps werden Umgebungsvariablen, Konfigurationsdateien usw. verwaltet. Der Code bleibt unverändert, die Konfiguration kann unabhängig davon angepasst werden. Im Pod können ConfigMaps als Umgebungsvariable oder als Datei gemountet werden.
-->

---

# Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: example-secret
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQ=    # Base64-encoded!
```

- Für vertrauliche Daten (z.B. Passwörter, Tokens, Zertifikate)
- Werte sind Base64-codiert gespeichert
- Können als Umgebungsvariable oder Datei bereitgestellt werden

<!-- 
Secrets werden ähnlich wie ConfigMaps benutzt, aber für sensible Daten wie Passwörter und API-Tokens. Die Werte sind Base64-codiert (nicht verschlüsselt, aber nicht im Klartext). Secrets können Pods als Umgebungsvariable oder als Datei zur Verfügung gestellt werden.
-->

---
layout: two-cols-header
---

# Volume & PersistentVolumeClaim

::left::

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: example-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

- Volumes: Speicher für Pods (z.B. für Logs oder Uploads)
- PersistentVolumeClaim (PVC): Fordert persistenten Speicher an
- Speicher bleibt über Pod-Neustarts erhalten

::right::

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-volume
spec:
  containers:
    - name: app
      image: docker.io/traefik/whoami
      volumeMounts:
        - mountPath: "/data"
          name: my-storage
  volumes:
    - name: my-storage
      persistentVolumeClaim:
        claimName: example-pvc
```

<!-- 
Volumes sind notwendig, wenn Anwendungen im Container Daten dauerhaft speichern sollen. Mit einem PersistentVolumeClaim fordere ich Speicher vom Cluster an, z.B. für eine Datenbank oder Datei-Uploads. Im Pod wird das Volume eingebunden, sodass Container darauf zugreifen können – auch nach einem Pod-Restart bleibt der Speicher erhalten. 
-->

---
layout: two-cols-header
---

# Zusammenfassung der wichtigsten Ressourcen in Kubernetes

::left::

- **Pod:**  
  Kleinste deploybare Einheit, darin laufen ein oder mehrere Container
- **Deployment:**  
  Verwaltung und Rollout von stateless Anwendungen
- **StatefulSet:**  
  Verwaltung zustandsbehafteter Anwendungen, z.B. Datenbanken
- **Service:**  
  Stellt Netzwerkzugriff auf Pods bereit, sorgt für Load Balancing


::right::

- **ConfigMap:**  
  Konfiguration als Key/Value-Paare
- **Secret:**  
  vertrauliche ConfigMap
- **PersistentVolume** / **PersistentVolumeClaim**
  bleibende Daten
- **Volume**  
  kurzfristige Daten


---

# CustomResourceDefinitions (CRDs)

- API-Erweiterung: Ermöglichen neue, benutzerdefinierte Objekttypen (Custom Resources) in Kubernetes einzuführen
- Schema-Definition: Eine CRD legt das Daten-Schema (Namen, Spezifikations-Felder) der neuen Ressource fest (JSON Schema)
- Ohne Logik: Allein ermöglicht eine CRD nur Speicherung/Abruf dieser Daten – keine eigene Steuerungslogik oder Automatisierung

---

# Kubernetes Operatoren

- Controller mit Domain-Wissen: Ein Operator ist ein spezieller Kubernetes-Controller, der eine Custom Resource überwacht und basierend darauf Aktionen im Cluster ausführt
- Kontrollschleife: Er folgt dem Kubernetes-Control Loop-Prinzip: vergleicht den Soll-Zustand (aus der CR-Spezifikation) mit dem Ist-Zustand und korrigiert Abweichungen automatisch
- Automatisierung: Verpackt Operations-Wissen als Code, um komplexe Aufgaben (z.B. Backups, Skalierung, Updates) automatisch und standardisiert durchzuführen
- Erweiterbarkeit: Durch CRDs + Operatoren lässt sich Kubernetes um neue Funktionen erweitern, ohne den Kubernetes-Core zu verändern

---

# Beispiele für populäre & Operatoren

- **Sealed Secrets**  
  → Operator verschlüsselt/dechiffriert Secrets serverseitig

- **Gateway API**  
  → Ermöglicht flexibles Routing auf Cluster‑Ebene

- **Prometheus Operator**  
  → Automatisches Setup von Monitoring & Alerting

- **External Secrets Operator**  
  → Synchronisiert Secrets aus AWS, Vault, Azure, GCP etc.

- **Redis/Kafka/TiDB Operators**  
  → CRDs für Datenbank‑Cluster
  → Automatisieren Provisioning, Skalierung, Backup, Recovery

- **Keycloak Operator**    
  → Automatisiert Provisionierung von Keycloak, Realms, Clients, DB, TLS etc. :contentReference[oaicite:4]{index=4}

- **Flux**

---

# Flux – GitOps

- Flux besteht aus mehreren spezialisierten Controllern:
  - `source-controller`: verwaltet Git-Repositories & Helm-Repos
  - `kustomize-controller`: kümmert sich um Kustomization-Deployments (und YAML Manifeste)
  - `helm-controller`: für Helm Releases
- Diese arbeiten auf Basis von **Custom Resources (CRDs)** wie:
  - `GitRepository`, `Kustomization`, `HelmRelease` etc.
- Jeder Controller beobachtet „seine“ Ressourcen und gleicht Soll- mit Ist-Zustand ab

<!-- note:
Flux ist nicht ein einzelner Operator, sondern eine modulare Sammlung spezialisierter Controller.  
Jeder Controller beobachtet bestimmte CRDs – zum Beispiel kümmert sich der `source-controller` um `GitRepository`-Ressourcen.  
Diese beschreiben, welches Git-Repo, welcher Branch oder welches Helm-Chart die Quelle der Wahrheit für Konfigurationen ist.  

Der `kustomize-controller` übernimmt dann die eigentliche Anwendung dieser Konfigurationen im Cluster – er liest Kustomization-Ressourcen und führt sie aus.  
Optional kann auch der `helm-controller` Helm-Charts installieren.  

Das Prinzip: Jede Ressource in Git wird zu einer CRD im Cluster, die von genau einem passenden Controller ausgewertet wird.  
Änderungen im Git führen – indirekt, aber automatisch – zu synchronisierten Deployments.
-->

---

# Live-Demo mit Flux

- Flux schon installiert (namespace: `flux-system`)
- Anwendung schon ausgerollt (namespace: `flux-demo`)
- https://github.com/mcnilz/flux-demo
- Änderungen im Git führen zu einem neuen Deployment

<!-- note:
Ich habe Flux bereits eingerichtet.  
Jetzt zeige ich euch, wie eine einfache App – ein nginx – über Git deployed wird.  
Der Clou: Der HTML-Inhalt liegt nicht im Image, sondern kommt aus einer ConfigMap.  
Wir werden sehen, wie Flux Änderungen erkennt und automatisch neu deployed.
-->

---
layout: center
---

<div style="text-align: center">

# Ende

<img style="display: inline-block" src="/mic-drop.gif" />

https://mcnilz.github.io/kubernetes-bridgecon-2025/

<p style="font-size: 0.8em; color: gray;">
Diese Präsentation wurde mit Unterstützung von <strong>ChatGPT</strong> erstellt ✨
</p>

</div>

---

# für Besserwisser Nachfragen

---

# Wie funktioniert das Anwenden von Ressourcen in Kubernetes?

- Ressourcen werden als YAML (oder JSON) an die Kubernetes API gesendet (z.B. via `kubectl apply`)
- Die Kubernetes API speichert alle Ressourcen im **etcd**-Cluster (Zustandsdatenbank)
- **Controller** überwachen kontinuierlich den „Soll-Zustand“ (aus YAML) und den „Ist-Zustand“ (laufende Pods etc.)

<!--
Kubernetes ist komplett API-gesteuert.  
Alles – Pods, Deployments, Services usw. – werden als Ressourcen an die API übermittelt.  
Die API speichert den gewünschten Zustand (Soll-Zustand) zentral ab.  
Controller (z.B. Deployment Controller, ReplicaSet Controller) beobachten ständig, ob der Ist-Zustand im Cluster mit dem Soll-Zustand übereinstimmt.  
Wenn etwas abweicht (z.B. ein Pod ist abgestürzt), steuern die Controller gegen (z.B. neuer Pod wird gestartet).
-->

---

# Kubernetes Architektur

```mermaid
flowchart LR
    User([kubectl / User])
    API[API Server]
    etcd[(etcd)]
    Scheduler[Scheduler]
    Controller[Controller]
    Node1[Kubelet auf Node 1]
    Node2[Kubelet auf Node 2]

    User --"apply YAML"--> API
    API --"speichert Zustand"--> etcd
    API --"informiert"--> Controller
    Controller --"verwaltet Ressourcen"--> API
    API --"stellt bereit"--> Scheduler
    Scheduler --"entscheidet Zuweisung"--> API
    Node1 --"fragt Pod-Status ab"--> API
    Node2 --"fragt Pod-Status ab"--> API

```

- API-Server: Drehscheibe für alle Komponenten
- Scheduler: Entscheidet, welcher Pod auf welchem Node läuft
- Kubelet: Fragt den API Server ab und startet die Container

<!-- 
Wichtig: Der API Server kommuniziert nicht direkt mit den Nodes!
Die Kubelets auf den Nodes „pullen“ regelmäßig den API Server, um neue Aufgaben (Pods) zu bekommen.
Der Scheduler ist derjenige, der Pods bestimmten Nodes zuweist.
Controller sorgen dafür, dass alles im Soll-Zustand bleibt (z.B. ReplicaSet Controller sorgt für richtige Anzahl Pods).
-->