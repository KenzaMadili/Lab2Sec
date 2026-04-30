# Rapport LAB 2 : Rooting Android - MADILI Kenza

## Introduction

Ce rapport documente les étapes techniques effectuées pour tenter d'obtenir les privilèges root sur un émulateur Android. Bien que l'accès superutilisateur (`uid=0`) soit obtenu dans le shell, des mécanismes de sécurité comme le verrouillage du bootloader limitent les modifications système permanentes.

## Matériel et Logiciels Utilisés

- **Émulateur :** Pixel 6 
- **Outils :** Android Debug Bridge (ADB)
- **Environnement :** Windows Command Prompt (C:\Users\kenza)

## Étapes de la Procédure

### 1. Initialisation de l'environnement
Lancement de l'appareil virtuel personnalisé.
<img width="703" height="203" alt="Capture d’écran 2026-04-30 125508" src="https://github.com/user-attachments/assets/cb27ea7a-3c97-42a6-9933-00860592a618" />

- **Commande :** `emulator -avd Pixel_6`
- **Configuration RAM :** 3072 MB
- **Logs notables :** Initialisation du backend graphique `gfxstream` et détection du serveur IPv4 sur `192.168.2.1`.

### 2. Vérification de la connectivité ADB
Avant toute manipulation, nous vérifions que l'instance est reconnue par les outils de développement.
<img width="671" height="184" alt="Capture d’écran 2026-04-30 125602" src="https://github.com/user-attachments/assets/662f2aee-f873-4685-aac1-ecc69edf79f5" />

- **Commande :** `adb devices`
- **Statut :** Le daemon ADB démarre avec succès sur le port 5037. L'appareil `emulator-5554` est listé.

### 3. Escalade de privilèges (Shell Root)
Nous vérifions l'identité de l'utilisateur actuel dans le système Android.
<img width="697" height="199" alt="Capture d’écran 2026-04-30 130012" src="https://github.com/user-attachments/assets/4c2dfe3c-ff44-491f-9262-b56f9a4bf5f2" />

- **Commande :** `adb shell id`
- **Résultat :** `uid=0(root) gid=0(root)`. 
- **Analyse :** J'ai obtenu un accès root au niveau du shell, permettant de lire des fichiers protégés mais pas encore de modifier la partition système.

### 4. Analyse des sécurités système (dm-verity)
Pour modifier le système, il est nécessaire de désactiver la vérification d'intégrité au démarrage.


- **Vérification du mode :** `adb shell getprop ro.boot.veritymode` -> renvoie `enforcing`.
- **Tentative de désactivation :** `adb disable-verity`.
- **Résultat :** `Device must be bootloader unlocked`.
- **Analyse :** La sécurité **dm-verity** est active. Le système rejette la modification car le bootloader de l'émulateur est verrouillé, protégeant ainsi la partition `/system`.

## Conclusion

Ce travail  démontre que l'obtention de l'UID 0 (root) dans un terminal ne signifie pas un contrôle total de l'appareil. Sur les versions modernes d'Android, le **verrouillage du bootloader** constitue la barrière principale empêchant l'écriture sur les partitions système, même pour un utilisateur root.

---
*Document généré par MADILI Kenza .
