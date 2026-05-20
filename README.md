# lab12-securite
          Contexte et objectifs du contournement
Ce laboratoire se place dans une optique défensive : comprendre comment les applications Android détectent un appareil rooté, puis apprendre à neutraliser ces vérifications par des hooks dynamiques, uniquement dans un cadre pédagogique et autorisé. L’environnement technique repose sur Frida (instrumentation), Medusa (automatisation), un émulateur Android ou un appareil physique, et l’application vulnérable Uncrackable Level 3. L’apprenant doit être capable d’identifier les contrôles Java et natifs, d’utiliser les hooks, de diagnostiquer les échecs de bypass et de vérifier le bon fonctionnement des modifications.

          Préparation de l’environnement et démarrage de Frida-Server
Avant toute manipulation, on vérifie les versions de Frida, de son bindings Python, et la présence de l’appareil via adb devices. L’architecture CPU de l’émulateur est obtenue (adb shell getprop ro.product.cpu.abi) pour choisir le bon binaire frida-server. Celui-ci est poussé dans /data/local/tmp/, rendu exécutable (chmod 755), puis lancé avec l’option -l 0.0.0.0 pour écouter sur toutes les interfaces. Les ports 27042 et 27043 sont redirigés via adb forward, et la connexion est validée par frida-ps -Uai.

           Mécanismes de détection root ciblés
Les applications utilisent plusieurs techniques :

Java : inspection de Build.TAGS (présence de « test-keys »), recherche des binaires su ou busybox via Runtime.exec(), utilisation de bibliothèques comme RootBeer.

Natif (C/C++) : appels système open(), access(), stat(), lecture de /proc/mounts.

Anti-Frida : détection des ports Frida, recherche de chaînes « frida » ou de débogueurs.
Le laboratoire montre comment ces vérifications peuvent être bloquées.

        État initial et déclenchement du bypass
Sans intervention, Uncrackable Level 3 affiche un message d’avertissement (« Rooting or tampering detected… ») puis se ferme automatiquement. L’étape suivante consiste à lancer Medusa avec la commande :
python medusa.py --usb --spawn com.example.uncrackable3 --module root-bypass.
Medusa récupère automatiquement des informations (Android 11, API 30, type d’appareil) et applique un ensemble de hooks prédéfinis.

Hooks injectés et résultat
Les hooks utilisés sont :

Forcer Build.TAGS à renvoyer "release-keys".

Intercepter File.exists() pour tous les chemins sensibles (/system/bin/su, /system/xbin/su, etc.) et retourner false.

Bloquer Runtime.exec() sur les commandes su, busybox, which su.
Une fois les hooks actifs, la détection root est contournée : l’application ne se ferme plus et fonctionne normalement. Les logs Frida permettent de visualiser les appels interceptés et de confirmer le bon chargement des hooks.

Dépannage et bonnes pratiques
Si le root est toujours détecté, il faut suspecter du code natif (C/C++). La solution est d’activer les Native Hooks de Medusa pour intercepter fopen(), access() et les contrôles JNI. Par ailleurs, une incompatibilité de versions entre le client Frida (sur le PC) et frida-server (sur l’appareil) est une cause fréquente d’échec : il est impératif que les numéros de version correspondent exactement.

Cadre éthique et références
Ce type de manipulation n’est autorisé que sur des applications de test, des APK pédagogiques, des émulateurs personnels ou des environnements explicitement consentis. Il est interdit de l’utiliser pour analyser des applications tierces réelles ou à des fins malveillantes. Le laboratoire se réfère aux standards OWASP (MASVS, MASTG) pour ancrer la démarche dans une approche de sécurité offensive autorisée. Des captures d’écran illustrent la détection initiale, la console Frida/Medusa, le bypass confirmé et l’application fonctionnant sur l’émulateur.
