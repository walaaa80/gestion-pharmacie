# 💊 Gestion Pharmacie

Application de gestion d'officine développée en **Java**, avec une interface graphique **JavaFX**, une version **console** et une base **Oracle**. Elle couvre la vente, le suivi du stock, le réapprovisionnement et la traçabilité des mouvements, avec deux rôles : **pharmacien** et **gestionnaire**.

> Projet réalisé par **Tasnim Abid** et **Wala Ouaili** : classe IOT2, FST (Université de Tunis El Manar), janvier 2026.
> Le rapport complet est dans [`docs/rapport-projet.pdf`](docs/rapport-projet.pdf).

## Fonctionnalités

| Rôle | Actions |
|---|---|
| 👨‍⚕️ **Pharmacien** | Enregistrer une vente · Annuler une vente (stock restitué) · Consulter les ventes du jour · Consulter le stock |
| 👨‍💼 **Gestionnaire** | Ajouter un médicament · Mettre à jour le stock · Voir les stocks critiques (≤ 10) · Passer une commande de réapprovisionnement · Consulter l'historique des mouvements · Rapport des ventes (quantités et chiffre d'affaires) |

Chaque opération qui touche au stock (vente, annulation, commande, ajustement) est exécutée dans **une seule transaction** (`commit` / `rollback`) et journalisée dans `StockHistorique`.

![Diagramme de cas d'utilisation](docs/images/diagramme-cas-utilisation.png)

## Architecture

```
┌────────────────────┐   ┌──────────────────┐
│ PharmacieApp       │   │ Main             │   Interfaces
│ (JavaFX)           │   │ (console)        │
└─────────┬──────────┘   └────────┬─────────┘
          └──────────┬────────────┘
                     ▼
        ┌─────────────────────────┐
        │ Utilisateur (abstract)  │   Modèle métier
        │  ├─ Pharmacien          │   (règles de vente / stock
        │  └─ Gestionnaire        │    dans ces classes)
        └────────────┬────────────┘
                     ▼
   Medicament · Vente · Commande · StockHistorique   (entités)
                     │
                DAO ─┴─ DatabaseConnection ──► Oracle (JDBC)
```

| Table | Rôle |
|---|---|
| `Pharmacien`, `Gestionnaire` | Comptes (login unique) |
| `Client` | Clients (optionnel sur une vente) |
| `Medicament` | Catalogue et stock courant |
| `Vente`, `Commande` | Transactions |
| `StockHistorique` | Journal des mouvements (+ entrée / − sortie) |

## Démarrage

### Prérequis
- **JDK 17+** et **Maven 3.9+**
- **Oracle Database XE** (ou toute base Oracle accessible en JDBC)

### 1. Base de données

Voir [`database/`](database/README.md). En résumé :

```sql
-- en SYSTEM
@database/00_create_user.sql
-- puis connecté en PHARMACIE
@database/01_schema.sql
@database/02_seed.sql
```

### 2. Configuration de la connexion

```bash
cp db.properties.example db.properties   # puis édite le mot de passe
```

`db.properties` est ignoré par Git. Tu peux aussi utiliser les variables d'environnement `PHARMACIE_DB_URL`, `PHARMACIE_DB_USER`, `PHARMACIE_DB_PASSWORD` (prioritaires).

> Oracle XE 18c+ : si l'URL `…:1521:xe` échoue, utilise le service PDB : `jdbc:oracle:thin:@//localhost:1521/XEPDB1`.

### 3. Lancer

```bash
mvn javafx:run                 # interface graphique
mvn compile exec:java          # version console
```

### Comptes de démonstration

| Rôle | Login | Mot de passe |
|---|---|---|
| Pharmacien | `karim` | `pharm2025` |
| Gestionnaire | `gestionnaire1` | `admin` |

## Structure du dépôt

```
.
├── pom.xml
├── db.properties.example
├── database/            # scripts SQL (utilisateur, schéma, données de démo)
├── docs/                # rapport PDF + diagramme de cas d'utilisation
└── src/main/java/pharmacie/
    ├── PharmacieApp.java        # interface JavaFX
    ├── Main.java                # interface console
    ├── Utilisateur.java · Pharmacien.java · Gestionnaire.java
    ├── Medicament.java · Vente.java · Commande.java · StockHistorique.java
    ├── DAO.java
    ├── DatabaseConnection.java
    └── Launcher.java            # point d'entrée pour un JAR
```

## Limites connues et pistes d'amélioration

- 🔐 **Mots de passe stockés en clair** dans la base (démo uniquement) → hachage `bcrypt`/Argon2 à implémenter.
- 🧪 **Aucun test automatisé** : l'intégration continue ne fait que compiler.
- 🧱 La logique SQL est encore répartie entre `Pharmacien`, `Gestionnaire` et `PharmacieApp` → à centraliser dans la couche `DAO`.
- 🛒 Une commande ajoute **immédiatement** le stock (pas de statut « commandée / reçue »).
- 📅 Pistes du rapport : lots et dates de péremption, export PDF, recherche et filtres, API REST.

## Licence

[MIT](LICENSE)
