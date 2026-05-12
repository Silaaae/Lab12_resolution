# Lab12_resolution

<img width="1475" height="557" alt="image" src="https://github.com/user-attachments/assets/08759cf1-bc69-4217-9897-d30a3d0abc23" />


Lab 11 — Bypass de la Détection de Root Android avec Frida
Cours : Sécurité des applications mobiles
Outil : Frida (Plan B — scripts directs)

Étape 1 — Installation de Frida et vérification de l'environnement
Frida a été installé côté PC via pip, et la connexion avec l'émulateur a été vérifiée via ADB.
Show Image

Étape 2 — Déploiement de frida-server sur l'émulateur
Le binaire frida-server-android-x86_64 (déjà extrait) a été poussé sur l'émulateur via ADB, rendu exécutable, puis lancé en arrière-plan.
bashadb push frida-server.1-android-x86_64 /data/local/tmp/frida-server
adb shell chmod 755 /data/local/tmp/frida-server
adb shell "/data/local/tmp/frida-server &"
Show Image

Étape 3 — Exécution du script de bypass root (Java hooks)
Le script bypass_root.js a été injecté dans l'application cible via Frida. Il neutralise les détections Java classiques : Build.TAGS, File.exists(), Runtime.exec(), et RootBeer.
bashfrida -U -f com.scottyab.rootbeer.sample -l bypass_root.js --no-pause
Les logs confirment que les hooks sont actifs :
Show Image

Étape 4 — Hooks natifs (bypass_native.js)
Un second script bypass_native.js a été ajouté pour bloquer les appels natifs C/C++ (open, openat, access, stat, lstat) sur les chemins suspects comme /system/xbin/su ou /system/bin/busybox.
bashfrida -U -f com.scottyab.rootbeer.sample -l bypass_root.js -l bypass_native.js --no-pause
Show Image

Étape 5 — Anti-détection Frida (optionnel)
Un script supplémentaire anti_frida.js a été utilisé pour masquer la présence de Frida à l'application, en cachant les variables d'environnement liées à Frida et en bloquant les connexions aux ports habituels (27042, 27043) :
javascriptJava.perform(function() {
  // Masquer les variables d'environnement mentionnant FRIDA
  const Sys = Java.use('java.lang.System');
  Sys.getenv.overload('java.lang.String').implementation = function (name) {
    if (name && name.toLowerCase().indexOf('frida') !== -1) {
      console.log('[+] Hiding env var', name);
      return null;
    }
    return this.getenv(name);
  };

  // Bloquer connexions aux ports Frida habituels (27042, 27043)
  const Socket = Java.use('java.net.Socket');
  Socket.connect.overload('java.net.SocketAddress').implementation = function (addr) {
    const s = addr.toString();
    if (s.indexOf(':27042') !== -1 || s.indexOf(':27043') !== -1) {
      console.log('[+] Blocked connect to', s);
      throw new Error('Connection refused');
    }
    return this.connect(addr);
  };
});

Conclusion
Ce lab a permis de mettre en pratique le bypass de détection de root via Frida en mode Plan B (scripts directs). Les hooks Java ont neutralisé les vérifications classiques (Build.TAGS, File.exists, Runtime.exec, RootBeer), les hooks natifs ont bloqué les appels système sur les chemins suspects, et le script anti-Frida a masqué la présence de l'outil à l'application. L'ensemble forme une chaîne de bypass complète et fonctionnelle.
