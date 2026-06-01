# 01 — Préparer le domaine, Cloudflare et l’application locale

> À lire avant le tunnel permanent.
>
> Pour un Quick Tunnel temporaire, ce document n’est pas obligatoire. Il devient important dès qu’une vraie adresse du type `app.mondomaine.fr` doit être utilisée.

---

## 1. Objectif

Ce document prépare tout ce qui doit exister avant de créer un tunnel permanent :

- un nom de domaine 
- un compte Cloudflare 
- le domaine ajouté dans Cloudflare 
- les serveurs DNS du registrar remplacés par ceux de Cloudflare 
- les DNS existants vérifiés 
- une application locale déjà fonctionnelle 
- les informations locales utiles notées proprement.

Le tunnel permanent ne doit être créé qu’après cette préparation. Sinon, les erreurs DNS, de domaine ou de Service URL deviennent difficiles à distinguer.

---

## 2. Acheter ou posséder un nom de domaine

Un domaine peut être acheté chez un registrar, par exemple :

- OVH 
- Ionos 
- Namecheap 
- Gandi 
- Infomaniak 
- Dynadot 
- Porkbun 
- Cloudflare Registrar si disponible pour le domaine concerné.

Exemple :

```text
mondomaine.fr
```

> [!NOTE]
> Pour Cloudflare Tunnel, un hébergement web classique n’est pas nécessaire. Le point essentiel est de contrôler le nom de domaine et ses DNS.

---

## 3. Créer le compte Cloudflare

Aller sur Cloudflare, créer un compte, puis accéder au Dashboard Cloudflare.

Chemin général :

```text
Cloudflare Dashboard
→ Sign up / Log in
```

Une fois connecté, l’objectif est d’ajouter le domaine dans Cloudflare.

---

## 4. Ajouter le domaine dans Cloudflare

Dans le Dashboard Cloudflare :

```text
Cloudflare Dashboard
→ Add a site
→ entrer le domaine, par exemple mondomaine.fr
→ choisir le plan adapté
→ continuer
```

Cloudflare analyse ensuite les enregistrements DNS existants. Cette étape est importante si le domaine sert déjà à autre chose.

---

## 5. Vérifier les DNS détectés par Cloudflare

Avant de modifier les serveurs DNS chez le registrar, il faut vérifier ce que Cloudflare a détecté.

Dans Cloudflare :

```text
Cloudflare Dashboard
→ sélectionner le domaine
→ DNS
→ Records
```

À contrôler en priorité :

| Type | Utilité | Risque si supprimé |
|---|---|---|
| `A` / `AAAA` | Site existant ou serveur existant | Site qui ne répond plus |
| `CNAME` | Sous-domaines existants | Sous-domaines cassés |
| `MX` | Réception des e-mails | Plus de réception e-mail |
| `TXT` SPF | Autorisation d’envoi e-mail | Mails qui arrivent en spam |
| `TXT` DKIM | Signature des mails | Perte de confiance e-mail |
| `TXT` DMARC | Politique anti-usurpation | Protection e-mail dégradée |

> [!WARNING]
> Si le domaine est déjà utilisé pour les e-mails, les enregistrements `MX`, `SPF`, `DKIM` et `DMARC` doivent être conservés. Une mauvaise migration DNS peut casser l’e-mail du domaine.

---

## 6. Récupérer les nameservers Cloudflare

Pendant l’ajout du domaine, Cloudflare fournit deux serveurs DNS à utiliser.

Exemple fictif :

```text
alice.ns.cloudflare.com
bob.ns.cloudflare.com
```

Ces valeurs sont propres au domaine. Elles ne doivent pas être inventées ni reprises d’un autre domaine.

---

## 7. Modifier les serveurs DNS chez le registrar

Cette étape ne se fait pas dans Cloudflare, mais chez le site où le domaine a été acheté.

Chemin générique chez le registrar :

```text
Registrar
→ Mes domaines / Domaines
→ sélectionner mondomaine.fr
→ DNS / Serveurs DNS / Nameservers
→ remplacer les anciens nameservers
→ enregistrer
```

Exemple :

```text
Avant :
ns1.registrar.com
ns2.registrar.com

Après :
alice.ns.cloudflare.com
bob.ns.cloudflare.com
```

Selon le registrar, le menu peut s’appeler :

- `Serveurs DNS` 
- `Nameservers` 
- `Gestion DNS` 
- `DNS avancé` 
- `Délégation DNS`.

Le principe reste le même : les anciens serveurs DNS du registrar doivent être remplacés par les deux serveurs DNS fournis par Cloudflare.

---

## 8. Attendre l’activation du domaine

Après la modification des nameservers, il faut attendre que Cloudflare voie le domaine comme actif.

Dans Cloudflare :

```text
Cloudflare Dashboard
→ Websites / Sites
→ sélectionner le domaine
→ vérifier le statut du domaine
```

La propagation peut prendre quelques minutes, plusieurs heures, et parfois davantage selon le registrar.

Tant que le domaine n’est pas actif dans Cloudflare, il est préférable de ne pas créer la configuration finale du tunnel permanent.

---

## 9. Choisir le sous-domaine public

Pour publier une application, il est préférable d’utiliser un sous-domaine clair.

Exemples :

```text
app.mondomaine.fr
api.mondomaine.fr
dashboard.mondomaine.fr
panel.mondomaine.fr
monitoring.mondomaine.fr
```

Éviter d’utiliser directement la racine :

```text
mondomaine.fr
```

sauf si c’est volontaire.

Un sous-domaine est plus propre pour organiser plusieurs services, déplacer une application, appliquer Cloudflare Access ou isoler les règles.

---

## 10. Préparer l’application locale

Avant Cloudflare, l’application doit répondre localement.

Exemples :

```text
http://localhost:8080
http://127.0.0.1:3000
http://192.168.1.50:8080
https://localhost:8443
```

Test depuis la machine qui exécutera `cloudflared` :

```bash
curl -I http://localhost:8080
```

Sur Windows :

```powershell
curl.exe -I http://localhost:8080
```

Si l’application est sur une autre machine du réseau local :

```bash
curl -I http://192.168.1.50:8080
```

Si ce test échoue, le problème doit être corrigé avant de créer le tunnel.

---

## 11. Choisir où installer `cloudflared`

`cloudflared` doit être installé sur une machine capable de joindre l’application locale.

| Situation | Machine conseillée pour `cloudflared` | Service URL probable |
|---|---|---|
| Application sur le même PC/serveur | Même machine | `http://localhost:8080` |
| Application sur un autre serveur LAN | Machine stable du LAN | `http://192.168.1.50:8080` |
| Application dans Docker | Hôte Docker ou conteneur dédié | `http://web:80` ou `host.docker.internal` |
| Raspberry Pi toujours allumé | Raspberry Pi | Selon l’adresse locale de l’application |

La machine `cloudflared` doit rester allumée si le tunnel permanent doit rester disponible.

---

## 12. Fiche de préparation

Avant de passer au tunnel permanent, conserver ces informations :

```text
Nom de l’application :
Domaine principal :
Sous-domaine public :
Machine qui héberge l’application :
Machine qui exécutera cloudflared :
Système : Windows / Linux / Docker
Adresse locale de l’application :
Port local :
Protocole local : HTTP / HTTPS
Application sensible : Oui / Non
Protection Cloudflare Access prévue : Oui / Non
```

Exemple :

```text
Nom de l’application : Dashboard maison
Domaine principal : mondomaine.fr
Sous-domaine public : dashboard.mondomaine.fr
Machine qui héberge l’application : serveur-linux
Machine qui exécutera cloudflared : serveur-linux
Système : Linux
Adresse locale de l’application : http://localhost:3000
Port local : 3000
Protocole local : HTTP
Application sensible : Oui
Protection Cloudflare Access prévue : Oui
```

---

## 13. Après cette préparation

Quand le domaine est actif dans Cloudflare, que les DNS importants ont été vérifiés et que l’application répond localement, passer au guide :

```text
03_Tunnel_Permanent.md
```

---

## Sources utiles

- Ajouter un site Cloudflare : https://developers.cloudflare.com/fundamentals/setup/manage-domains/add-site/
- DNS Cloudflare : https://developers.cloudflare.com/dns/
- Cloudflare Tunnel : https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/
- Création d’un tunnel : https://developers.cloudflare.com/tunnel/setup/
