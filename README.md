# k8s-training

Mini-kube est une version miniaturisée d'un cluster k8s
Il permet de faire des tests en local
il crée un environnement de dev pour k8s
il ne doit pas être utilisé en prod

La conteunarisation est autorisée sur les VM AWS mais la virtualisation est interdite
Cad sur une VM AWS, il n'est pas permis d'installer (une autre VM ou) un service virtualisé (concrètement, 
dans certains environnements on peut utiliser VirtualBox pour qu'il aille récupérer en ligne une image d'une VM 
sur laquelle k8s est installée ensuite la lancer)
=> pas possible d'utiliser la méthode standard qui permet à minikube de lancer une VM qui contient k8s

Pour contourner le problème, on lance plutôt les containers avec les services qu'il faut dessus,
un container pour kube-controller-manager, un container pour etcd (la bd), un container pour api-server, 
un autre pour kube-scheduler, un pour kube-proxy

wget permet de télécharger en local les binaires sur des sites distants 

kubectl => outil de ligne de commande de k8s

kubelet => service permet d'interpréter les ordres de k8s relatifs au déploiement des appli,
Si on n'a pas kubelet on pourra émettre des ordres mais pas de retour 

et comme avec minikube on n'a pas de worker distant du master, 
je comprends qu'on n'aura qu'un master dans notre config
et kubelet permet de simuler la présence d'un worker au sein du master



****** kubeadm, minikube sont des solutions de training sur k8s ***********
quand on fait => docker ps
=> on voit les containers qui tournent 
et on se rend compte que pour chacun des environnements (kubeadm ou minikube)
ce sont des containers docker qui sont lancés

=> ceci fait encore réaliser que docker est devenu une norme en matière de containerisation 
pour packager un service ou une appli 



*******************************************

un objet est une ressource qu'on peut créer dans k8s.
=> le pod
Un pod contient l'environnement (réseau, stockage, container) qui permet d'exécuter un ou +sieurs container : il peut avoir un volume, un ou +sieurs containers,...

le pod est l'élt que k8s connait,
c'est le + petit elt qu'on peut déployer,
+ petite unité d'exécution de k8s

Tous les container dans un pod partagent un même réseau, les mêmes volumes,  même espace de nom.

Si par exemple, je veux déployer un container,
je dois constituer un pod


=> (objet suivant)

**** replicaset *****
ils permettent d'apporter de la résilience et 
de la scalabilité

en garantissant qu'il y ait toujours un nombre minimal de pods

dès qu'il y a suppression et qu'il se rend compte que le minimum n'est pas atteint, 
il le recrée

on définit un modèle de pods à déployer pour que 
le réplicaset puisse déployer 

scalabilité car grâce au replicaset on 
peut déployer autant de pods qu'on veut
=> il suffit d'augmenter le nombre de replicaset
selon la montée de charge


résilience assimilée (dans cette explication) à 
de la haute dispo i.e. si des pods lâchent, le 
replicaset garde un nombre minimal de pods 


****** deployment ****
permet de piloter les replicasets dans des versions différentes

=> donc grâce au deployment on gère aussi la 
scalabilité


***** TP déploiement 1er pod ***** 
quand on écrit le fichier yaml du pod, 
on marque containerPort: 8080 
et Dirane ajoute que 

(question pour le bootcamp)
=>  ce port de container est exposé dans le cluster 
et pas à l'extérieur ??? ça signifie quoi concrètement ???
le port


*********** replicaset/replica/pod ***************
ce que je comprends 

réplicaset = set de réplica = ensemble de réplica

réplica = réplique d'un même pod

=> réplicaset = (abstraction d'un) ensemble de répliques d'un même pod 
=> le cardinal de ce set est défini dans le manifeste du replicaset
=> il définit dont le nombre de pods (répliques) du set

UN set de réplica permet de gérer les pods de ce set, dans une version précise

************ deployment *************
le deployment c'est l'abstraction qui permet de manager plusieurs set de réplica 
il permet notamment de switcher d'une version à une autre d'un set 

---
quand on crée un deployment via la ligne de commande il n'est pas possible d'indiquer le nombre de réplica
si on veut changer le nombre de replica il faut utiliser une autre commande

kubectl scale --replicas=<nbre-de-replica> deployment/<nom-du-deployment>

nota: Chaque fois qu'on fait un "kubectl get " il faut rajouter l'option "-o wide" pour avoir les détails 

===== mise à jour de l'image dans le deployment (à chaud) ====
=> kubectl set image deployment/<nom-du-deployment>  <nom-du-container>=<nom-de-la-nouvelle-image>


=====  **** Utilisation des manifests **** ======
Création de ressource
=> kubectl apply -f <chemin-fichier.yml>   

Suppression de ressource
=> kubectl delete -f <chemin-fichier.yml>



***********************************************************************************************************************************


Questions
=> Pourquoi "restart=Never" correspond à un pod et sans le mentionner ça correspond à un deployment ???


*********************************************

en production il est recommandé d'utiliser des ressources deployment


- kustomize est très used en entreprise surtout quand on n'a pas beaucoup d'objets
- mais helm est bcp plus used


=> pour l'exam il faut faire les tests kodekloud
et à moins 3 jours de l'exam faire les exams sur killer.sh

=> en recherchant la doc sur kubernetes.io s'il n'y a pas doc dans l'url (kubernetes.io>docs>...) la session est directement fermée 

=> recommandation d'Ulrich => utiliser au max le CLI pour les questions, ça nous fera gagner du temps, et éviter d'être éliminé en recherchant dans la doc 

kubectl api-resources => permet de voir les versions d'objets utilisés notamment la version d'api utilisée

namespace = true => signifie que la ressource peut se retrouver dans un namespace

kubectl api-versions => donne les versions de l'api pour les versions

kubectl explain <objet>  => pour comprendre les manip possibles sur l'objet 

labels & selectors

=> le label c'est l'étiquette 
=> le selector c'est le choix que je fais de prendre tous les objets dont le label correspond à ce que je mentionner

request => quantité de ressource allouée au pod 
limit => limite à ne pas dépasser pour un pod (en cas de pic de fonctionnement)

DaemonSet est souvent utilisé pour maintenir un service fonctionnel en front au niveau de k8s, exemple assurer la continuité d'un service nginx 

Static Pods 
=> on ne passe pas par l'apiserver
=> c'est pas nous qui les créons (ce ne sont pas des ressources qui nous sont affectées)
=> kubelet prend en charge la définition des static pod
=> leur fichier manifest se retrouve dans le répertoire /etc/kubernetes/manifests/


Multiple scheduler
=> faire des déploiements de façon spécifique
pour un déploiement rapide avec une priorisation de l'affection des ressources (criticité des ressources)


devoirs => labs 5 à 7 à réaliser 


*****************************************************************************
************** tp2 ************************
générer un manifest à partir du CLI 

pour un pod 
=> kubectl run <nom-pod> --image=<nom-img> --dry-run=client -o yaml > <nom-fichier.yml>
Nota: C'est bien "run" et pas create

pour un deployment
=> kubectl create deployment <nom-deployment> --image=<nom-img> --dry-run=client -o yaml > <nom-fichier.yml>

pour un namespace
=> kubectl create namespace <nom-namespace> --dry-run=client -o yaml > <nom-fichier.yml>

création d'un pod en précisant namespace, port et label 
=> kubectl run <nom-pod> --image=<nom-img> --dry-run=client  --env="APP_COLOR=blue" --port=<port> --namespace=production --labels="app=web" -o yaml > pod-blue.yml

Nota: Dans nom-img on peut préciser la version si on veut, exemple nginx:1.18 (si on ne précise pas, c'est la "latest" qui est considérée)

pour supprimer un objet
ayant le fichier manifest c'est facile de faire : 
=> kubectl delete -f <nom-fichier.yml> 

si on n'indexe pas le manifest on peut aussi faire:
=> kubectl delete <type_d'objet> <nom-objet>   
ex.=>  kubectl delete pods mon-pod 

pour obtenir le manifest d'un objet existant on peut faire:
=> kubectl get <objet> <nom-objet> -o yaml > <nom-fichier.yml>

pour afficher les objets existants on peut faire
=> kubectl get <type_d'objets> -A (le tiret A permet d'avoir toutes les infos sur l'objet)
ex. => kubectl get rs -A
=> l'option "-o wide" donne les infos détaillées sur l'objet, 
par exemple la version de l'image qui tourne 

Abréviations

deploy = deployment
rs = replicasets
po = pods


****** redirection de ports *******
On peut rediriger le traffic venant de l'extérieur vers un port précis d'un pod 

kubectl port-forward <nom-pod> <port-externe>:<port-du-pod> --address 0.0.0.0

Nota: pour stopper la redirection on arrête juste le prompt, en faisant par exemple Ctrl+C 

A vérifier, mais je le comprends ainsi, cette commande port-forward envoie toutes les requetes venant de l'extérieur sur 
me <port-externe> du node vers le port <port-du-pod> du pod <nom-pod>


*** pour que les replicas soient modifiés progressivement ****
il faut définir la stratégie comme suit:

 strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1     
      maxUnavailable: 1

**** pour l'aide sur une commande il faut juste terminer la commande par --help ********

***** Utilisation de variable d'environnement *********

kubectl run <nom-pod> --image=<nom-img> --env="APP_COLOR=blue"  
Nota: On peut utiliser "\" pour passer à la ligne dans l'écriture des commandes sur le CLI

Une fois le pod créé on peut l'exposer avec la commande => kubectl port-forward

****** Création de deployment sur le CLI *****
kubectl create deployment <nom-deployment> --image=<nom-img>

Cette commande crée un deployment avec un seul replica 

On peut vérifier en faisant par exemple: 
=> kubectl get deployments.<label-du-deployment>

Nota: Avec le CLI on ne peut pas mettre à jour le nombre de réplica 
on utilise le mot-clé "scale"

kubectl scale --replicas=<nbre-de-replica> deployment/<nom-deployment>


********** Changement de l'image à chaud avec le CLI ***************
=> kubectl set image deployment/<nom-deployment> <nom-du-container-dans-le-pod>=<nom-img>

Nota: la valeur nom-du-container-dans-le-pod peut être obtenue en observant la colonne CONTAINERS 
quand on fait la commande "kubectl get deployments.<label-du-deployment> -o wide" ou "kubectl get rs.<label-du-replicaset> -o wide"


Conclusion: On a fait l'équivalent en ligne de commande mais le manifest reste la méthode à privilégier car c'est une bonne pratique
Ca permet de versionner notre infra, de sauvegarder une description de la config déployée et ça facilite le travail en équipe,
la réutilisabilité de notre code par une autre équipe 



*****************  services kubernetes ******************************

+sieurs pods et un seul point d'entrée => utiliser un service
Les services ont des IP. Pour joindre un 

Le service c'est l'elt fédérateur capable d'exposer un ensemble de pods via un seul endpoint(i.e. point d'entrée).
La requete attérit sur le service qui la redirige vers le pod adéquat (il peut avoir suppression de certains pods, indisponibilité d'autres pods à un moment donné...)
Le service (est un objet qui) aide à garantir la haute dispo.

*** service nodeport ***
expose les pods via un port à l'extérieur de la machine (ce port est appelé nodeport). L'user final peut directement les consommer
Les services de type nodeport sont en général > 30 000 et < 32 000 ?? (à vérifier)

3 types de ports dans la définition d'un service nodeport
=> targetPort = port cible à l'intérieur du pod (c'est le port d'écoute du pod comme on connait depuis les ports pour container)


=> port = port d'écoute interne dans le cluster (c'est le port du service; il permet à une ressource interne au cluster de contacter un pod derrière le service)
Par exemple si je veux joindre un service à partir d'une ressource à l'intérieur du cluster (un pod du cluster par exemple)
Je peux utiliser l'IP du service ou son nom et le port "port".

=> nodeport = port à exposer à l'extérieur (directement atteignible par un user à l'extérieur)


selector => désigne les pods fédérés par le service: les pods à sélectionner par le service nodeport
k8s regardera les labels correspondant à la valeur du selector 

**** service clusterIP ****

permettent d'exposer une application à l'intérieur du cluster.


*** service Loadbalancer ****

Utilisation d'un service via un cloud provider.
Pour cela il faut que le cluster soit sur un cloud provider qui supporte la configuration d'un service de type loadbalancer


********************************  namespace ******************************************
séparation/partition

grâce aux namespaces les environnements sont isolés
on limite les ressources consommés par un projet, une équipe 
on limite aussi les droits d'accès user selon l'environnement 


ex. on ne veut pas que les pods et services liés à l'application de comptabilité affecte/impacte les ressources de l'appli dédiée à la pose des congés 

sur k8 on a le namespace default => namespace par défaut

namespace kubesystem => dans lequel k8s crée les ressources liées à son fonctionnement interne (exemple, les pods liés au fonctionnement interne de k8s)

namespace kubepublic => (Tous les pods créés) Toutes les ressources créées dans le namespace kubepublic sont visibles uniquement en lecture seule par tous 
i.e. que personne d'autre que moi ne peut les modifier 

































































































































