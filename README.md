# 📱 Lab Rooting Android — Analyse de Sécurité Mobile

> **Environnement :** Genymotion AVD · Android 11 API 30  
> **Auteure :** kenza  
> **Date :** 26 Avril 2026  
> **Statut Root :** `uid=0(root)` — Confirmé ✅  
> **SELinux :** Permissive

---

## ⚠️ Avertissement légal

> Toutes les manipulations ont été réalisées sur un **émulateur Genymotion dédié**.  
> Aucune donnée réelle n'a été utilisée.  
> Réseau : **Host-Only** — Environnement totalement isolé.

---

## 🎯 Objectif

Ce laboratoire vise à comprendre les implications du **rooting sur Android** :

- Comprendre ce que le rooting modifie (sandbox, SELinux, Verified Boot)
- Vérifier l'état du système et les privilèges obtenus via ADB
- Remettre l'environnement à zéro après le lab (wipe AVD, traçabilité complète)

---

## 🧪 Environnement de Test

| Composant | Détail |
|-----------|--------|
| PC Hôte | Windows 10/11 |
| Émulateur | Genymotion Personal Edition |
| Version Android | Android 11 (API 30) |
| Adresse AVD | `192.168.56.102:5555` |
| Outil | ADB (Android Debug Bridge) |
| Réseau | Host-Only (isolé) |
| SELinux | Mode Permissive |
| Application testée | F-Droid (`org.fdroid.fdroid`) |

---

## 🔧 Déroulement du Lab

### Étape 1 — Vérification de la connexion à l'AVD

```bash
adb devices
```
<img width="4792" height="2030" alt="image-enhancer-1777239815127" src="https://github.com/user-attachments/assets/013575b0-4a85-4727-9692-2fe770a90fb7" />

> Résultat : `192.168.56.102:5555` reconnu et actif.

---

### Étape 2 — Activation du mode Root

```bash
adb root
```
<img width="1188" height="402" alt="image-enhancer-1777239875192" src="https://github.com/user-attachments/assets/8da67e77-69a3-4a0a-be29-14259bf5168a" />

> Relance le démon ADB avec les privilèges super-utilisateur.

---

### Étape 3 — Remontage de la partition système en R/W

```bash
adb remount
```
<img width="1132" height="398" alt="image-enhancer-1777239915852" src="https://github.com/user-attachments/assets/2911fa00-ce3a-4243-a779-45c181e941b4" />

> Monte `/system` en lecture/écriture.

---

### Étape 4 — Vérification des privilèges root

```bash
adb shell id
# → uid=0(root) gid=0(root)
```
<img width="1990" height="536" alt="image-enhancer-1777239966851" src="https://github.com/user-attachments/assets/d2f8e8ad-4514-4cbf-8f01-c5f530e3bde9" />

---

### Étape 5 — Test de la commande `su`

```bash
adb shell "su -c id"
# → uid=0(root)
```
<img width="1364" height="432" alt="image-enhancer-1777240011361" src="https://github.com/user-attachments/assets/51b1e378-43da-47ce-8225-cd17235ab293" />

---

### Étape 6 — Vérification de la version Android

```bash
adb shell getprop ro.build.version.release
# → 11
```
<img width="1768" height="348" alt="image-enhancer-1777240155906" src="https://github.com/user-attachments/assets/3efe6f98-7fbf-48ca-82ce-96b093bfb497" />

---

### Étape 7 — Capture des logs système

```bash
adb logcat -d > logcat_root_check.txt
```
<img width="2426" height="626" alt="image-enhancer-1777240204257" src="https://github.com/user-attachments/assets/8378a758-786c-41de-a8d0-bbdb2eee149c" />

---

### Étape 8 — Installation de F-Droid via ADB

```bash
adb install F-Droid.apk
```
<img width="1460" height="518" alt="image-enhancer-1777240246872" src="https://github.com/user-attachments/assets/6f447ec0-46c4-4e0f-8a07-27a745dda611" />

---

### Étape 9 — Identification de l'activité principale

```bash
adb shell cmd package resolve-activity --brief org.fdroid.fdroid
```
<img width="2386" height="492" alt="image-enhancer-1777240291516" src="https://github.com/user-attachments/assets/99535f01-7270-49e2-99fb-b4b110b29aa1" />

---

### Étape 10 — Lancement de l'application

```bash
adb shell am start -n org.fdroid.fdroid/.views.main.MainActivity
```
<img width="2566" height="514" alt="image-enhancer-1777240375020" src="https://github.com/user-attachments/assets/9ae8b886-0e76-4839-acbb-9acf422de9e4" />

---

### Étape 11 — Accès aux données privées avec root

```bash
adb shell ls -la /data/data/org.fdroid.fdroid/
```
<img width="1480" height="658" alt="image-enhancer-1777240544269" src="https://github.com/user-attachments/assets/0d95163b-a44c-4a04-8104-0f5b882e17a1" />

> **⚠️ Impact Sécurité :** L'accès à `/data/data/<package>/` avec root expose toutes les données privées : bases SQLite, tokens, clés API, préférences partagées.

---

## 📋 Récapitulatif des Commandes

| Commande | Objectif | Résultat |
|----------|----------|----------|
| `adb devices` | Vérifier connexion | device actif |
| `adb root` | Activer mode root | already running as root |
| `adb remount` | Remonter /system | remount succeeded |
| `adb shell id` | Vérifier uid | `uid=0(root)` |
| `adb shell "su -c id"` | Tester su | `uid=0(root)` |
| `adb shell getprop ro.build.version.release` | Version Android | 11 |
| `adb logcat -d > logcat_root_check.txt` | Capturer logs | fichier créé |
| `adb install F-Droid.apk` | Installer APK | Success |
| `adb shell am start -n ...` | Lancer app | Status: ok |
| `adb shell ls -la /data/data/...` | Lister données app | Accès confirmé |

---

## 🔐 Sécurité Android — Concepts Clés

- **Sandboxing** : chaque application est isolée via un UID unique. Le rooting casse cette isolation.
- **Modèle de permissions** : le root bypasse entièrement le système de permissions utilisateur.
- **Verified Boot (AVB 2.0)** : vérifie l'intégrité des partitions au démarrage via une chaîne de confiance.



> *Verified Boot n'est pas disponible sur l'émulateur Genymotion.*

---

## ⚡ Matrice des Risques

| # | Risque | Description |
|---|--------|-------------|
| 1 | Intégrité non garantie | Conclusions potentiellement fausses si env. compromis |
| 2 | Surface d'attaque accrue | Exposition à des menaces externes via root |
| 3 | Données sensibles exposées | Fuite potentielle via `/data/data/` |
| 4 | Instabilité système | Tests non reproductibles sur env. rooté |
| 5 | Mélange comptes perso/test | Fuite d'informations personnelles |
| 6 | Mauvais nettoyage | Persistance de données sensibles post-session |
| 7 | Réseau non isolé | Effets involontaires sur le réseau hôte |
| 8 | Traçabilité insuffisante | Impossibilité d'auditer les actions réalisées |

---

## 🛡️ Mesures Défensives Appliquées

| # | Mesure | Description |
|---|--------|-------------|
| 1 | Réseau isolé (Host-Only) | Empêche les communications non contrôlées |
| 2 | Données fictives uniquement | Aucun risque de fuite de données réelles |
| 3 | AVD dédié | Émulateur exclusivement réservé aux tests |
| 4 | Snapshots / Wipe | Aucune trace laissée après la session |
| 5 | Journal de configuration | Reproductibilité et auditabilité |
| 6 | Aucun compte personnel | Pas de mélange de données perso/test |
| 7 | Contrôle strict des APK | Limite les risques d'installation non maîtrisée |
| 8 | Horodatage et captures | Traçabilité complète de chaque étape |

---

## 📚 Référentiel OWASP MAS

### MASVS — Exigences applicables

| Exigence | Description |
|----------|-------------|
| MASVS-STORAGE-1 | Les données sensibles doivent être stockées avec chiffrement (AES-256 minimum) |
| MASVS-NETWORK-1 | Les communications doivent utiliser TLS avec vérification stricte des certificats |

### MASTG — Tests suggérés

| Test | Procédure |
|------|-----------|
| MASTG-TEST-1 | Vérifier les SharedPreferences dans `/data/data/[package]/shared_prefs/` — données non chiffrées ? |
| MASTG-TEST-2 | Analyser logcat pour détecter des fuites d'informations sensibles en clair |

---

## 🔄 Reset de l'Environnement

En fin de session, l'AVD est systématiquement remis à zéro :

```bash
adb emu avd stop
adb emu avd wipe-data
```

> **Alternative :** Device Manager Genymotion → clic droit → *Wipe Data*

> ✅ **Reset effectué** — L'AVD a été wipé en fin de session. F-Droid et tous les artefacts ont été supprimés. L'émulateur affiche l'écran de configuration initial (état usine).

<img width="2056" height="1328" alt="image-enhancer-1777240506161" src="https://github.com/user-attachments/assets/5425b367-2987-4f6f-98fd-58f939563bcf" />

---

## ✅ Checklist Finale

**DÉBUT de session :**
- [x] Périmètre écrit et validé
- [x] AVD neuf démarré (état usine)
- [x] Application F-Droid installée via `adb install`
- [x] 3 scénarios notés (lancer app, naviguer paramètres, quitter)
- [x] Version Android notée : **11**

**FIN de session :**
- [x] Aucun compte personnel utilisé
- [x] Toutes les commandes documentées avec captures
- [x] Logs capturés (`logcat_root_check.txt`)
- [x] Reset AVD effectué (wipe-data)
- [x] Rapport rédigé et archivé

---

## 🧾 Fiche Environnement Complète

| Champ | Valeur |
|-------|--------|
| Date | 26/04/2026 |
| Auteure | kenza |
| Support | Genymotion AVD |
| Version Android | 11 API 30 |
| Application testée | F-Droid (`org.fdroid.fdroid`) |
| Scénarios testés | 3 — lancer app, naviguer paramètres, quitter |
| Rooting effectué | OUI — `uid=0(root)` confirmé |
| État Verified Boot | Non disponible (émulateur) |
| État SELinux | Permissive |
| Preuves | Logs + captures d'écran |

---

## 🏁 Conclusion

Le rooting de l'AVD Genymotion Android 11 a été réalisé avec succès. Ce lab a permis de démontrer :

- Comment obtenir les privilèges root sur un AVD via ADB
- Quelles commandes permettent de vérifier l'état du système (`id`, `getprop`, `ls -la`)
- L'impact du rooting sur les mécanismes de protection Android
- L'importance de la traçabilité, de l'isolation réseau et du reset en fin de session
- Les risques associés et les mesures défensives à mettre en place

---

## 🔗 Ressources

- [F-Droid](https://f-droid.org/)
- [OWASP MASVS](https://mas.owasp.org/MASVS/)
- [OWASP MASTG](https://mas.owasp.org/MASTG/)
- [Android Security](https://source.android.com/docs/security)

---

*Lab réalisé par **MADILI Kenza** — Genymotion Android 11 API 30*
