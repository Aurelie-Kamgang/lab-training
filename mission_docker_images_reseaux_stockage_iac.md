# Mission Docker — Déployer une mini-application complète

## Contexte

Vous rejoignez une petite équipe DevOps qui doit livrer rapidement une application web interne.

L'équipe de développement vous fournit un petit site statique. Votre mission consiste à le **conteneuriser**, à le rendre **persistant**, à l'isoler dans un **réseau Docker dédié**, puis à automatiser son déploiement avec **Docker Compose**.

L'objectif n'est pas seulement de faire fonctionner l'application, mais de montrer que vous maîtrisez les notions vues jusqu'ici :

- installation et vérification de Docker ;
- images Docker ;
- Dockerfile ;
- conteneurs ;
- ports ;
- réseaux Docker ;
- volumes et bind mounts ;
- Docker Compose ;
- bonnes pratiques de base.

---

# Objectif final

À la fin de la mission, vous devez pouvoir lancer toute l'infrastructure avec :

```bash
docker compose up -d
```

Puis accéder au site depuis votre navigateur.

L'architecture finale devra ressembler à ceci :

```text
Navigateur
    |
    | port 8080
    v
+------------------+
|  conteneur web   |
|      nginx       |
+------------------+
        |
        | réseau Docker : app_network
        |
+------------------+
| conteneur tools  |
|     alpine       |
+------------------+

Données persistantes :
- un volume Docker pour les logs
- un bind mount pour le contenu du site
```

---

# Partie 1 — Vérifier votre environnement Docker

Avant de commencer, vérifiez que Docker fonctionne correctement.

Vous devez être capable d'afficher :

```bash
docker --version
docker info
docker ps
```

### Question

Quelle est la différence entre :

```bash
docker ps
```

et :

```bash
docker ps -a
```

Notez votre réponse dans un fichier nommé :

```text
REPONSES.md
```

---

# Partie 2 — Préparer le projet

Créez l'arborescence suivante :

```text
docker-mission/
│
├── web/
│   ├── index.html
│   └── Dockerfile
│
├── logs/
│
├── compose.yaml
│
└── REPONSES.md
```

Dans `index.html`, créez une page simple contenant au minimum :

```html
<h1>Mission Docker réussie</h1>
<p>Déployé avec Docker par votre prénom.</p>
```

Vous pouvez personnaliser davantage la page.

---

# Partie 3 — Construire votre propre image Docker

Dans le dossier `web`, créez un `Dockerfile`.

Contraintes :

- l'image de base doit être `nginx:alpine`;
- votre fichier `index.html` doit être copié dans le bon répertoire Nginx ;
- le port utilisé par Nginx doit être indiqué dans le Dockerfile.

Construisez ensuite votre image.

Le nom de l'image doit être :

```text
mission-web:v1
```

Vérifiez qu'elle existe avec :

```bash
docker images
```

### Question

Expliquez avec vos propres mots la différence entre :

- une image Docker ;
- un conteneur Docker.

Ajoutez la réponse dans `REPONSES.md`.

---

# Partie 4 — Tester l'image manuellement

Avant d'utiliser Docker Compose, testez votre image avec `docker run`.

Contraintes :

- nom du conteneur : `mission-web`;
- port du poste : `8080`;
- port du conteneur : `80`.

Vous devez pouvoir ouvrir :

```text
http://localhost:8080
```

et voir votre page.

### Vérifications à effectuer

```bash
docker ps
docker logs mission-web
docker inspect mission-web
```

### Question

Que signifie cette syntaxe ?

```text
8080:80
```

Ajoutez votre réponse dans `REPONSES.md`.

---

# Partie 5 — Créer un réseau Docker dédié

Créez un réseau de type `bridge` nommé :

```text
app_network
```

Vérifiez sa création.

Créez ensuite un deuxième conteneur basé sur l'image :

```text
alpine
```

Nom du conteneur :

```text
tools
```

Les deux conteneurs doivent appartenir au réseau :

```text
app_network
```

Depuis `tools`, vérifiez que vous pouvez joindre le conteneur web **par son nom**.

Exemple attendu :

```bash
ping mission-web
```

ou avec un outil HTTP disponible dans le conteneur.

### Question importante

Pourquoi est-il préférable de communiquer avec un conteneur par son **nom** plutôt que par son adresse IP ?

Répondez dans `REPONSES.md`.

---

# Partie 6 — Tester la persistance des données

Nous voulons maintenant conserver des données même si un conteneur est supprimé.

Créez un volume Docker nommé :

```text
web_logs
```

Listez vos volumes :

```bash
docker volume ls
```

Montez ce volume dans un conteneur temporaire.

Créez à l'intérieur un fichier :

```text
test-volume.txt
```

Supprimez ensuite le conteneur.

Créez un nouveau conteneur utilisant le même volume.

### Résultat attendu

Le fichier :

```text
test-volume.txt
```

doit toujours être présent.

### Question

Pourquoi les données ont-elles survécu à la suppression du premier conteneur ?

Répondez dans `REPONSES.md`.

---

# Partie 7 — Utiliser un bind mount

Cette fois, le fichier `index.html` présent sur votre machine doit être monté directement dans le conteneur Nginx.

Objectif :

modifier `index.html` sur votre machine puis rafraîchir le navigateur pour voir la modification sans reconstruire l'image.

### Question

Expliquez la différence entre :

- un volume Docker ;
- un bind mount.

Ajoutez votre réponse dans `REPONSES.md`.

---

# Partie 8 — Passer à Docker Compose

Nous voulons maintenant arrêter de lancer les conteneurs manuellement.

Créez un fichier :

```text
compose.yaml
```

Il doit contenir au minimum deux services.

## Service 1 : web

Contraintes :

- construit à partir du Dockerfile du dossier `web`;
- nom du conteneur : `mission-web`;
- port `8080:80`;
- connecté au réseau `app_network`;
- bind mount du fichier `index.html`;
- volume `web_logs`.

## Service 2 : tools

Contraintes :

- image `alpine`;
- nom du conteneur : `tools`;
- connecté au même réseau ;
- doit rester démarré suffisamment longtemps pour permettre vos tests.

Votre fichier Compose doit également déclarer :

- le réseau `app_network`;
- le volume `web_logs`.

---

# Partie 9 — Déployer toute l'infrastructure

Votre déploiement doit fonctionner avec :

```bash
docker compose up -d
```

Vérifiez ensuite :

```bash
docker compose ps
docker network ls
docker volume ls
```

Testez également :

```bash
docker compose logs
```

---

# Partie 10 — Test de résilience

Effectuez le test suivant :

1. arrêtez le conteneur web ;
2. redémarrez-le ;
3. vérifiez que le site fonctionne toujours ;
4. supprimez les conteneurs avec Docker Compose ;
5. redémarrez l'infrastructure.

Commandes autorisées :

```bash
docker compose stop
docker compose start
docker compose down
docker compose up -d
```

### Attention

Testez aussi :

```bash
docker compose down -v
```

Puis répondez à cette question :

> Quelle différence observez-vous entre `docker compose down` et `docker compose down -v` ?

Ajoutez la réponse dans `REPONSES.md`.

---

# Partie 11 — Inspection et diagnostic

Sans modifier la configuration, retrouvez les informations suivantes :

- adresse IP du conteneur `mission-web`;
- réseau auquel il appartient ;
- volumes montés ;
- ports publiés ;
- image utilisée.

Vous pouvez utiliser :

```bash
docker inspect mission-web
```

### Challenge

Essayez de retrouver uniquement l'adresse IP sans lire tout le JSON retourné par `docker inspect`.

---

# Partie 12 — Nettoyage

À la fin de la mission, vous devez savoir nettoyer votre environnement.

Supprimez :

- les conteneurs du projet ;
- les conteneurs inutilisés ;
- les images de test inutiles ;
- les réseaux inutilisés.

Mais attention :

> Ne supprimez pas aveuglément toutes les images ou tous les volumes présents sur la machine.

Expliquez pourquoi cette pratique pourrait être dangereuse sur un serveur réel.

---

# Livrables attendus

Votre dossier final doit contenir :

```text
docker-mission/
│
├── web/
│   ├── index.html
│   └── Dockerfile
│
├── compose.yaml
│
└── REPONSES.md
```

Vous devez également être capable de démontrer :

- votre image Docker ;
- vos conteneurs ;
- votre réseau ;
- votre volume ;
- votre site accessible dans le navigateur ;
- l'arrêt et le redémarrage de l'infrastructure avec Docker Compose.

---

# Critères de validation

La mission est réussie si :

- [ ] Docker fonctionne correctement ;
- [ ] l'image `mission-web:v1` est construite ;
- [ ] le site fonctionne sur `localhost:8080` ;
- [ ] les conteneurs communiquent sur `app_network` ;
- [ ] le volume conserve les données ;
- [ ] le bind mount permet de modifier le site sans rebuild ;
- [ ] `docker compose up -d` déploie tout le projet ;
- [ ] `docker compose down` arrête et supprime correctement les conteneurs ;
- [ ] les réponses techniques sont présentes dans `REPONSES.md`.

---

# Bonus — Pour aller plus loin

## Bonus 1 — Healthcheck

Ajoutez un `healthcheck` au service web afin de vérifier automatiquement que Nginx répond correctement.

## Bonus 2 — Restart policy

Ajoutez une politique :

```yaml
restart: unless-stopped
```

Puis expliquez son intérêt.

## Bonus 3 — Tags d'image

Construisez une deuxième version :

```text
mission-web:v2
```

Modifiez le contenu du site et comparez les deux images.

## Bonus 4 — Docker Hub

Si vous disposez d'un compte Docker Hub :

1. taguez votre image ;
2. connectez-vous ;
3. poussez l'image dans votre registre ;
4. supprimez l'image locale ;
5. récupérez-la avec `docker pull`.

---

# Questions pour l'échange en groupe

À la fin de l'exercice, soyez prêts à répondre oralement aux questions suivantes :

1. Pourquoi un conteneur n'est-il pas une machine virtuelle ?
2. Pourquoi évite-t-on de stocker les données importantes directement dans un conteneur ?
3. À quoi sert un réseau Docker personnalisé ?
4. Pourquoi Docker Compose devient-il intéressant dès que plusieurs conteneurs doivent travailler ensemble ?
5. Dans quel cas utiliseriez-vous un bind mount plutôt qu'un volume ?
6. Qu'est-ce qui est réellement supprimé lorsqu'on supprime un conteneur ?
7. Que se passe-t-il si deux conteneurs essaient d'utiliser le même port de la machine hôte ?
8. Pourquoi éviter de travailler uniquement avec le tag `latest` ?
9. Pourquoi faut-il préférer des images de base légères et maintenues ?
10. Quelle partie de cette mission vous semble la plus proche d'un vrai travail DevOps ?

---

# Conseil

Ne cherchez pas simplement à faire fonctionner les commandes.

Pour chaque étape, posez-vous toujours trois questions :

```text
Qu'est-ce que je viens de créer ?
Où est-ce stocké ?
Que se passe-t-il si je supprime le conteneur ?
```

Si vous savez répondre à ces trois questions, vous commencez réellement à comprendre Docker.
