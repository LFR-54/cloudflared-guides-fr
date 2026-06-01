# 05 — Mémo commandes cloudflared

> Commandes utiles, classées par catégorie.
>
> Les exemples de port, domaine et token doivent être remplacés par les vraies valeurs.

---

## 1. Tester une application locale

### Linux/macOS

```bash
curl -I http://localhost:8080
```

### Windows

```powershell
curl.exe -I http://localhost:8080
```

### Tester une autre machine du réseau local

```bash
curl -I http://192.168.1.50:8080
```

---

## 2. Vérifier `cloudflared`

```bash
cloudflared --version
```

---

## 3. Quick Tunnel

### Windows/Linux

```bash
cloudflared tunnel --url http://localhost:8080
```

### Windows si exécutable dans le dossier courant

```powershell
.\cloudflared.exe tunnel --url http://localhost:8080
```

### Docker — application sur hôte Docker Desktop

```bash
docker run --rm cloudflare/cloudflared:latest tunnel --url http://host.docker.internal:8080
```

### Docker — Linux avec réseau hôte

```bash
docker run --rm --network host cloudflare/cloudflared:latest tunnel --url http://localhost:8080
```

### Docker — application dans le même réseau Docker

```bash
docker run --rm --network tunnel_net cloudflare/cloudflared:latest tunnel --url http://web:80
```

### Arrêter un Quick Tunnel

```text
CTRL + C
```

---

## 4. Windows — service `cloudflared`

### Vérifier le service

```powershell
Get-Service cloudflared
```

### Démarrer

```powershell
Start-Service cloudflared
```

### Redémarrer

```powershell
Restart-Service cloudflared
```

### Installer le service avec token

```powershell
cloudflared service install <TOKEN_DU_TUNNEL>
```

ou :

```powershell
cloudflared.exe service install <TOKEN_DU_TUNNEL>
```

### Désinstaller le service

```powershell
cloudflared service uninstall
```

Si l’exécutable est dans le dossier courant :

```powershell
.\cloudflared.exe service uninstall
```

---

## 5. Linux — service `cloudflared`

### Statut

```bash
sudo systemctl status cloudflared
```

### Démarrer

```bash
sudo systemctl start cloudflared
```

### Activer au démarrage

```bash
sudo systemctl enable cloudflared
```

### Redémarrer

```bash
sudo systemctl restart cloudflared
```

### Logs en direct

```bash
sudo journalctl -u cloudflared -f
```

### Installer le service avec token

```bash
sudo cloudflared service install <TOKEN_DU_TUNNEL>
```

### Désinstaller le service

```bash
sudo cloudflared service uninstall
```

---

## 6. Installer `cloudflared` sur Debian/Ubuntu/Raspberry Pi OS

Exemple représentatif. Si Cloudflare affiche des commandes dans son Dashboard, privilégier celles du Dashboard.

```bash
sudo mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg >/dev/null

echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared any main' | sudo tee /etc/apt/sources.list.d/cloudflared.list

sudo apt-get update && sudo apt-get install cloudflared
```

---

## 7. Docker — tunnel permanent

### Lancer durablement

```bash
docker run -d \
  --name cloudflared \
  --restart unless-stopped \
  cloudflare/cloudflared:latest \
  tunnel --no-autoupdate run --token <TOKEN_DU_TUNNEL>
```

### Voir les conteneurs

```bash
docker ps
```

### Logs

```bash
docker logs cloudflared
```

### Logs en direct

```bash
docker logs -f cloudflared
```

### Redémarrer

```bash
docker restart cloudflared
```

### Supprimer le conteneur

```bash
docker rm -f cloudflared
```

### Supprimer l’image

```bash
docker image rm cloudflare/cloudflared:latest
```

---

## 8. Docker Compose — tunnel permanent

```yaml
services:
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    restart: unless-stopped
    command: tunnel --no-autoupdate run --token ${CLOUDFLARED_TOKEN}
```

Fichier `.env` :

```env
CLOUDFLARED_TOKEN=valeur_du_token
```

Lancer :

```bash
docker compose up -d
```

Voir les logs :

```bash
docker compose logs -f cloudflared
```

Arrêter :

```bash
docker compose down
```

---

## 9. Désinstaller complètement un tunnel permanent

### Côté Cloudflare

```text
Cloudflare Dashboard
→ Protect & Connect
→ Networking
→ Tunnels
→ choisir le tunnel
→ supprimer la route publique
→ supprimer le tunnel si aucun service ne l’utilise encore
```

### Windows

```powershell
Stop-Service cloudflared
cloudflared service uninstall
winget uninstall --id Cloudflare.cloudflared
```

### Linux Debian/Ubuntu/Raspberry Pi OS

```bash
sudo cloudflared service uninstall
sudo apt-get purge cloudflared
sudo apt-get autoremove
```

### Docker

```bash
docker rm -f cloudflared
docker image rm cloudflare/cloudflared:latest
```

### Docker Compose

```bash
docker compose down
```

---

## 10. DNS

### Voir les serveurs DNS du domaine

```bash
nslookup -type=ns mondomaine.fr
```

### Résoudre un sous-domaine

```bash
nslookup app.mondomaine.fr
```

### Tester depuis un DNS précis

```bash
nslookup app.mondomaine.fr 1.1.1.1
```

```bash
nslookup app.mondomaine.fr 8.8.8.8
```

---

## 11. Ports locaux

### Linux — voir les ports en écoute

```bash
ss -ltnp
```

Filtrer un port :

```bash
ss -ltnp | grep :8080
```

### Windows — voir un port

```powershell
netstat -ano | findstr :8080
```

---

## 12. Fiche express

```text
Domaine :
Sous-domaine :
Tunnel :
Machine cloudflared :
Système :
Service URL :
Port local :
Protection Access : Oui / Non
Date :
Notes :
```

---

## Sources utiles

- Cloudflare Tunnel : https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/
- Quick Tunnels : https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/do-more-with-tunnels/trycloudflare/
- Création d’un tunnel : https://developers.cloudflare.com/tunnel/setup/
- Exécution comme service : https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/do-more-with-tunnels/local-management/as-a-service/
- Désinstallation du service `cloudflared` : https://developers.cloudflare.com/tunnel/troubleshooting/
- Téléchargement et paquets `cloudflared` : https://developers.cloudflare.com/tunnel/downloads/
- Dépannage : https://developers.cloudflare.com/tunnel/troubleshooting/
