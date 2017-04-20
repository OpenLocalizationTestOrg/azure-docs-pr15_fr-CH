<properties
        pageTitle="Ajouter un utilisateur à une VM Linux sur Azure | Microsoft Azure"
        description="Ajouter un utilisateur à une VM Linux sur Azure."
        services="virtual-machines-linux"
        documentationCenter=""
        authors="vlivech"
        manager="timlt"
        editor=""
        tags="azure-resource-manager"
/>

<tags
        ms.service="virtual-machines-linux"
        ms.workload="infrastructure-services"
        ms.tgt_pltfrm="vm-linux"
        ms.devlang="na"
        ms.topic="article"
        ms.date="08/18/2016"
        ms.author="v-livech"
/>

# <a name="add-a-user-to-an-azure-vm"></a>Ajouter un utilisateur à un ordinateur virtuel d’Azure

Une des premières tâches sur n’importe quel VM Linux nouvelle consiste à créer un nouvel utilisateur.  Dans cet article, nous guider dans la création d’un compte d’utilisateur sudo, le mot de passe, ajout de clés publiques SSH et enfin `visudo` pour autoriser sudo sans mot de passe.

Les composants requis sont : [un compte Azure](https://azure.microsoft.com/pricing/free-trial/), [SSH les clés publique et privée](virtual-machines-linux-mac-create-ssh-keys.md), un groupe de ressources Azure et la CLI Azure installés et de passer au mode d’Azure le Gestionnaire de ressources à l’aide `azure config mode arm`.

## <a name="quick-commands"></a>Commandes rapides

```bash
# Add a new user on RedHat family distros
sudo useradd -G wheel exampleUser

# Add a new user on Debian family distros
sudo useradd -G sudo exampleUser

# Set a password
sudo passwd exampleUser
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully

# Copy the SSH Public Key to the new user
ssh-copy-id -i ~/.ssh/id_rsa exampleuser@exampleserver

# Change sudoers to allow no password
# Execute visudo as root to edit the /etc/sudoers file
visudo

# On RedHat family distros uncomment this line:
## Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL

# to this
## Same thing without a password
%wheel        ALL=(ALL)       NOPASSWD: ALL

# On Debian family distros change this line:
# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL

# to this
%sudo   ALL=(ALL) NOPASSWD:ALL

# Verify everything
# Verify the SSH keys & User account
bill@slackware$ ssh -i ~/.ssh/id_rsa exampleuser@exampleserver

# Verify sudo access
sudo top
```

## <a name="detailed-walkthrough"></a>Procédure pas à pas détaillé

### <a name="introduction"></a>Introduction

Une de la tâche avec un nouveau serveur de première et la plus courante consiste à ajouter un compte d’utilisateur.  Connexions d’accès racine doivent être désactivées et le compte racine lui-même ne doit pas être utilisé avec votre serveur Linux, sudo uniquement.  En donnant une racine de l’utilisateur des privilèges de remontée des problèmes à l’aide de sudo il la façon correcte de l’administration et l’utilisation de Linux.

À l’aide de la commande `useradd` nous avons ajouté des comptes d’utilisateurs de Linux VM.  Exécution de `useradd` modifie `/etc/passwd`, `/etc/shadow`, `/etc/group`, et `/etc/gshadow`.  Nous avons ajouté un indicateur de ligne de commande pour le `useradd` commande à ajouter le nouvel utilisateur au groupe approprié sudo sous Linux.  Même thou `useradd` crée une entrée dans `/etc/passwd` il ne donne pas le nouveau compte d’utilisateur un mot de passe.  Nous allons créer un mot de passe initial pour le nouvel utilisateur à l’aide de la simple `passwd` commande.  La dernière étape consiste à modifier les règles sudo pour permettre à cet utilisateur d’exécuter des commandes avec des privilèges sudo sans avoir à entrer un mot de passe pour chaque commande.  Connexion à l’aide de la clé privée, nous supposons que compte d’utilisateur est à l’abri des acteurs de mauvais et souhaitez autoriser l’accès sudo sans mot de passe.  

### <a name="adding-a-single-sudo-user-to-an-azure-vm"></a>Ajouter un utilisateur unique sudo un Azure VM

Connectez-vous à l’Azure VM à l’aide de clés SSH.  Si vous n'avez pas le programme d’installation public clé accès SSH, complète cet article première [utilisation authentification par clé publique avec Azure](http://link.to/article).  

Le `useradd` commande réalise les tâches suivantes :

- créer un nouveau compte d’utilisateur
- créer un nouveau groupe d’utilisateurs avec le même nom
- Ajouter une entrée à blanc pour`/etc/passwd`
- Ajouter une entrée à blanc pour`/etc/gpasswd`

Le `-G` indicateur de ligne de commande ajoute le nouveau compte d’utilisateur au groupe approprié de Linux donnant le nouveau compte d’utilisateur des privilèges de remontée de racine.

#### <a name="add-the-user"></a>Ajouter l’utilisateur

```bash
# On RedHat family distros
sudo useradd -G wheel exampleUser

# On Debian family distros
sudo useradd -G sudo exampleUser
```

#### <a name="set-a-password"></a>Définir un mot de passe

Le `useradd` commande crée l’utilisateur et ajoute une entrée à la fois à `/etc/passwd` et `/etc/gpasswd` mais ne définit ne pas le mot de passe.  Le mot de passe est ajouté à l’entrée à l’aide de la `passwd` commande.

```bash
sudo passwd exampleUser
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
```

Nous disposons désormais d’un utilisateur avec des privilèges sudo sur le serveur.

### <a name="add-your-ssh-public-key-to-the-new-user-account"></a>Ajouter votre clé publique SSH vers le nouveau compte d’utilisateur

À partir de votre ordinateur, vous devez utiliser le `ssh-copy-id` avec le nouveau mot de passe.

```bash
ssh-copy-id -i ~/.ssh/id_rsa exampleuser@exampleserver
```

### <a name="using-visudo-to-allow-sudo-usage-without-a-password"></a>Pour permettre l’utilisation de sudo sans mot de passe à l’aide de visudo

À l’aide de `visudo` pour modifier la `/etc/sudoers` fichier ajoute plusieurs couches de protection contre une modification incorrecte de ce fichier important.  Lors de l’exécution `visudo`, le `/etc/sudoers` fichier est verrouillé afin d’aucun autre utilisateur ne peut apporter des modifications pendant qu’il est activement en cours de modification.  Le `/etc/sudoers` fichier est également activée pour les erreurs en `visudo` lorsque vous essayez d’enregistrer ou de quitter afin que vous ne peut pas enregistrer un fichier sudoers rompue.

Nous avons déjà des utilisateurs dans le groupe par défaut pour l’accès de sudo.  Nous allons maintenant activer ces groupes à utiliser sudo sans mot de passe.

```bash
# Execute visudo as root to edit the /etc/sudoers file
visudo

# On RedHat family distros uncomment this line:
## Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL

# to this
## Same thing without a password
%wheel        ALL=(ALL)       NOPASSWD: ALL

# On Debian family distros change this line:
# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL

# to this
%sudo   ALL=(ALL) NOPASSWD:ALL
```

### <a name="verify-the-user-ssh-keys-and-sudo"></a>Vérifiez que l’utilisateur ssh clés et sudo

```bash
# Verify the SSH keys & User account
ssh -i ~/.ssh/id_rsa exampleuser@exampleserver

# Verify sudo access
sudo top
```
