# Introduction

Ce tutoriel va vous montrer comment installer pas à pas un cluster Kubernetes sur des machines physiques ou virtuelles. La distribution Linux utilisée est Ubuntu 22.04 LTS. La distribution Kubernetes utilisée est microk8s.

# Matériel utilisé

Pour ce tutoriel, nous avons utilisé le matériel suivant :

* 3 mini-PC

    * AMD Ryzen 7 5800U 8 Cores up to 4.4GHz, 
    * 32GB DDR4 
    * 1TB SSD

* Réseau local

    * lab1.local (10.0.0.124)
    * lab2.local (10.0.0.125)
    * lab3.local (10.0.0.123)

Pour ce tutoriel, je vais supposer que vous avez déjà installé Ubuntu 22.04 LTS sur vos machines et que vous pouvez accéder à vos machines via SSH.

J'ai nommé mes machines lab1, lab2 et lab3. Vous pouvez les nommer comme vous le souhaitez. Vous pouvez également utiliser des machines virtuelles.

**Important** : Je me connecte en root sur les machines. Si ce n'est pas votre cas, vous devez utiliser sudo pour les commandes qui le nécessitent.

# Tutoriel





