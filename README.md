# 📱 Lab Rooting Android — Analyse de Sécurité Mobile

> **Environnement :** Genymotion AVD · Android 11 API 30  
> **Auteur :** MADILI Kenza  


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
<img width="970" height="164" alt="Capture d&#39;écran 2026-04-29 132704" src="https://github.com/user-attachments/assets/51c7f483-b8c3-4939-817c-501bc032e299" />


> Résultat : `192.168.56.102:5555` reconnu et actif.

---

### Étape 2 — Activation du mode Root

```bash
adb root
```
<img width="889" height="81" alt="Capture d&#39;écran 2026-04-29 132729" src="https://github.com/user-attachments/assets/1854111f-9537-4f90-9452-fc7240763e6d" />


> Relance le démon ADB avec les privilèges super-utilisateur.

---

### Étape 3 — Remontage de la partition système en R/W

```bash
adb remount
```


> Monte `/system` en lecture/écriture.

---

### Étape 4 — Vérification des privilèges root

```bash
adb shell id
# → uid=0(root) gid=0(root)
```


---

### Étape 5 — Test de la commande `su`

```bash
adb shell "su -c id"
# → uid=0(root)
```


---

### Étape 6 — Vérification de la version Android

```bash
adb shell getprop ro.build.version.release
# → 11
```


---

### Étape 7 — Capture des logs système

```bash
adb logcat -d > logcat_root_check.txt
```


---

### Étape 8 — Installation de F-Droid via ADB

```bash
adb install F-Droid.apk
```


---

### Étape 9 — Identification de l'activité principale

```bash
adb shell cmd package resolve-activity --brief org.fdroid.fdroid
```

---

### Étape 10 — Lancement de l'application

```bash
adb shell am start -n org.fdroid.fdroid/.views.main.MainActivity
```


---

### Étape 11 — Accès aux données privées avec root

```bash
adb shell ls -la /data/data/org.fdroid.fdroid/
```


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
