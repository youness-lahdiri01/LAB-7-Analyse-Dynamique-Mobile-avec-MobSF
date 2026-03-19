# LAB 7 : Analyse Dynamique Mobile avec MobSF

## Objectifs du lab

* Comprendre l’analyse dynamique (runtime) d’une application Android avec MobSF
* Configurer un émulateur Android propre sans Play Store
* Installer et utiliser MobSF via Docker
* Tester une application vulnérable (DIVA)
* Identifier des vulnérabilités en temps réel (logs, stockage, intents, réseau)

---

## Prérequis

* Android Studio + SDK Platform Tools (ADB)
* Docker Desktop
* Git
* 8 Go RAM minimum
* Processeur 64 bits

---

## Étape 1 : Création de l’émulateur

1. Ouvrir Android Studio
2. Aller dans AVD Manager
3. Créer un Virtual Device (Pixel 5 recommandé)
4. Choisir une image Android sans Google Play
5. API recommandée : 29 ou 30
6. Nommer : MobSF_DIVA_API_30

---

## Étape 2 : Cloner MobSF

```bash
git clone https://github.com/MobSF/Mobile-Security-Framework-MobSF.git
cd Mobile-Security-Framework-MobSF
```

---

## Étape 3 : Lancer l’émulateur

### Sous Windows (PowerShell)

```powershell
& "C:\Users\youne\AppData\Local\Android\Sdk\emulator\emulator.exe" -avd "Pixel 5"
```

Vérification :

```bash
adb devices
```

---

## Étape 4 : Lancer MobSF avec Docker

```powershell
docker run -it --rm -p 8000:8000 opensecurity/mobile-security-framework-mobsf:latest
```

Accès :

```
http://127.0.0.1:8000
```

---
<img width="1918" height="1017" alt="Screenshot 2026-03-19 161902" src="https://github.com/user-attachments/assets/73146349-7105-4a6a-b06f-896c65d9ce00" />

## Étape 5 : Télécharger DIVA

```bash
git clone https://github.com/xAltmime/diva-apk-file.git
```

---

## Étape 6 : Installer DIVA

### Avec Genymotion

```powershell
adb -s 192.168.56.102:5555 install -r "C:\Users\youne\Desktop\diva-apk-file\DivaApplication.apk"
```

---

## Étape 7 : Analyse statique

1. Aller dans MobSF
2. Upload de `DivaApplication.apk`
3. Lancer l’analyse

---
<img width="1917" height="974" alt="Screenshot 2026-03-19 162315" src="https://github.com/user-attachments/assets/b749d9f1-9527-4b78-a040-8e37c1fcc49f" />

## Étape 8 : Analyse dynamique

1. Cliquer sur "Start Dynamic Analyzer"
2. Lancer l’application DIVA
3. Interagir avec les challenges

---
<img width="1913" height="972" alt="Screenshot 2026-03-19 172948" src="https://github.com/user-attachments/assets/61c319b1-c521-4059-b48e-e073beb74b1c" />

## Étape 9 : Analyse des logs

```powershell
adb logcat -c
adb logcat | findstr /i "diva"
```

Observation :

* Logs système
* Activités de l’application

---
<img width="1906" height="977" alt="Screenshot 2026-03-19 174345" src="https://github.com/user-attachments/assets/17a935c9-7e8d-4351-9539-00b67b5d0e12" />

## Étape 10 : Analyse du stockage

Accès au système :

```bash
adb shell
cd /data/data/jakhar.aseem.diva
```

### Exploration :

```bash
ls
cd databases
sqlite3 divanotes.db
```

### Requêtes :

```sql
.tables
SELECT * FROM notes;
```
<img width="1064" height="451" alt="Screenshot 2026-03-19 180738" src="https://github.com/user-attachments/assets/857c3ff6-5c97-4ae0-a862-2e4a130629dc" />

Résultat :

* Données stockées en clair
* Aucune protection

---

## Étape 11 : Analyse Shared Preferences

```bash
cd /data/data/jakhar.aseem.diva/shared_prefs
ls
cat *.xml
```

---

## Étape 12 : Tests avancés

### Activity Tester (MobSF)

* Tester les activités internes

### Exported Activity Tester

* Vérifier les accès externes

### . BYPASS AVEC FRIDA
Objectif

Intercepter ou modifier le comportement de l’application en runtime

Code Frida

Dans MobSF → Frida → Spawn & Inject :

Java.perform(function() {
    var log = Java.use("android.util.Log");

    log.d.implementation = function(tag, msg) {
        send("Intercepted Log: " + msg);
        return this.d(tag, msg);
    };
});
Résultat

interception des logs en live

possibilité de modifier le comportement
<img width="1916" height="864" alt="image" src="https://github.com/user-attachments/assets/17abb9b8-aa8a-4e42-8626-0690d184ee74" />

Conclusion

L’application est vulnérable à l’instrumentation dynamique
### BYPASS ROOT DETECTION
Objectif

Contourner les protections anti-root

Code Frida
Java.perform(function() {
    var Runtime = Java.use("java.lang.Runtime");

    Runtime.exec.overload('java.lang.String').implementation = function(cmd) {
        if (cmd.indexOf("su") !== -1) {
            send("Bypass root detection");
            return null;
        }
        return this.exec(cmd);
    };
});
Résultat

détection root contournée
<img width="785" height="435" alt="image" src="https://github.com/user-attachments/assets/71aea76e-2a9f-4984-bdcf-28bed6826f76" />

Conclusion

Absence de protection efficace contre les environnements rootés
## Étape 13 : Vulnérabilités identifiées

### Insecure Data Storage

* Données stockées en clair dans SQLite

### Insecure Logging

* Logs système observés (bruit principalement)

### Weak Access Control

* Activités accessibles

### Absence de chiffrement

* Données sensibles non protégées

---

## Étape 14 : Conclusion

L’analyse dynamique avec MobSF a permis d’observer le comportement réel de l’application DIVA.

Les principales vulnérabilités identifiées sont :

* Stockage non sécurisé des données
* Absence de chiffrement
* Failles d’accès aux composants internes

Cette analyse démontre l’importance de sécuriser les données et les interactions runtime dans les applications mobiles.

---

## Commandes utiles

```bash
adb devices
adb logcat
adb shell
sqlite3 divanotes.db
SELECT * FROM notes;
```

---

## Résultat final

* Environnement fonctionnel
* Analyse dynamique réalisée
* Vulnérabilités identifiées et documentées
