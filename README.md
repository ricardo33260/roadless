# ROADLESS

Jeu de course arcade en pseudo-3D (esquive, flips, turbo, checkpoints, garage, radio de 10 morceaux).
Le jeu est une page web (`www/`) embarquée dans une application Android (`android/`).

```
roadless/
├─ www/                  le jeu (index.html) + assets/ (musiques MP3 d'origine, image d'accueil)
├─ android/              projet Android (WebView plein écran) -> APK + AAB
├─ store/                icône 512 px et image de présentation 1024x500 pour Google Play
├─ .github/workflows/    android.yml (APK + AAB) et pages.yml (jeu jouable sur GitHub Pages)
├─ CREDITS.md            crédits et licence de la musique (CC BY 4.0, à conserver)
└─ MUSIC-LICENCE.txt     licence fournie avec l'album
```

## 1. Mettre sur GitHub

```bash
cd roadless
git init -b main
git add .
git commit -m "ROADLESS"
git remote add origin https://github.com/<vous>/roadless.git
git push -u origin main
```

Les MP3 d'origine pèsent environ 64 Mo au total, chacun reste sous la limite GitHub de 100 Mo.

## 2. Jouer dans le navigateur

- En local : `cd www && python3 -m http.server 8000` puis ouvrir http://localhost:8000
  (ne pas ouvrir `index.html` en double-clic : les musiques ont besoin d'un serveur).
- En ligne : dans GitHub, *Settings > Pages > Source : GitHub Actions*.
  Le workflow `pages.yml` publie alors le jeu à chaque push.

## 3. Construire l'APK et l'AAB (automatique)

Chaque push sur `main` lance le workflow **Android (APK + AAB)**. Dans l'onglet *Actions*,
ouvrez l'exécution puis téléchargez :

- `roadless-debug-apk` : à installer sur un téléphone pour tester.
- `roadless-release-aab` : le bundle pour Google Play.

### Signer l'AAB (obligatoire pour Google Play)

Sans clé, l'AAB est produit **non signé**. Créez une clé d'envoi (une seule fois, à conserver précieusement) :

```bash
keytool -genkeypair -v -keystore upload.jks -alias roadless -keyalg RSA -keysize 2048 -validity 10000
base64 -w0 upload.jks > keystore.b64        # macOS : base64 -i upload.jks -o keystore.b64
```

Puis, dans GitHub (*Settings > Secrets and variables > Actions*), créez 4 secrets :

| Secret | Valeur |
|---|---|
| `KEYSTORE_BASE64` | contenu de `keystore.b64` |
| `KEYSTORE_PASSWORD` | mot de passe du keystore |
| `KEY_ALIAS` | `roadless` |
| `KEY_PASSWORD` | mot de passe de la clé |

Relancez le workflow : l'AAB est maintenant signé. Le numéro de version (`versionCode`) suit
automatiquement le numéro d'exécution du workflow, il augmente donc à chaque build.
N'ajoutez jamais `upload.jks` au dépôt (déjà exclu par `.gitignore`).

### Construire en local (Android Studio)

Ouvrir le dossier `android/` dans Android Studio, puis *Build > Generate Signed App Bundle*.
Ou en ligne de commande, avec Gradle 8.9 et un JDK 17 : `cd android && gradle bundleRelease`.

## 4. Google Play

- Identifiant de l'application : `com.roadless.game`.
- Images : `store/icon-512.png` et `store/feature-graphic-1024x500.png`.
- Ajoutez dans la description de la fiche : « Musique : Neon Husky, album Contribution Run, CC BY 4.0 ».
- Taille : l'AAB fait environ 70 Mo, bien sous les limites de Google Play.

## Fonctionnement

- Aucun accès réseau : tout est dans l'application (pas de permission Internet). Seule la vibration est demandée.
- Les scores, le garage et les réglages sont enregistrés sur l'appareil (stockage local).
- Les musiques sont lues en streaming depuis les MP3 d'origine, sans recompression.

## Limites connues

Ce projet a été préparé sans pouvoir être compilé ni testé sur un appareil Android
(pas de SDK Android dans l'environnement de création). Le premier build GitHub peut demander
un petit ajustement ; si le workflow échoue, ouvrez le journal de l'étape *Construire* et
envoyez-le pour correction.
