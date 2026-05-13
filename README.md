# Lab 12 — Bypass de la Détection de Root Android avec Frida

**Cours :** Sécurité des applications mobiles
**Outil :** Frida + frida-server
**Cible :** Émulateur Android x86_64

---

## 1. Preuve d'installation

Avant de commencer le lab, l'environnement a été vérifié. Frida a été installé via pip et les versions côté PC ont été confirmées avec `frida --version` et `python -c "import frida; print(frida.__version__)"`. La commande `adb devices` confirme que l'émulateur Android est bien reconnu et en état `device`.

<img width="1042" height="142" alt="image" src="https://github.com/user-attachments/assets/3684132d-4707-46a9-b162-5548921d6080" />

---

## 2. Déploiement et visibilité de frida-server

Le binaire `frida-server` a été pushé sur l'émulateur dans `/data/local/tmp/`, rendu exécutable avec `chmod 755`, puis lancé en arrière-plan. La commande `frida-ps -Uai` a ensuite été exécutée pour confirmer que le serveur tourne correctement et lister les applications disponibles sur l'émulateur.

<img width="1467" height="382" alt="image" src="https://github.com/user-attachments/assets/017fb76f-11cd-4633-bd72-a4b7ec26e5cf" />

<img width="1022" height="187" alt="image" src="https://github.com/user-attachments/assets/23febc0c-3614-4959-8dbd-b00f9cbb3443" />

---

## 3. Bypass avec Medusa

Medusa a été utilisé pour injecter automatiquement un module de bypass root sur l'application cible. La commande `--spawn` a été utilisée pour démarrer l'application directement sous instrumentation, avec le module `root-bypass` activé. Les logs de Medusa confirment que les hooks ont bien été installés et que les vérifications de root ont été neutralisées. On observe dans les captures l'état avant (root détecté) et après (bypass actif) le lancement via Medusa.

<img width="1057" height="316" alt="image" src="https://github.com/user-attachments/assets/0885e07c-982b-464b-bfae-8a5df51ba008" />

<img width="935" height="581" alt="image" src="https://github.com/user-attachments/assets/81deafe9-bd2d-44c1-9705-2c8f6f0787b6" />

<img width="1145" height="297" alt="image" src="https://github.com/user-attachments/assets/e1508ae3-e4ee-4a6d-95e1-864e8bb35781" />

---

## 4. Plan B — Frida pur (bypass_root.js + bypass_native.js)

En complément de Medusa, les scripts Frida manuels ont également été testés. Le script `bypass_root.js` installe des hooks Java sur `Build.TAGS`, `File.exists()`, `Runtime.exec()` et `RootBeer.isRooted()` pour neutraliser toutes les vérifications courantes côté Java. Le script `bypass_native.js` ajoute des hooks natifs sur les appels système `open`, `openat`, `access`, `stat` et `lstat` pour bloquer les tentatives de détection au niveau C/C++.

Les logs `[+]` dans le terminal confirment que chaque hook est actif. L'application, qui affichait précédemment une alerte de root, passe en état « Not rooted » après injection des scripts.

<img width="731" height="621" alt="image" src="https://github.com/user-attachments/assets/fd9c4e7b-f035-479f-93ab-bed647a74830" />

<img width="822" height="301" alt="image" src="https://github.com/user-attachments/assets/3a7a957e-254b-4d6f-93e1-83377ea537c4" />

---

## Conclusion

Ce lab a permis de mettre en pratique deux approches complémentaires pour contourner la détection de root sur Android. Medusa offre une solution rapide et modulaire grâce à ses modules préconfigurés, tandis que Frida pur permet un contrôle granulaire via des scripts personnalisés ciblant aussi bien la couche Java que la couche native. Les deux approches ont produit le même résultat : l'application ne détecte plus l'environnement rooté, ce qui démontre l'efficacité de l'instrumentation dynamique dans un contexte d'analyse de sécurité mobile.
