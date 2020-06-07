# Astuces Windows 10

### Activation pavé numérique au démarrage

1. Lancer Regedit en mode administrateur.

2. Modifier la valeur courante de la clef de registre **HKEY_CURRENT_USER/ControlPanel/Keyboard/InitialKeyboardIndicators** par la valeur 2.

3. Modifier la valeur courante de la clef de registre **HKEY_USERS/.DEFAULT/ControlPanel/Keyboard/InitialKeyboardIndicators** par la valeur 2.

4. Désactiver le démarrage rapide : **Paramètres** -> **Système** -> **Alimentation et mise en veille** -> **Paramètres d'alimentation supplémentaires** -> **Choisir l'action du bouton d'alimentation** -> **Modifier des paramètres actuellement non disponibles** et décocher l'option **Activer le démarrage rapide (recommandé)**.

5. Redémarrer le PC. 

### Désactiver Cortana

* Windows 10 Pro

1. Lancer gpedit.msc en mode administrateur (depuis la barre de recherche du menu démarrer).

2. Aller à **Computer Configuration** -> **Administrative Templates** -> **Windows Components** -> **Search**.

3. Double cliquer sur **Allow Cortana** (volet de droite).

4. Choisir **Disabled** et valider.

5. Redémarrer le PC.

* Windows 10 Famille

1. Lancer Regedit en mode administrateur.

2. Naviguer dans la base de registre **HKEY_LOCAL_MACHINE/SOFTWARE/Policies/Microsoft**.

3. Créer une nouvelle clef nommée **Windows Search** (clic droit sur "Windows" puis "Nouveau" et "Clé" dans le menu déroulant).

4. Créer une sous clef nommée **AllowCortana** (clic droit sur "Windows Search" puis "Nouveau" et "Valeur DWORD 32 bits").

5. Modifier la valeur de la sous clef par **0** (clic droit sur "AllowCortana" puis "Modifier").

6. Redémarrer le PC.

### Convertir un disque dur MBR en GPT (installation Windows en mode UEFI)

1. Lancer l'invite de commande (MAJ + F10 depuis le programme d'installation).

2. Taper **diskpart** pour ouvrir l'utilitaire de gestion de disque dur puis **list disk** pour afficher tous les disques détectés (un numéro est affecté à chaque disque). 

3. Sélectionner le disque à partitionner avec **select disk X** (où X est le numéro du disque, taper **detail disk** pour plus d'information sur le disque).

4. Enfin procéder à la conversion avec **clean** puis **convert gpt**. Quitter avec **exit**.
