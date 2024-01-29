# Scalabilité & exploitation

Paramètres permettant d'assurer la stabilité du système :
- on utilise des charts en dépendance versionnés
- nombre de replicas
- livenessProbe & readinessProbe pour s'assurer que les conteneurs sont up & ready 
- limites de ressources (request & limit)
- drop de capabilities via securityContext (le user du conteneur est root)
- gestion des secrets (external-secrets, gestionnaire de secrets externe tel que vault)
- restartPolicy
- service account spécifique par service (moindre privilège)

En ce qui concerne la scalabilité : 
- HPA sur les données des pods (CPU / RAM / on peut déterminer selon test de charge) + request 
- Set min/max replicas

Tout en maitrisant les coûts :
- Karpenter (spots ?)
- Maitrise des resources des pods

502 - Bad Gateway - Rolling update:
- les nouveaux pods ont une readinessProbe mal configurée 
- le lb redirige trop tôt vers les nouveaux pods (readinessGates; preStop hook pour graceful termination & connection draining)
- trop de pods updated en même temps (ne tient pas la charge)
- la nouvelle version renvoie des 502 sur une ou un ensemble de route (fail de deps)

# Monitoring

Métriques / Logs / APM (Stack Prometheus / Datadog / ELK, etc.)
https://promcat.io/ (I'd recommend to start with basic metrics & alerts from it)

En termes de système, daemonset de conteneurs de collecte de logs (formatées en stdout/stderr et ship vers un ou plusieurs serveurs)
On peut gérer la collecte de logs via des labels (datadog / prom)

API
- Requests and errors
- Errors by code
- Latency (p90, p95, p99)
(via trace, OpenTelemetry)

Apdex

MongoDB
- metrics 
- alerts (disk space, open cursors, CPU, RAM, etc.)

Redis
- metrics
- alerts (disk space, open cursors, CPU, RAM, etc.)

LB
- métriques
  - activeconnection
  - HTTPCODE_3XX_Count
  ...
  - HTTPCODE_5XX_Count (500 / 502 / 503 / 504)
  - target metrics (sticky / non sticky, HTTP target; healthy host count, target_response_time.p99, 95, 90, 70, 50)

SLO & alerts 

# CI/CD

ArgoCD pour la partie CD ; grosse possibilité de rollout (blue/green, canary) & automated rollback with metrics
Un argo par cluster
Pour la partie CI, application node, gestion des deps, dependabot pour bump de version, packaging de libs transverse éventuellement, mirroring interne de npm?, docker  build, push dans une registry, promotion d'images entre envs si le build le permet, images minimalistes, SonarQube, test unitaires, tests E2E