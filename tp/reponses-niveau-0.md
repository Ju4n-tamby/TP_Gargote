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

---

## 0.4 Suivre une requête de bout en bout

| # | Étape | Fichier | Fonction / ligne |
|---|---|---|---|
| 1 | Le formulaire est soumis | `components/PlatForm.vue` | `enregistrer()` |
| 2 | La requête HTTP est construite | `src/api.js` | `requete()` |
| 3 | Vite transmet `/api/...` au port 8080 | `frontend/vite.config.js` | `server.proxy['/api']` |
| 4 | CodeIgniter choisit le contrôleur | `backend/app/Config/Routes.php` | route `POST plats` → `Plats::create()` |
| 5 | Le contrôleur lit le JSON | `backend/app/Controllers/Api/ApiController.php` | `donneesJson()` |
| 6 | Les règles de validation sont vérifiées | `backend/app/Models/PlatModel.php` | `$validationRules`, lors de `insert()` |
| 7 | La ligne est insérée | `backend/app/Controllers/Api/Plats.php` | `create()` → `$this->plats->insert()` |
| 8 | Le formulaire prévient l’écran parent | `frontend/src/components/PlatForm.vue` | `emit('enregistre', resultat)` |
| 9 | L’écran recharge la liste | `frontend/src/views/CarteView.vue` | `surEnregistrement()` → `charger()` |

8. Où est le code métier de la création d’une commande ? Pourquoi n’est-il pas dans le contrôleur `Commandes.php` ?

> Le code métier se trouve dans `backend/app/Services/CommandeService.php`, dans la méthode
`creer()`. Le contrôleur `Commandes.php` reçoit la requête et renvoie la réponse HTTP ;
le service centralise les règles métier et la transaction, ce qui évite de mélanger les
responsabilités et permet de réutiliser et de tester ces règles indépendamment de HTTP.

---

## 0.5 Repérer les notions de Vue

| Notion                 | À quoi ça sert                                                    | Votre exemple                                                                                                                                                                                                                            |
|------------------------|-------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `ref()`                | une variable dont le changement met l’écran à jour                | frontend/src/views/CommandeView.vue:7-17<br/> const plats = ref([]) et const numeroTable = ref('') : ce sont des variables réactives, donc tout changement les met automatiquement à jour dans l’écran.                                  |
| `computed()`           | une valeur calculée à partir d’autres, recalculée automatiquement | frontend/src/App.vue:14-15<br/>const vueCourante = computed(() => onglets.find((o) => o.id === ongletActif.value).composant) : c’est une valeur calculée à partir d’autres données, recalculée automatiquement quand celles-ci changent. |
| `v-for`                | répéter un élément pour chaque élément d’une liste                | frontend/src/App.vue:26-3<br/><button v-for="onglet in onglets" ...> : Vue répète le bloc pour chaque élément de la liste onglets.                                                                                                       |
| `v-if` / `v-else`      | afficher un élément seulement si une condition est vraie          | frontend/src/views/HistoriqueView.vue:23-49<br/> l’élément s’affiche seulement si la condition est vraie, sinon on affiche l’autre.                                                                                                      |
| `v-model`              | lier un champ de formulaire à une variable, dans les deux sens    | frontend/src/views/CommandeView.vue:72-75<br/> <input v-model.number="numeroTable" ... /> : le champ de formulaire et la variable numeroTable sont liés dans les deux sens.                                                              |
| `defineProps`          | recevoir des données du composant parent                          | frontend/src/components/PlatList.vue:6-8<br/> const props = defineProps({ plats: { type: Array, required: true } }) : le composant reçoit des données du parent via les props.                                                           |
| `defineEmits` / `emit` | envoyer un événement au composant parent                          | frontend/src/components/PlatList.vue:10-11 et 28-29<br/> const emit = defineEmits(['modifier', 'supprimer']) puis @click="emit('modifier', plat)" : le composant envoie un événement au parent pour lui demander une action.             |
| `onMounted`            | exécuter du code quand le composant apparaît                      | frontend/src/views/HistoriqueView.vue:10-12<br/> onMounted(async () => { commandes.value = await api.commandes.lister() }) : le code s’exécute quand le composant est affiché pour la première fois.                                     |

9. Panier.vue ne modifie jamais la quantité lui-même : il émet plus et moins. Pourquoi ce choix ? Qui modifie réellement la quantité ?

> Panier.vue émet les événements `plus` et `moins` pour déléguer la modification de la quantité au composant parent, qui est responsable de gérer l'état global de la commande. Cela permet de centraliser la logique de gestion des quantités et d'assurer que toutes les modifications passent par le même point, facilitant ainsi le suivi et la cohérence des données. La quantité est réellement modifiée par le composant parent, qui écoute ces événements et met à jour l'état de la commande en conséquence.

10. Dans PlatForm.vue, à quoi sert le watch ? Que se passerait-il sans lui quand on clique sur Modifier d’un autre plat ?

> Le `watch` dans PlatForm.vue sert à surveiller les changements de la prop `plat` et à mettre à jour les champs du formulaire en conséquence. Sans ce `watch`, si l'utilisateur clique sur "Modifier" pour un autre plat, les champs du formulaire ne se mettraient pas à jour avec les nouvelles données du plat sélectionné, et l'utilisateur verrait toujours les informations du plat précédent, ce qui pourrait entraîner des erreurs lors de la modification.

---