# PalmStreet auto-hébergé sur ton Mac mini — guide complet

Ce dossier remplace entièrement Netlify : le jeu, le classement et le
journal des visites tournent directement sur ton Mac mini, via Docker
(que tu as déjà configuré avec Colima). Pour que **tout le monde** puisse
y jouer depuis internet (pas seulement toi via Tailscale), on utilise
**Tailscale Funnel** — une fonctionnalité de Tailscale qui expose un
service local sur une vraie URL publique HTTPS, sans toucher à ta box
internet ni ouvrir de port.

## Contenu du dossier

```
PalmStreet/
├── server.js          <- remplace scores.js + visits.js de Netlify
├── package.json
├── Dockerfile
├── docker-compose.yml
├── index.html          <- ton jeu (copie identique servie par server.js via public/)
├── public/
│   └── index.html     <- copie servie en prod, à garder synchro avec celle-ci
└── data/              <- créé automatiquement, contient scores.json et visits.json
```

## 1. Cloner le dépôt sur le Mac mini

Connecte-toi en SSH sur le Mac mini et clone le dépôt GitHub (une seule
fois — les mises à jour suivantes se font avec `git pull`, voir section 6) :

```
ssh jussan@games-carlitos.tail736807.ts.net
git clone https://github.com/charlit/PalmStreet.git
```

## 2. Se connecter au Mac mini et changer le mot de passe des visites

```
ssh jussan@games-carlitos.tail736807.ts.net
cd PalmStreet
nano docker-compose.yml
```

Change la ligne `VISITS_KEY=change-moi-avant-de-publier` par un mot de
passe à toi. Sauvegarde (Ctrl+O puis Entrée, Ctrl+X pour quitter nano).

## 3. Lancer le serveur avec Docker

Toujours connecté en SSH sur le Mac mini :

```
colima start --vm-type=vz   # si Colima n'est pas déjà démarré
cd PalmStreet
docker compose up -d --build
```

Vérifie que ça tourne :
```
docker compose logs -f
```
Tu dois voir `PalmStreet auto-hébergé, écoute sur le port 8082`. Ctrl+C
pour quitter l'affichage des logs (le serveur continue de tourner).

Teste en local sur le Mac mini :
```
curl http://localhost:8082/api/scores
```
Tu dois voir `[]`.

## 4. Rendre le jeu accessible à tout le monde (Tailscale Funnel)

Toujours en SSH sur le Mac mini :

```
sudo tailscale funnel --bg 8082
```

Tailscale affiche alors une URL publique du style :
```
https://games-carlitos.tail736807.ts.net/
```

**C'est cette adresse que tu partages** — n'importe qui peut l'ouvrir et
jouer, sans avoir Tailscale installé de son côté.

Pour vérifier que le partage est actif :
```
sudo tailscale funnel status
```

Pour arrêter de partager publiquement (le jeu reste accessible à toi
seul via Tailscale normal) :
```
sudo tailscale funnel --bg off
```

## 5. Ce qui change par rapport à Netlify

- Le jeu n'est en ligne que si **le Mac mini est allumé et connecté** —
  contrairement à Netlify qui reste toujours actif sur ses serveurs.
- `docker compose logs -f` remplace les logs de fonctions Netlify.
- Les scores et visites sont dans `PalmStreet/data/scores.json`
  et `visits.json` — tu peux les ouvrir avec `cat data/scores.json` ou
  les éditer/sauvegarder comme n'importe quel fichier.
- Pour consulter les visites avec une page lisible (tableau triable,
  filtre, statistiques) : `https://TON_URL_FUNNEL/visits.html`, puis entre
  ton `VISITS_KEY` quand elle te la demande. L'API brute reste disponible
  sur `https://TON_URL_FUNNEL/api/visits?key=TON_MOT_DE_PASSE`.

## 6. Mettre à jour le jeu plus tard

Quand une nouvelle version est disponible sur GitHub (nouveau personnage,
figure, combat de boss, etc.), connecte-toi en SSH sur le Mac mini et :

```
cd PalmStreet
git pull
docker-compose up -d --build
```

`docker-compose up -d --build` reconstruit l'image avec le nouveau code et
redémarre le conteneur — pas besoin de `docker compose restart` séparément.

## 7. Démarrage automatique au redémarrage du Mac mini

Le conteneur a `restart: unless-stopped`, donc si Docker/Colima
redémarre, le conteneur repart tout seul. Mais si le Mac mini lui-même
redémarre, il faut que Colima soit relancé aussi. Pour automatiser ça
complètement, il faudrait configurer un agent `launchd` qui lance
`colima start` puis `docker compose up -d` au démarrage — dis-moi si tu
veux que je te prépare ce fichier en plus.
