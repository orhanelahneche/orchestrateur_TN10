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

Maintenant que nous avons installé notre distribution il faut installer docker. Entrez la commande suivante pour ouvrir un terminal de commande dans votre nouvelle distribution :

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
$ k3d cluster create <cluster_name> --k3s-arg '--kubelet-arg=--system-reserved=cpu=2000m,memory=1Gi,ephemeral-storage=1Gi@server:*'
```

Le drapeau *--k3s-arg* vous permet de passer des arguments supplémentaires à la commande k3s server qui est exécutée sur chaque nœud du cluster k3d. Le drapeau *--kubelet-arg* nous sert à passer des arguments au  kubelet du noeuds filtré par *@server:*, dans notre cas on réserve une partie des ressources du nœud au fonctionnement du système.

## Installation kubectl
Avant de passer à la suite il faut installer le cli kubernetes, kubectl, pour interagir avec notre cluster. Pour ceci, on va télécharger la dernière release kubectl puis utiliser le binaire pour en faire une commande éxecutable.

```
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
$ chmod +x ./kubectl
$ sudo mv ./kubectl /usr/bin/kubectl
```

Vous pouvez vous assurer que l'installation s'est bein déroulée avec la commande *kubectl version*

## Installation de argo
On crée un namespace "argo" dans le cluster précédemment créé et on y applique une une configuration pour éxécuter le server et le controler argo:

```
$ kubectl create namespace argo
$ kubectl apply -n argo -f https://github.com/argoproj/argo-workflows/releases/download/v<<ARGO_WORKFLOWS_VERSION>>/install.yaml
```

Remplacez *<<ARGO_WORKFLOWS_VERSION>>* par le numéro de la version que vous voulez utiliser. Vous pouvez trouver la dernière release sur la [release page](https://github.com/argoproj/argo-workflows/releases/latest). 

Dans un deuxième temps on va modifier le mode d'authentification du server pour éviter d'avoir besoin de se login :

```
$ kubectl patch deployment \
  argo-server \
  --namespace argo \
  --type='json' \
  -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/args", "value": [
  "server",
  "--auth-mode=server"
]}]'
```

Dans un troisième temps nous installons le cli argo :

```
# Download the binary
$ curl -sLO https://github.com/argoproj/argo-workflows/releases/download/v3.4.5/argo-linux-amd64.gz

# Unzip
$ gunzip argo-linux-amd64.gz

# Make binary executable
$ chmod +x argo-linux-amd64

# Move binary to path
$ mv ./argo-linux-amd64 /usr/local/bin/argo

# Test installation
$ argo version
```

Et enfin nous rendons le serveur accesibles depuis l'extérieur : 

```
$ kubectl -n argo port-forward deployment/argo-server 2746:2746
```
