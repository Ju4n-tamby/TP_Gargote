# Niveau 0 — Prise en main

---

## 0.2 Lire le schéma relationnel

1. Schéma relationnel des 4 tables :

- categorie (__id__, nom)
- plat (__id__, nom, prix, #id_categorie)
- commande (__id__, date_commande, numero_table)
- commande_ligne (__id__, #id_commande, #id_plat, quantite, prix_unitaire)

2. Pourquoi commande_ligne contient-elle une colonne prix_unitaire, alors que le prix est déjà dans plat ? Donnez un exemple concret où les deux valeurs seraient différentes.

> La colonne `prix_unitaire` dans la table `commande_ligne` est utilisée pour stocker le prix du plat au moment de la commande. Cela permet de conserver l'historique des prix, même si le prix du plat change dans la table `plat` par la suite. Par exemple, si un plat coûte 1000 aujourd'hui et qu'il est commandé, mais que demain le prix est augmenté à 9000, les commandes passées avant l'augmentation conserveront le prix de 8000 dans `commande_ligne`, tandis que les nouvelles commandes utiliseront le nouveau prix de 9000.

3. Que se passe-t-il pour les lignes d’une commande quand on supprime la commande ? Quelle instruction en est responsable ?

> Lorsque l'on supprime une commande, toutes les lignes associées à cette commande dans la table `commande_ligne` sont également supprimées. Cela est dû à la contrainte de clé étrangère avec l'option `ON DELETE CASCADE` définie sur la colonne `id_commande` de la table `commande_ligne`. Cette instruction garantit que la suppression d'une commande entraîne automatiquement la suppression de toutes ses lignes associées.

4. Repérez trois données absurdes que le schéma actuel accepte sans broncher (indice : essayez d’insérer à la main un prix négatif). Notez-les : vous les corrigerez au niveau 1 et 2.

> Trois données absurdes que le schéma actuel accepte sans broncher sont :
1. Un prix négatif pour un plat (par exemple, `-500`).
2. Une quantité négative pour une ligne de commande (par exemple, `-2`).
3. Un numéro de table nul ou négatif (par exemple, `0` ou `-1`).

---

## 0.3 Interroger l’API à la main

5. Pour la requête b, quel code obtenez-vous ? Pourquoi pas 200 ?

> Pour la requete b, j(obtient un code 201 Created car la requête POST a été utilisée pour créer un nouveau plat. Le code 201 indique que la ressource a été créée avec succès, tandis que le code 200 est généralement utilisé pour les requêtes GET ou PUT qui ne créent pas de nouvelles ressources.

6. Pour la requête d, qui bloque la suppression : PHP ou PostgreSQL ? Montrez la ligne de code qui transforme ce blocage en réponse 409.

> Pour la requête d, c'est PostgreSQL qui bloque la suppression en raison de la contrainte de clé étrangère. La ligne de code qui transforme ce blocage en réponse 409 se trouve dans le contrôleur PHP Plats.php :
```php
        try {
            $this->plats->delete($id);
        } catch (\Throwable $e) {
            // Le plat apparaît dans des commandes : la clé étrangère bloque la suppression.
            return $this->erreur('Ce plat figure déjà dans des commandes, il ne peut pas être supprimé.', 409);
        }
```

7. Créez une commande avec curl (table 6, deux Romazava et un Café). Écrivez la commande curl complète.

```bash
curl -i -X POST http://localhost:8080/api/commandes      -H "Content-Type: application/json"      -d '{"numero_table": 6, "lignes": [{"id_plat": 4, "quantite": 2}, {"id_plat": 10, "quantite": 1}]}'
```