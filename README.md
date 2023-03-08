# Install_kub_cluster K3D

## Installation distribution Ubuntu20.04

Cette installation peut se faire grâce au store windows ou grâce à une commande dans le powershell:

```
$ wsl --install -d Ubuntu-20.04
```

Ensuite on set la version de wsl à 2 : 

```
$ wsl --set-default-version 2
```

Maintenant que nous avons installé notre distribution il faut installé docker. Entrez la commande suivante pour ouvrir un terminal de commande dans votre nouvelle distribution :

```
$ wsl -d Ubuntu-20.04
```

Si vous avez téléchargé l'application via le microsoft store, vous devriez être en mesure d'ouvrir un terminale en éxécutant l'application téléchargée.


## Création d'un utilisateur docker
Docker n'accepte pas que le root l'utilise, il faut donc créer un utilisateur et lui donner les privilèges d'un super administrateur:

```
$ sudo adduser -m <user_name>
$ sudo usermod -aG sudo <user_name>
```

## Installation de docker dans la distribution

On éxecute les commande suivante dans le terminal Ubuntu :

```
$ sudo apt-get update -y
$ sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
 
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo apt-key fingerprint 0EBFCD88
 
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
$ sudo apt-get update -y
$ sudo apt-get install -y docker-ce
```
*Ces commandes ont été trouvées à l'adresse : https://blog.lecacheur.com/2021/11/23/docker-sous-windows-wsl-2-sans-docker-desktop/*

Une fois cette étape effectuée on ajoute l'utilisateur au groupe docker:

```
$ usermod -aG docker <user_name>
```

Vous pouvez vérifier que cette étape a bien fonctionné avec la commande :

```
$ grep docker /etc/group
```
## Installation k3d et création d'un premier cluster
Maintenant, nous allons procéder à l'installation de k3d, une distribution kubernetes légère fonctionant à l'aide d'un driver docker. Pour ceci, on installe le script d'installation fourni par k3d puis on l'éxécute :

```
$ curl -s https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash
```

Une fois k3d installé, on peut créer notre premier cluster : 

```
