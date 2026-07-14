# Site de référencement des directives ESTI

Site statique qui liste les directives ESTI avec icônes thématiques, recherche par
mots-clés et lien vers chaque PDF officiel. Un script **Google Apps Script** contrôle
automatiquement le site ESTI **chaque dimanche à midi**, compare les versions et
**republie le site sur GitHub** si une nouvelle version paraît.

## Fichiers

| Fichier      | Rôle |
|--------------|------|
| `index.html` | Le site. Lit ses données depuis `data.json` (repli intégré si absent). |
| `data.json`  | Données live : directives + dates de vérification. **Mis à jour par le script.** |
| `Code.gs`    | Script Google Apps Script : contrôle hebdomadaire + republication GitHub. |

En haut du site s'affichent **« Dernière vérification »** et **« Prochaine vérification »**,
alimentées par `data.json` (`meta.lastCheck` / `meta.nextCheck`).

---

## 1. Publier le site sur GitHub Pages

1. Créez un dépôt GitHub, p. ex. `esti-directives`.
2. Téléversez `index.html` et `data.json` à la racine.
3. **Settings → Pages →** Source = `Deploy from a branch`, branche = `main`, dossier = `/ (root)`.
4. Le site est en ligne sous `https://VOTRE_UTILISATEUR.github.io/esti-directives/`.

## 2. Créer le jeton GitHub (écriture)

1. GitHub → **Settings → Developer settings → Personal access tokens → Fine-grained tokens**.
2. **Repository access** : seulement le dépôt `esti-directives`.
3. **Permissions → Repository → Contents : Read and write**.
4. Générez le jeton et **copiez-le** (il ne s'affiche qu'une fois).
   ⚠️ Ne le collez JAMAIS dans un fichier du dépôt : uniquement dans *Propriétés du script* (étape 3.5).

## 3. Configurer le script Google Apps Script

1. Allez sur <https://script.google.com> → **Nouveau projet**.
2. Collez le contenu de `Code.gs` dans l'éditeur.
3. En haut du fichier, renseignez `CONFIG` :
   ```js
   githubOwner : 'VOTRE_UTILISATEUR_GITHUB',
   githubRepo  : 'esti-directives',
   githubBranch: 'main',
   ```
4. **⚙️ Paramètres du projet → Fuseau horaire → `(GMT+01:00) Zurich`** (important pour « dimanche midi »).
5. **Paramètres du projet → Propriétés du script → Ajouter une propriété** :
   - Nom : `GITHUB_TOKEN`
   - Valeur : *le jeton copié à l'étape 2*
6. Revenez à l'éditeur, sélectionnez la fonction **`installer`** et cliquez **Exécuter**.
   - Autorisez l'accès quand Google le demande.
   - Cela **crée le déclencheur hebdomadaire** (dimanche 12h) **et lance un premier contrôle**.

C'est terminé. Chaque dimanche à midi, le script :

- lit `data.json`,
- télécharge la page ESTI et compare les versions,
- met à jour `data.json` (versions + dates de vérification),
- republie sur GitHub → le site se met à jour tout seul,
- vous envoie un e-mail **uniquement si** une version a changé.

---

## Vérifier / déboguer

Dans l'éditeur Apps Script, vous pouvez exécuter manuellement :

- **`testAnalyse`** — affiche les directives et versions détectées sur la page ESTI (n'écrit rien).
- **`testGithub`** — vérifie l'accès en lecture au dépôt GitHub.
- **`controleHebdomadaire`** — lance un contrôle complet immédiatement.

Les résultats apparaissent dans **Exécutions** / **Journaux** (`Logger.log`).

## Notes

- Le script compare la **version** (code à 4 chiffres du nom de fichier, ex. `0621`) **et** l'URL du PDF.
  Quand l'ESTI publie une nouvelle version, le nom de fichier change → détection automatique.
- Les **annexes** de la directive 235 (`…_anhang…`) sont ignorées ; seule la directive principale est suivie.
- Une **nouvelle directive** détectée est ajoutée automatiquement avec une icône par défaut
  (`document`) et sans mots-clés : complétez alors `icone` et `mots` dans `data.json`
  (icônes disponibles dans `index.html`, objet `ICONES`).
- La directive **508** n'a pas encore de PDF français en ligne : son URL reste vide
  (bouton « Bientôt disponible ») jusqu'à publication par l'ESTI.
- GitHub Pages sert via un CDN : après une mise à jour, comptez quelques minutes de cache.
