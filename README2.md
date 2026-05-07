# TinyInsta Benchmark Report

## Web Application

Base URL:
https://rouguycloudproject.ew.r.appspot.com

API endpoint tested:
https://rouguycloudproject.ew.r.appspot.com/api/timeline

## Dataset Initialization

- Pour peupler la base avec 1000 users, 50 posts par user et 20 followees, j'ai utilisé la seed(). La requête curl:
- curl -i -X POST \
 H "X-Seed-Token: change-me-seed-token" \
"https://rouguycloudproject.ew.r.appspot.com/admin/seed?users=1000&posts=50000&follows_min=20&follows_max=40&prefix=load"

## Concurrency Tests


- Pour remplir le fichier csv de la concurrence, je n'ai pas utilisé un script python mais une requête locust que je variais pour chaque run et chaque nombre de users concurents:
"locust -f locustfile.py \
 --host=https://rouguycloudproject.ew.r.appspot.com \
 --headless \
 -u 1 \
 -r 1 \
 -t 60s \
 --csv=out/conc_1_run1
"

Test de la concurence avec variation du nombre d'utilisateurs:

Tests réalisés avec :
- 1 utilisateur
- 10 utilisateurs
- 20 utilisateurs
- 50 utilisateurs
- 100 utilisateurs
- 1000 utilisateurs

<img width="952" height="577" alt="image" src="https://github.com/user-attachments/assets/fe53c740-86ce-498d-b6ec-6099a67622ac"/>

## Results Interpretation

Les tests de charge montrent que l’application supporte correctement les accès concurrents avec un faible taux de latence moyen et aucune erreur observée.
Les performances restent stables jusqu’à une charge modérée, mais une augmentation importante des percentiles élevés indique une dégradation progressive sous forte concurrence.
Le système est robuste mais présente des limites de scalabilité sur les requêtes extrêmes, probablement liées aux mécanismes de scaling ou aux ressources backend.
Aussi, comme le système est distribué, utilise du hash partionning et de l'auto scaling, google gère bien la répartiton des instances en fonction du besoin.

## Fanout Tests


Pareil que pour la concurence, j'ai utilisé les mêmes requêtes locust et curl pour peupler et pour tester le temps moyen.

Fanout testé avec :
- 20 followees
- 40 followees
- 60 followees

<img width="861" height="520" alt="image" src="https://github.com/user-attachments/assets/174d0b0a-8463-4840-a397-45bf66b8b591" />

## Results Interpretation

Alors que TinyInsta scale très bien sur la charge grâce à l'auto-scaling, il scale moins bien sur la densité du graphe social (fan-out) car la latence des requêtes augmente proportionnellement avec nombre d'abonnements.
Contrairement aux tests précédents, le temps moyen ou AVG_TIME augmente de façon significative à mesure que PARAM augmente. Parce que la timeline est générée par une requête GQL : SELECT * FROM Post WHERE author IN @authors ORDER BY created DESC, Datastore doit fusionner les index de chaque auteur suivi et plus il y a d'auteurs (20 -> 60), plus le "merge-sort" côté serveur est coûteux en temps processeur.

## Conclusion

Les expérimentations montrent que TinyInsta bénéficie efficacement de l’auto-scaling de Google App Engine pour absorber la montée en charge concurrente.
Cependant, les performances deviennent plus sensibles lorsque la densité du graphe social augmente. Le coût des requêtes de timeline croît avec le nombre d’auteurs suivis, ce qui impacte directement la latence moyenne et les percentiles élevés.
Le système reste robuste sans erreurs observées pendant les tests, mais les scénarios à très fort fanout montrent les limites de scalabilité liées au modèle de requête Datastore.

