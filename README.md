= Reconnaissance vocale sur Raspberry Pi =
== En Utilisant Jasper ==
Jasper est un programme qui permet de faire le lien entre reconnaissance vocale, les internets, et de la synthèse vocale<br />
Un scénario type : vous demandez (vocalement) à votre ordinateur de vous lire (avec une voix de synthèse) vos derniers tweets

http://jasperproject.github.io

Jasper a le mérite de fournir une image de carte SD pour Raspberry avec les outils pré-installés, il convient de rajouter les dictionnaires et éléments pour le français, cf [http://doc.ubuntu-fr.org/pocketsphinx tuto pour ubuntu], il faut adapter l'arborescence en fonction de la PI / Jasper :
 pocketsphinx_continuous -dict /usr/local/share/pocketsphinx/model/lm/fr_FR/frenchWords62K.dic -hmm /usr/local/share/pocketsphinx/model/hmm/fr_FR/french_f0/ -lm /usr/local/share/pocketsphinx/model/lm/fr_FR/french3g62K.lm.dmp -adcdev plughw:0,0 -live -verbose yes -samprate 44100

=== Régler les problèmes de carte son externe usb ===
 sudo apt-get install sox
Régler le volume
 alsamixer
 sudo alsactl store
Identifier la carte son
 arecord -l
<syntaxhighlight lang="bash">
**** List of CAPTURE Hardware Devices ****
card 0: Set [C-Media USB Headphone Set], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
</syntaxhighlight>
 aplay -l
<syntaxhighlight lang="bash">
**** List of PLAYBACK Hardware Devices ****
card 0: Set [C-Media USB Headphone Set], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: ALSA [bcm2835 ALSA], device 0: bcm2835 ALSA [bcm2835 ALSA]
  Subdevices: 8/8
  Subdevice #0: subdevice #0
  Subdevice #1: subdevice #1
  Subdevice #2: subdevice #2
  Subdevice #3: subdevice #3
  Subdevice #4: subdevice #4
  Subdevice #5: subdevice #5
  Subdevice #6: subdevice #6
  Subdevice #7: subdevice #7
</syntaxhighlight>
 aplay -L

 cat /proc/asound/cards
 0 [Set            ]: USB-Audio - C-Media USB Headphone Set C-Media USB Headphone Set at usb-bcm2708_usb-1.3, full speed

Gérer les controles
 amixer controls
<syntaxhighlight lang="bash">
numid=4,iface=MIXER,name='Headphone Playback Switch'
numid=5,iface=MIXER,name='Headphone Playback Volume'
numid=2,iface=MIXER,name='Mic Playback Switch'
numid=3,iface=MIXER,name='Mic Playback Volume'
numid=6,iface=MIXER,name='Mic Capture Switch'
numid=7,iface=MIXER,name='Mic Capture Volume'
numid=8,iface=MIXER,name='Auto Gain Control'
numid=1,iface=PCM,name='Playback Channel Map'
</syntaxhighlight>
 amixer cset numid=3 N


In the last command, change N to 1 if you're using analog or 2 if you're using HDMI

Changer l'ordre
 sudo nano /etc/modprobe.d/alsa-base.conf
<syntaxhighlight lang="bash">
# autoloader aliases
install sound-slot-0 /sbin/modprobe snd-card-0
install sound-slot-1 /sbin/modprobe snd-card-1
install sound-slot-2 /sbin/modprobe snd-card-2
install sound-slot-3 /sbin/modprobe snd-card-3
install sound-slot-4 /sbin/modprobe snd-card-4
install sound-slot-5 /sbin/modprobe snd-card-5
install sound-slot-6 /sbin/modprobe snd-card-6
install sound-slot-7 /sbin/modprobe snd-card-7
# Cause optional modules to be loaded above generic modules
install snd /sbin/modprobe --ignore-install snd && { /sbin/modprobe --quiet snd-ioctl32 ; /sbin/m$
install snd-rawmidi /sbin/modprobe --ignore-install snd-rawmidi && { /sbin/modprobe --quiet snd-s$
install snd-emu10k1 /sbin/modprobe --ignore-install snd-emu10k1 && { /sbin/modprobe --quiet snd-e$
# Keep snd-pcsp from beeing loaded as first soundcard
options snd-pcsp index=-2
# Keep snd-usb-audio from beeing loaded as first soundcard
options snd-usb-audio index=0
# Prevent abnormal drivers from grabbing index 0
options bt87x index=-2
options cx88_alsa index=-2
options snd-atiixp-modem index=-2
options snd-intel8x0m index=-2
options snd-via82xx-modem index=-2
options snd_bcm2835 index=-2
</syntaxhighlight>
Tester l'enregistrement
 AUDIODEV=hw:0,0 rec temp.wav
 AUDIODEV=hw:0,0 play temp.wav

= Reconnaissance vocale sur PC Ubuntu =

Cette page explique comment démarrer une application utilisant la reconnaissance vocale avec sphinx, sous linux (testé avec ubuntu 14.04). Nous utilisons sphinx4 (en java), mais il existe aussi pocketsphinx, qui a l'air plus simple (?)

Pour l'instant, cette page décrit l'utilisation de sphinx en langage Java. Il existe aussi une version en C.

''Note: sphinx est aussi (entre autres) le nom d'un générateur de documentation écrit en python, ainsi les recherches à son sujet sur le web renvoient parfois des résultats qui ne sont pas appropriés''

Nous sommes partis de [https://code.google.com/p/voicecmdr/wiki/VoiceRecognitionFR], ceci en est une version peut-être plus digeste. Le rapport de projet étudiant ici: [http://diuf.unifr.ch/diva/courses/multimodal/projects/eif-sphinx/Rapport.pdf] donne d'autres éléments sur l'utilisation de sphinx, en java aussi.

== Installation de sphinx (pour Java)==

* Installer l'environnement java. Sous ubuntu, les paquets "eclipse" et "default-jdk" suffisent. Pour les installer, ouvrir un terminal et taper la commande suivante:
   sudo apt-get install eclipse default-jdk
Ces deux paquets permettent de programmer en java. eclipse est un éditeur de texte adapté à java. default-jdk contient un compilateur java avec les bibliothèques dont il a besoin. eclipse et default-jdk sont conçus pour être utilisés ensemble: eclipse possède un bouton pour exécuter directement le code que vous créerez ou modifierez (en utilisant default-jdk).

* Télécharger les sources de sphinx; la dernière version stable en juillet 2014 est la [http://sourceforge.net/projects/cmusphinx/files/sphinx4/1.0%20beta6/sphinx4-1.0beta6-src.zip/download 4.1.0beta6]. On décompresse l'archive obtenue, et on obtient un dossier sphinx4-1.0beta6. Pour décompresser, on peut utiliser la commande suivante dans un terminal:
   unzip sphinx4-1.0beta6-src.zip

* Une partie de sphinx (la "jsapi") n'est pas sous licence libre, contrairement au reste. Pour installer cette partie, qui nous sera utile par la suite, il faut lancer un script qui va nous demander d'accepter cette licence puis décompresser la "jsapi".
- installer le paquet sharutils qui est nécessaire pour le script, avec la commande (dans un terminal):
   sudo apt-get install sharutils
- lancer le script d'installation de la jsapi. Dans un terminal, depuis le dossier sphinx4-1.0beta6 obtenu en décompressant les sources de sphinx, taper les commandes suivantes:
   cd lib
   sh jsapi.sh
appuyer sur <espace> pour faire défiler le texte de la licence, puis répondre 'y' à la question 'Accept (y/n)?:'
- Si l'on obtient l'erreur suivante, c'est qu'il faut installer le paquet sharutils (cf ci-dessus):
   jsapi.sh: 257: jsapi.sh: uudecode: not found

== Utilisation (pour Java)==
Première étape: tester un code d'exemple présent dans le paquet. La reconnaissance vocale sera en anglais.
Nous allons compiler un des programmes d'exemple ("hello world") depuis eclipse.

* Démarrer eclipse, soit depuis le bureau, soit en tapant la commande 'eclipse' dans un terminal. Si c'est le premier démarrage d'eclipse, celui-ci demande de choisir un répertoire "workspace" où il stockera les projets. Accepter la réponse par défaut (/home/VOTRELOGIN/workspace), et cocher la case "Use this as default".

* Dans l'écran d'accueil d'eclipse, cliquer sur "workbench" (flèche en haut à droite).

* Créer un nouveau projet: sélectionner "Fichier -> Nouveau -> Projet", puis dans la boîte de dialogue choisir "projet java", puis cliquer sur "suivant"

* Comme nom de projet, choisir "helloworld" et cliquer sur suivant

* Passer la dernière boîte de dialogue en cliquant sur "terminer", et si eclipse le demande, accepter d'activer la "perspective java".

* Importer le répertoire contenant l'exemple "helloworld": dans le menu "Fichier -> Importer", "General -> File System". Cliquer sur "parcourir", et sélectionner le ''répertoire'' "sphinx-4.1.0/src/apps/edu/cmu/sphinx/demo/helloworld". Une fois le répertoire sélectionné cocher tous les fichiers qui s'affichent, puis cliquer sur "Terminer".

*Charger les .jar: clic droit sur le projet "helloworld" dans l'arborescence qui apparaît à gauche. 
Dans le menu, choisir "configurer le buildpath", puis dans la boîte de dialogue qui s'ouvre "Java build path", onglet "libraries" puis "external jar".
Sélectionner le répertoire sphinx-4.1.0/lib
Une liste des jar apparaît, il faut tous les sélectionner et valider.
Ensuite, recliquer sur "external jar" et aller chercher "sphinx4-1.0beta6/src/jsapi2/org.jvoicexml.jsapi2.jse.sphinx4/3rdparty/sphinx4/lib/sphinx4.jar", ainsi que "./sphinx4-1.0beta6/src/jsapi2/org.jvoicexml.jsapi2.jse.sphinx4/3rdparty/sphinx4/lib/WSJ_8gau_13dCep_16k_40mel_130Hz_6800Hz.jar"

* Changer la déclaration du package "helloworld": eclipse indique une erreur à la ligne 3 du fichier (une ampoule avec une croix sur fond rouge). Pour voir ce dont il s'agit, laisser la souris au-dessus de la ligne 3 du fichier, une ligne de diagnostic, à propos de la position du package. Parmi les solutions ("quickfixes") proposées en dessous, choisir "déplacer helloworld.java dans edu.cmu.sphinx.demo.helloworld".

* Charger les ressources. À ce stade-là, il ne doit plus y avoir d'erreurs dans la marge de "helloworld.java". On peut donc lancer l'application avec le bouton "run" (dans la barre d'outils). Choisir "as java application". On obtient l'erreur suivante:
  Exception in thread "main" java.lang.NullPointerException
	at edu.cmu.sphinx.util.props.SaxLoader.load(SaxLoader.java:74)
	at edu.cmu.sphinx.util.props.ConfigurationManager.<init>(ConfigurationManager.java:58)
	at edu.cmu.sphinx.demo.helloworld.HelloWorld.main(HelloWorld.java:33)
C'est parce qu'il faut importer les "ressources" dans le projet. Dans le "project explorer", dérouler l'arborescence jusqu'à voir la classe HelloWorld ("HelloWorld" avec un 'c' blanc sur fond vert à gauche). Cliquer avec le bouton droit ''sur la classe'', et choisir "Import…". Choisir "General -> FileSystem", ce qui ouvre une boîte de dialogue dans laquelle on clique sur "browser". On navigue dans l'arborescence, on sélectionne le répertoire "sphinx-4.1.0/src/apps/edu/cmu/sphinx/demo/helloworld". Une fois le répertoire sélectionné cocher tous les fichiers qui s'affichent, puis cliquer sur "Terminer".

* Adapter le fichier de configuration à la position des ressources. Dans le "project explorer", ouvrir le fichier "helloworld.config.xml" qui est ''dans la classe helloWorld''. Le contenu de ce fichier apparaît dans le panneau de droite. Trouver la ligne suivante:
          <property name="fillerPath" value="resource:/WSJ_8gau_13dCep_16k_40mel_130Hz_6800Hz/noisedict"/>
et remplacer "noisedict" par "fillerdict". On obtient:
          <property name="fillerPath" value="resource:/WSJ_8gau_13dCep_16k_40mel_130Hz_6800Hz/dict/fillerdict"/>

* Toujours dans le fichier de configuration, après la ligne 131, ajouter les deux lignes suivantes:
        <property name="modelDefinition" value="etc/WSJ_clean_13dCep_16k_40mel_130Hz_6800Hz.4000.mdef"/>
        <property name="dataLocation" value="cd_continuous_8gau/"/>
        

''à partir d'ici, pas vérifié''

Dans le fichier de configuration, trouver la section concernant le dictionnaire. L'un des items de cette section indique le chemin du dictionnaire (commence par /ressource...)
Il faut remplacer ce chemin par la position où nous avons importé le dictionnaire de notre projet, par exemple ressource:///edu/sphinx/.../helloworld/dict
Faire de même avec la grammaire et le dictionnaire de filler.

On peut maintenant tester le helloworld en cliquant sur Debug (flèche verte). Si la qualité de la reconnaissance est mauvaise, c'est normal.

== Configuration du micro (sous linux) ==
Pour améliorer la reconnaissance, il faut que la carte son échantillonne à 8kHz.
Pour cela, écrire un fichier .asoundrc dans le home, dans lequel on écrira:
"???la formule magique"
Redémarrer la session.
La qualité de la reconnaissance devrait être améliorée.

== Pour le français ==

Modifier le "helloworld" anglais pour l'adapter au français. Pour cela, il faut récupérer les ressources (grammaire, dictionnaire et modèle acoustique), et modifier le fichier de configuration pour que ces nouvelles ressources soient prises en compte.

# Dictionnaire: récupérer le fichier french62K.dic à l'url https://code.google.com/p/voicecmdr/source/browse/branches/workflow/src/sphinx4/french/etc/frenchWords62K.dic, l'importer dans le projet eclipse, puis modifier le chemin du dictionnaire dans la section "dictionnary" du fichier configuration.xml,

# Grammaire: il faut créer une grammaire, sur le modèle de celle qui existe en anglais, l'importer dans le projet eclipse, puis modifier le chemin pour qu'elle soit prise en compte,

# Modèle acoustique: il faut l'importer dans le projet eclipse, puis le pointer depuis le fichier de configuration. On le récupère à l'url https://voicecmdr.googlecode.com/svn/branches/workflow/src/sphinx4/french/model_architecture/french_f0.5725.mdef 

# le fichier dmp: https://voicecmdr.googlecode.com/svn/branches/workflow/src/sphinx4/french/etc/french3g62K.DMP

[[Catégorie:Reconnaissance_vocale]]


source [http://wiki.labomedia.org/index.php/Reconnaissance_vocale_avec_sphinx]
