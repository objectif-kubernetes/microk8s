# Introduction

Ce tutoriel va vous montrer comment installer pas à pas un cluster Kubernetes sur des machines physiques ou virtuelles. La distribution Linux utilisée est Ubuntu 22.04 LTS. La distribution Kubernetes utilisée est microk8s.

Site de microk8s : https://microk8s.io/

# Matériel utilisé

Pour ce tutoriel, nous avons utilisé le matériel suivant :

* 3 mini-PC

    * AMD Ryzen 7 5800U 8 Cores up to 4.4GHz, 
    * 32GB DDR4 
    * 1TB SSD

J'ai acheté les 3 PCs sur le site de [Geekbuying](https://www.geekbuying.com/item/T-bao-MN58U-Mini-PC-5800U-32GB-DDR4-1TB-SSD-EU-520707.html)


* Réseau local

    * lab1.local (10.0.0.124)
    * lab2.local (10.0.0.125)
    * lab3.local (10.0.0.123)

Pour ce tutoriel, je vais supposer que vous avez déjà installé Ubuntu 22.04 LTS sur vos machines et que vous pouvez accéder à vos machines via SSH.

J'ai nommé mes machines lab1, lab2 et lab3. Vous pouvez les nommer comme vous le souhaitez. Vous pouvez également utiliser des machines virtuelles.

**Important** : Je me connecte en *root* sur les machines. Si ce n'est pas votre cas, vous devez utiliser *sudo* pour les commandes qui le nécessitent.

# Tutoriel

## Installation de microk8s

### Installation de microk8s sur les 3 machines

```bash
snap install microk8s --classic
```

Une fois installé, vous pouvez vérifier que microk8s est bien installé en exécutant la commande suivante :

```bash
microk8s status --wait-ready
```

## CLI microk8s

microk8s est livré avec une CLI qui vous permet de gérer votre cluster Kubernetes. Cette CLI est appelée microk8s.

microk8s vous permet de lancer des commandes kubectl directement.

```bash
microk8s kubectl get nodes
```

### Ajout des machines au cluster

Sur la machine lab1, exécutez la commande suivante :

```bash
microk8s add-node
```

Cette commande va vous afficher une commande à exécuter sur les autres machines pour les ajouter au cluster.

```bash 
microk8s join
```

Exécutez cette commande sur les machines lab2 et lab3.

### Vérification du cluster

Sur la machine lab1, exécutez la commande suivante :

```bash
microk8s kubectl get nodes
```

Vous devriez voir les 3 machines dans le cluster.

## Les addons

microk8s est livré avec des addons. Les addons sont des applications qui peuvent être installées sur votre cluster Kubernetes.

### Liste des addons

Pour lister les addons disponibles, vous pouvez aller sur le site officiel de microk8s : https://microk8s.io/docs/addons

Vous remarquerez que les addons sont classés en 3 catégories :

* Core addons
* Community addons
* Disabled addons

### Installation d'un addon


Pour installer un addon, vous pouvez utiliser la commande suivante :

```bash
microk8s enable <addon>
```

Par exemple, pour installer l'addon *dns* :

```bash
microk8s enable dns
```

### Désinstallation d'un addon

Pour désinstaller un addon, vous pouvez utiliser la commande suivante :

```bash
microk8s disable <addon>
```

Par exemple, pour désinstaller l'addon *dns* :

```bash
microk8s disable dns
```

## Installation des addons

Nous allons installer les addons suivants :

* rbac
* community
* metallb
* traefik

#### rbac

```bash
microk8s enable rbac
```

#### community

```bash
microk8s enable community
```

#### metallb

```bash
microk8s enable metallb:10.0.0.150-10.0.0.200
```

#### traefik

```bash
microk8s enable traefik
```

## Vérification des addons

### Vérification de l'addon traefik

```bash
microk8s kubectl get service -n traefik traefik
```

On va obtennir l'adresse IP du service traefik.

```bash
NAME      TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
traefik   LoadBalancer   10.152.183.47   10.0.0.150    80:31541/TCP,443:32594/TCP   20s
```

On peut maintenant lancer un curl sur l'adresse IP du service traefik.

```bash
curl 10.0.0.150
```

On obtient le résultat suivant :

```bash
404 page not found
```

Qui montre que le service traefik est bien installé et fonctionnel.

## Import de kubeconfig

Pour pouvoir utiliser kubectl sur votre machine, vous devez importer le fichier kubeconfig.

```bash
microk8s config > kubeconfig.yml
```

Une fois le fichier kubeconfig.yml généré, vous devez le copier sur votre machine (en scp par exemple).

Le plugin konfig de kubectl permet d'importer un fichier kubeconfig. Ce plugin peut être obtenu à l'aide de krew.

https://krew.sigs.k8s.io/


Maintenant à l'aide du plugin [konfig](https://github.com/corneliusweig/konfig/tree/master) de kubectl, vous pouvez importer le fichier kubeconfig.

```bash
kubectl konfig import --save kubeconfig.yml
```

Une fois importé, vous pouvez vérifier que le fichier kubeconfig est bien importé.

```bash
kubectl ctx
```

et sélectionner le contexte microk8s.

```bash
kubectl ctx microk8s
```

Vous pouvez maintenant utiliser kubectl sur votre machine.

```bash
kubectl get nodes
```

## Démonstration

Le fichier [test.yml](test.yml) contient tout ce qu'il faut pour déployer notre service whoami sur notre cluster.

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: whoami

spec:
  ports:
    - protocol: TCP
      name: web
      port: 80
  selector:
    app: whoami

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traefik-ingress-controller

---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: whoami
  labels:
    app: whoami

spec:
  replicas: 2
  selector:
    matchLabels:
      app: whoami
  template:
    metadata:
      labels:
        app: whoami
    spec:
      containers:
        - name: whoami
          image: traefik/whoami
          ports:
            - name: web
              containerPort: 80
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: simpleingressroute
spec:
  entryPoints:
    - web
  routes:
    - match: Host(`lb.local`)
      kind: Rule
      services:
        - name: whoami
          port: 80
```

vous pouvez déployer ce fichier sur votre cluster.

```bash
kubectl apply -f test.yml
```

Une fois déployé, vous pouvez vérifier que le service est bien déployé.

```bash
kubectl get service
```

et vous pouvez vérifier que le service est bien accessible.

```bash
curl lb.local
```

après avoir ajouté lb.local dans votre fichier hosts.

```bash
lb.local 10.0.0.150
```

Vous recevrez le résultat suivant :

```bash
Hostname: whoami-8c9864b56-m9fq7
IP: 127.0.0.1
IP: ::1
IP: 10.1.197.66
IP: fe80::6462:68ff:fe56:d793
RemoteAddr: 10.1.197.65:48788
GET / HTTP/1.1
Host: lb.local
User-Agent: curl/8.1.2
Accept: */*
Accept-Encoding: gzip
X-Forwarded-For: 10.1.127.128
X-Forwarded-Host: lb.local
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: traefik-857c7dbc5d-xwmhz
X-Real-Ip: 10.1.127.128
```


# Conclusion

Nous avons vu comment installer un cluster Kubernetes avec microk8s. Nous avons vu comment installer des addons et comment déployer une application sur notre cluster.

# Liens

* https://www.youtube.com/watch?v=s8R_q60pZkU
* https://microk8s.io




