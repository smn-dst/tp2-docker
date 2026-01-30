# Docker — Volumes & Networks  
Exercices de persistance et de réseau (niveau débutant)

Ce README résume **pas à pas** les manipulations réalisées dans les **exercices 1 et 2**, afin de comprendre :
- la **persistance des données** avec les volumes
- la **communication réseau** entre conteneurs Docker

---

## 🧪 Exercice 1 — Persistance des données avec les volumes

### 🎯 Objectif
Comprendre pourquoi les données disparaissent quand un conteneur est supprimé,  
et comment **les volumes Docker permettent de persister ces données**.

> Idée clé du cours :  
> Le *writable layer* d’un conteneur est **éphémère**.  
> **Les volumes** sont le mécanisme recommandé pour stocker des données persistantes.

---

### Étape 1 — Créer un volume nommé
```bash
docker volume create mydata
docker volume ls

### Étape 2
docker run --name writer -v mydata:/data alpine \
  sh -c 'echo "Hello Docker" > /data/hello.txt && ls -la /data'

### Étape 3
docker rm -f writer

### Etape 4
docker run --rm -v mydata:/data alpine \
  sh -c 'ls -la /data && cat /data/hello.txt'

### Bonus
docker run --rm \
  -v mydata:/data:ro \
  -v "$PWD":/backup \
  alpine sh -c 'tar -czf /backup/backup_mydata.tar.gz -C /data .'

docker volume rm mydata
docker volume create mydata

docker run --rm \
  -v mydata:/data \
  -v "$PWD":/backup \
  alpine sh -c 'tar -xzf /backup/backup_mydata.tar.gz -C /data'




## 🧪 Exercice 2
docker network create mynet
docker network ls

docker run -dit --name c1 --network mynet alpine sh
docker run -dit --name c2 --network mynet alpine sh

docker exec -it c2 sh
ping c1

docker run -d \
  --name web \
  --network mynet \
  -p 8080:80 \
  nginx

curl http://localhost:8080

docker exec -it c2 sh
apk add --no-cache curl
curl http://web



# Docker — Volumes & Networks  
Cours + Exercices + Questions (niveau débutant)

Ce document regroupe :
- les **notions du cours**
- les **exercices 1 et 2 réalisés**
- les **réponses aux questions de compréhension**

Objectif : comprendre **la persistance des données** et **la communication réseau** avec Docker.

---

## 🔹 Volumes et Networks — Notions clés

### Pourquoi les volumes ?

**Idée clé :**  
Le *writable layer* d’un conteneur **disparaît quand le conteneur est supprimé**.

👉 Sans volume :
- l’image existe
- le conteneur tourne
- **les données sont perdues à l’arrêt**

👉 Avec un volume :
- les données sont **séparées du conteneur**
- elles **persistent**
- elles peuvent être **sauvegardées, restaurées, migrées**

Les volumes sont le mécanisme recommandé pour :
- bases de données
- uploads
- caches
- artefacts applicatifs

---

## 🔹 Choisir le bon type de montage

### Volume (named)
- Géré par Docker (`/var/lib/docker/...`)
- Indépendant du host
- Persistant
- Facile à sauvegarder / migrer
- Partageable entre conteneurs
- 👉 **Recommandé en production**

### Bind mount (host)
- Monte un chemin exact du host dans le conteneur
- Dépend de l’OS et du filesystem
- Idéal pour le dev (code live)
- Risque d’exposer trop de fichiers du host

### tmpfs
- Stockage en mémoire (RAM)
- Jamais écrit sur disque
- Disparaît à l’arrêt du conteneur
- Idéal pour secrets temporaires et caches sensibles

---

### Tableau comparatif

| Critère | Volume (named) | Bind mount | tmpfs |
|------|----------------|-----------|-------|
| Géré par Docker | ✅ | ❌ | ✅ |
| Stockage | Disque Docker | Disque host | RAM |
| Persistance | ✅ | ✅ | ❌ |
| Dépendance OS | ❌ | ✅ | ❌ |
| Sécurité | ✅ | ⚠️ | 🔒 |
| Performance | ✅ | ⚠️ | 🚀 |
| Usage | Prod / DB | Dev / code | Secrets |

---

## 🔹 Cycle de vie des volumes

**CRÉER → MONTER → INSPECTER → BACKUP → NETTOYER**

### Commandes essentielles
```bash
docker volume ls
docker volume create myvol
docker inspect myvol
docker volume rm myvol
