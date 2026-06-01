# 02 — Quick Tunnel temporaire

> À lire avant : `00_Principes_et_Securite.md`.
>
> Le document `01_Preparer_Domaine_Cloudflare.md` n’est pas obligatoire pour un Quick Tunnel, car un Quick Tunnel peut fonctionner sans domaine personnalisé. Il devient utile si un passage vers un tunnel permanent est prévu.

---

## 1. À quoi sert un Quick Tunnel ?

Un Quick Tunnel sert à exposer rapidement une application locale avec une adresse temporaire du type :

```text
https://quelque-chose.trycloudflare.com
```

C’est utile pour :

- tester une application locale ;
- montrer un projet pendant une durée courte ;
- vérifier si une application répond depuis Internet ;
- faire un test avant un tunnel permanent ;
- dépanner sans toucher au domaine officiel.

---

## 2. Limites importantes

> [!WARNING]
> Un Quick Tunnel n’est pas adapté à une mise en production.

À retenir :

- adresse aléatoire ;
- domaine en `trycloudflare.com` ;
- pas de domaine personnalisé ;
- pas prévu pour rester permanent ;
- arrêt du lien quand la commande s’arrête ;
- pas de service automatique par défaut ;
- protection à prévoir si l’application est sensible.

---

## 3. Préparer l’application locale

Avant de lancer le Quick Tunnel, tester l’application localement.

Linux/macOS :

```bash
curl -I http://localhost:8080
```

Windows :

```powershell
curl.exe -I http://localhost:8080
```

Si l’application ne répond pas localement, le Quick Tunnel ne fonctionnera pas.

---

## 4. Quick Tunnel sur Windows

### 4.1. Vérifier que `cloudflared` est disponible

```powershell
cloudflared --version
```

Si `cloudflared.exe` n’est pas dans le `PATH`, se placer dans le dossier où se trouve l’exécutable :

```powershell
cd C:\Cloudflared\bin
```

### 4.2. Lancer le tunnel temporaire

Pour exposer une application locale sur le port `8080` :

```powershell
cloudflared tunnel --url http://localhost:8080
```

Si l’exécutable est dans le dossier courant :

```powershell
.\cloudflared.exe tunnel --url http://localhost:8080
```

Cloudflare affiche ensuite une URL temporaire en :

```text
https://xxxxx.trycloudflare.com
```

### 4.3. Arrêter le Quick Tunnel

Dans la fenêtre du terminal :

```text
CTRL + C
```

Dès que la commande s’arrête, le lien ne fonctionne plus.

---

## 5. Quick Tunnel sur Linux

### 5.1. Vérifier `cloudflared`

```bash
cloudflared --version
```

### 5.2. Lancer le tunnel temporaire

```bash
cloudflared tunnel --url http://localhost:8080
```

Cloudflare affiche une URL temporaire en :

```text
https://xxxxx.trycloudflare.com
```

### 5.3. Arrêter

```text
CTRL + C
```

---

## 6. Quick Tunnel avec Docker

Oui, un Quick Tunnel peut être lancé avec Docker.

Mais le point essentiel est le suivant :

> [!IMPORTANT]
> Dans un conteneur Docker, `localhost` désigne le conteneur lui-même, pas forcément la machine hôte.

---

### 6.1. Application sur l’hôte avec Docker Desktop Windows/macOS

Si l’application tourne sur la machine hôte en `http://localhost:8080`, depuis Docker Desktop il faut généralement viser :

```bash
docker run --rm cloudflare/cloudflared:latest tunnel --url http://host.docker.internal:8080
```

---

### 6.2. Application sur l’hôte avec Docker sous Linux

Option avec réseau hôte :

```bash
docker run --rm --network host cloudflare/cloudflared:latest tunnel --url http://localhost:8080
```

Alternative sans réseau hôte :

```bash
docker run --rm \
  --add-host=host.docker.internal:host-gateway \
  cloudflare/cloudflared:latest \
  tunnel --url http://host.docker.internal:8080
```

---

### 6.3. Application dans un autre conteneur Docker

Créer un réseau commun :

```bash
docker network create tunnel_net
```

Application exemple :

```bash
docker run -d --name web --network tunnel_net nginx:alpine
```

Quick Tunnel :

```bash
docker run --rm --network tunnel_net \
  cloudflare/cloudflared:latest \
  tunnel --url http://web:80
```

Ici, `web` est le nom du conteneur de l’application.

---

## 7. Quand passer au tunnel permanent ?

Le tunnel permanent devient nécessaire pour :

- une adresse propre comme `app.mondomaine.fr` ;
- un service qui redémarre automatiquement ;
- une configuration durable ;
- une protection via Cloudflare Access ;
- une publication sérieuse ou régulière.

Dans ce cas, lire ensuite :

```text
01_Preparer_Domaine_Cloudflare.md
03_Tunnel_Permanent.md
```

---

## Sources utiles

- Quick Tunnels : https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/do-more-with-tunnels/trycloudflare/
- Cloudflare Tunnel : https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/
