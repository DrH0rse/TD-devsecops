# Rapport d'incident – TD DevSecOps/SRE

##  Contexte

- **Date** : 29 juillet 2025  
- **Heure de l'incident** : 15h42 (CET)  
- **Service impacté** : API `todo-app` – route `/error`  
- **Environnement** : Stack Docker locale orchestrée via `docker-compose`

---

##  Détection et diagnostic

L’incident a été **détecté manuellement** lors d’un test de résilience simulé via la route :

```bash
curl http://localhost:5000/error

Cette route génère volontairement une erreur HTTP 500 (division par zéro dans le code Flask).

Dans Prometheus, la montée du compteur suivant a confirmé la défaillance :

flask_http_request_total{status="500"}

Dans Grafana, la courbe du panel “Répartition des codes HTTP” a affiché une élévation des erreurs 500.

---

Cause racine

@app.route('/error')
def error():
    1 / 0  # division par zéro provoquant une exception

---

##Actions correctives et remise en route 

	•Observation en temps réel dans Grafana des erreurs HTTP (panel status → 500)
	•Vérification des métriques flask_http_request_exceptions_total
	•Relance de la route /error pour confirmer la reproductibilité
	•Fonctionnement attendu de l’observabilité (Prometheus + Grafana)
	•Aucun redémarrage nécessaire : l’application reste disponible

---

##Leçons apprises
	•Le pipeline d’observabilité est fonctionnel : les erreurs sont correctement tracées
	•Les panels Grafana permettent de visualiser l’évolution des codes HTTP en temps réel
	•Il est essentiel de définir des alertes Prometheus sur des seuils d’erreur pour anticiper des dégradations de service

---

##Actions préventives
	•Ajouter une alerte Prometheus si le taux d’erreur dépasse un certain seuil
	•Mettre en place un middleware Flask pour capturer et logguer proprement les erreurs
	•Étendre les métriques avec custom labels pour mieux filtrer par endpoints critiques
	•Introduire des tests automatisés pour éviter les routes non protégées en production