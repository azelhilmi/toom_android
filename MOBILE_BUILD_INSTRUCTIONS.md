# Toom Android — via Capacitor : mode d'emploi

Ce dépôt contient le code web de Toom **plus** le wrapper Capacitor Android,
pour produire l'APK/AAB à publier sur le Play Store. La version iOS vit dans
le dépôt séparé `toom_ios`.

## Ce qui a été fait

- Capacitor installé (`@capacitor/core`, `@capacitor/cli`, `@capacitor/android`),
  `appId` = `com.toom.app`, `appName` = `Toom`.
- Dossier `android/` (projet Gradle natif complet) avec :
  - permission caméra ajoutée à `AndroidManifest.xml`
  - configuration de signature (`signingConfigs.release`) qui lit les secrets
    via des propriétés Gradle (`-P...`), jamais en clair dans le dépôt.
- Workflow GitHub Actions `.github/workflows/mobile-build.yml` qui build
  l'APK + l'AAB **signés**, prêts pour le Play Store, sur un runner GitHub
  (`ubuntu-latest` — accès réseau complet à Google Maven, contrairement à
  l'environnement où ce code a été préparé).
- Un keystore de signature Android généré (`toom-release.keystore`), valide
  jusqu'en 2054, livré séparément (jamais dans ce dépôt Git).

## À faire une seule fois : pousser ce dépôt

Ce dépôt t'a été fourni sous forme de bundle Git (fichier `.bundle`) car
l'environnement qui l'a préparé n'a pas d'accès en écriture à ton GitHub :

```
git clone toom_android.bundle toom_android
cd toom_android
git remote set-url origin https://github.com/azelhilmi/toom_android.git
git push -u origin main
```

Le keystore (`toom-release.keystore`) et `PASSWORDS-SECRET.txt` sont fournis
**à part**, jamais dans le dépôt. Garde-les en lieu sûr. **Si tu les perds,
tu ne pourras plus jamais publier de mise à jour de l'app sous
`com.toom.app`** — il faudrait recréer une fiche Play Store entièrement
nouvelle.

## À faire une seule fois : configurer les secrets GitHub

Dans le dépôt GitHub `toom_android` → Settings → Secrets and variables →
Actions → New repository secret :

| Nom du secret                    | Valeur                                                                       |
| --------------------------------- | ----------------------------------------------------------------------------- |
| `ANDROID_RELEASE_KEYSTORE_BASE64` | Contenu de `keystore-base64.txt` fourni (une seule ligne, colle-la telle quelle) |
| `ANDROID_RELEASE_STORE_PASSWORD`  | Valeur `STORE_PASSWORD` de `PASSWORDS-SECRET.txt`                             |
| `ANDROID_RELEASE_KEY_ALIAS`       | `toom`                                                                        |
| `ANDROID_RELEASE_KEY_PASSWORD`    | Valeur `KEY_PASSWORD` de `PASSWORDS-SECRET.txt` (= la même que STORE_PASSWORD) |

## Lancer un build

- Automatique : à chaque `git push` sur `main` qui touche le code web,
  `android/` ou la config Capacitor.
- Manuel : onglet **Actions** → "Build Android..." → **Run workflow**.

Résultats dans **Actions** → le run → **Artifacts** :
- `toom-release-apk` : à installer directement sur un téléphone pour tester.
- `toom-release-aab` : à envoyer au Play Console pour publier.

## Publier sur le Play Store (résumé)

1. Compte Google Play Console (frais unique ~25 $) si pas déjà fait.
2. Créer l'application, remplir la fiche (nom, description, captures
   d'écran, icône, politique de confidentialité — obligatoire vu la caméra
   et Firebase).
3. Dans "App signing", laisser Google gérer la clé finale (**Play App
   Signing**, recommandé) : le keystore devient ta clé de "téléversement".
4. Créer une release (piste interne/fermée pour commencer), uploader le
   `.aab` téléchargé depuis GitHub Actions.
5. Remplir le questionnaire de contenu et soumettre à la revue.

## ⚠️ Ce dépôt est une copie du code web, pas une source live

Ce code web a été copié depuis le dépôt principal `azelhilmi/toom` au moment
de la préparation. **Ce dépôt ne se met pas à jour automatiquement** quand tu
modifies le code dans `azelhilmi/toom`. Avant chaque nouvelle publication
Android :
1. Reporte tes changements web (nouvelles fonctionnalités, correctifs) dans
   `src/`, `public/`, etc. de **ce** dépôt — soit en les recopiant depuis
   `azelhilmi/toom`, soit en développant directement ici si tu choisis de
   faire de ce dépôt la source pour les builds mobiles.
2. `npm run build` → `npx cap sync android` → commit → push → le workflow
   build une nouvelle APK/AAB.

L'app native embarque une copie figée du site au moment du build (contrairement
à la PWA web qui affiche toujours la version live) : chaque changement visible
sur `toom.web.app` doit être reporté ici et republié pour atteindre les
utilisateurs de l'app Android installée.

## Développement local (Android Studio)

```
npm run build
npx cap sync android
npx cap open android
```
