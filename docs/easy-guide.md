# Guide simple — installer DarkMoon dans le cloud

> **Guide facile à lire.** Phrases courtes, mots simples. Les mots difficiles sont expliqués. *(An easy-to-read guide, in French.)*

DarkMoon **n'est pas une boîte à acheter**. C'est un logiciel.
Vous le mettez sur un ordinateur que vous louez dans le cloud.
Et **une seule commande fait tout** : elle crée l'ordinateur, ouvre la porte, et installe DarkMoon.

---

## Avant de commencer, il vous faut 3 choses

1. Votre **clé de licence** DarkMoon. *(Vous la trouvez sur votre espace client.)*
2. Votre **clé d'API pour l'IA**. *(C'est un code donné par le fournisseur d'IA.)*
3. Un **compte cloud** : AWS, Google Cloud, Azure **ou** OVH.
   *Le cloud = des ordinateurs que vous louez sur internet.*

---

## La méthode automatique (une seule commande)

### Étape 1 — Ouvrez le CloudShell de votre cloud
- Dans la console de votre cloud, cliquez sur le bouton **CloudShell**.
  *CloudShell = une petite fenêtre noire, déjà connectée à votre compte. Rien à installer.*

### Étape 2 — Collez une commande
- Sur votre **espace client**, ouvrez la carte **« Deploy to cloud »**.
- Prenez l'option **① Fully automated**. Cliquez **Copy**.
- **Collez** la commande dans le CloudShell.
- Remplacez `<YOUR_LLM_API_KEY>` par **votre clé d'IA**.
- Pour un autre cloud : remplacez `aws` par `gcp`, `azure` ou `ovh`.
- Appuyez sur **Entrée**.

### Étape 3 — Attendez
- L'ordinateur **se crée tout seul**.
- La porte (le port 80) **s'ouvre toute seule**.
- DarkMoon **s'installe tout seul**. Cela prend **environ 3 minutes**.
- L'**adresse** du tableau de bord **s'affiche** à l'écran.

### Étape 4 — Ouvrez DarkMoon
- Copiez l'adresse affichée dans votre navigateur.
- Le tableau de bord DarkMoon s'ouvre. **C'est prêt.** 🎉

---

## Ce qui se fait tout seul

- ✅ Créer l'ordinateur dans le cloud.
- ✅ Ouvrir la porte (le port 80).
- ✅ Installer Docker.
- ✅ Installer DarkMoon.

Vous **n'avez pas** à créer la machine à la main.
Vous **n'avez pas** à ouvrir la porte à la main.
Vous **n'avez pas** à vous connecter en SSH.

---

## La seule chose qu'on ne peut pas faire à votre place

Vous devez être **connecté à VOTRE compte cloud**.
Le CloudShell fait ça pour vous : vous êtes déjà connecté.

On **ne prend jamais** vos identifiants de cloud.
C'est **vous** qui lancez la commande, dans **votre** compte. C'est plus sûr.

---

## Si quelque chose ne marche pas

- Connectez-vous à votre machine.
- Tapez : **`darkmoon doctor`**
- Doctor vérifie tout et vous dit quoi faire.

---

## Choisir la taille de la machine

La commande prend une taille par défaut (« standard »).
Vous pouvez en choisir une autre avec `--profile` :

| Besoin | Écrivez | Cœurs / Mémoire |
|---|---|---|
| Petit (essai) | `--profile minimum` | 2 / 8 Go |
| Normal | `--profile standard` | 4 / 16 Go |
| Puissant | `--profile performance` | 8 / 32 Go |
| Industriel | `--profile industrial` | 16 / 64 Go |

---

## Pour aller plus loin (personnes techniques)

- La commande complète et les 3 méthodes : voir [one-command.md](one-command.md).
- Par cloud : [AWS](aws.md) · [GCP](gcp.md) · [Azure](azure.md) · [OVH](ovh.md).
- Sécurité (mettre les clés dans un coffre-fort) : [security.md](security.md).
