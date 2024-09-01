# PROJET_FINAL_Jenkins
Projet final jenkins et ansible avec AWS
-----------------------------------------
Jeremie EDOUARD & Tomi VAILLANT
-----------------------------------------
### Compte Rendu de Pratique : Jenkins et Ansible avec AWS

#### Contexte du Projet

Dans ce projet, nous avons configuré un environnement automatisé pour la gestion des ressources AWS, en utilisant **Jenkins** pour orchestrer les déploiements et les tâches, et **Ansible** pour gérer les configurations des ressources AWS. Nous avons installé Jenkins et Ansible sur deux machines virtuelles distinctes, hébergées localement sur un PC, avec une connexion à nos comptes AWS via l'interface en ligne de commande (CLI).

Tout les fichier Yamel utiliser pour se projet on été crée dans GitHub. Jenkins les a ensuite récupérer à partir de se dépot.

#### Objectifs

- **Automatiser** la gestion des ressources AWS.
- **Orchestrer** les tâches via Jenkins.
- **Gérer** les configurations des ressources AWS avec Ansible.
- **Assurer** la sécurité et l'efficacité du système.

### Étapes du Projet

#### 1. Préparation des Ressources

Pour ce projet, nous avons configuré deux instances EC2 pour héberger Jenkins et Ansible. Les ressources AWS ont été configurées via des scripts Ansible, et la sécurité des données sensibles a été assurée par AWS KMS.

#### 2. Création des Jobs Jenkins

Tout nos obs Jenkins ont été configuré de la même manière, le depot Github étaitn indiqué puis dans la partie "script shell" nous avons mis en place une suite de commande suivant un schéma similaire pour chaque job.
Ce schéma consiste à envoyé sur la machine Ansible les fichier yml souhaitez via SCP, puis l'on déplace dans un répertoire "playbook" le fichier .yml et pour finir nous lançons la commande "ansible-playbook"

![alt text](https://github.com/tropizz/PROJET_FINAL_Jenkins/blob/main/screenshots/configuration_git.png)

Dans certains scripts shell nous avons du directement, introduis des D et Mot de passe en clair.... Nous avons utiliser cette solutions pour palier à certains bug et soucis de mise en place.
Nous restosn conscien qu'il est préférable de ne pas laisser d'informations d'identificatuion en clair.

##### a) Job pour lister les services AWS

Le job "list_service" est le premier script exécuté dans la pipeline. Il utilise un playbook Ansible pour lister les ressources AWS, notamment les instances EC2, les buckets S3, et les groupes de sécurité. 

Ce job est crucial car il permet de vérifier l'état actuel des ressources avant de procéder à d'autres opérations, comme le déploiement ou la suppression de ressources. Le playbook Ansible utilise les modules `aws_ec2_info` et `aws_s3_bucket_info` pour récupérer ces informations. 

fichier .yml utilisé : /lister_ressources/liste_ressources.yml

![alt text](https://github.com/tropizz/PROJET_FINAL_Jenkins/blob/main/screenshots/script_job_list_service.png)


##### b) Job pour créer de nouvelles ressources

Un playbook Ansible a été développé pour déployer des ressources AWS, incluant la création d'instances EC2 et de buckets S3. Ce playbook est exécuté via le job "deploy_bdd" dans Jenkins.

![alt text](https://github.com/tropizz/PROJET_FINAL_Jenkins/blob/main/screenshots/script_job_deploy_bdd.png)
 

##### c) Job pour modifier ou supprimer des ressources

Des scripts Ansible ont été développés pour mettre à jour ou supprimer des ressources AWS, tels que la suppression d'instances EC2. Ces scripts sont exécutés via le job "delete_ressources" dans Jenkins.

![alt text](https://github.com/tropizz/PROJET_FINAL_Jenkins/blob/main/screenshots/script_job_delete_ressources.png)

#### 3. Intégration d'Ansible

##### a) Développement de playbooks Ansible

Les playbooks Ansible ont été créés pour gérer les configurations des instances EC2, comme l'installation de MySQL et d'Apache.

Pour cette utilisation de Ansible nous avons du configurer d'autre paramètre supplémentaire directmeent dans les fichiers de configurations de la VM Ansible.
Fichier "ansible.cfg" :
![alt text](https://github.com/tropizz/PROJET_FINAL_Jenkins/blob/main/screenshots/fichier_ansible_cfg.png)

Fichier "hosts" :

![alt text](https://github.com/tropizz/PROJET_FINAL_Jenkins/blob/main/screenshots/fichier_ansible_hosts.png)

Le fichier host permet de regrouper un ou plusieurs serveur par utilisté pour ensuite déployer certains servies uniquement sur certain "hosts" machine.
Par exemple déployer le service apache2 sur uniquement les serveur web !

##### b) Automatisation via Jenkins

Jenkins a été configuré pour exécuter ces playbooks Ansible automatiquement selon des déclencheurs spécifiques, garantissant une gestion continue et automatisée des configurations.

#### 4. Présentation des Résultats

##### a) Documentation Technique

La méthodologie suivie, les configurations des jobs Jenkins, des playbooks Ansible, et les ressources AWS ont été documentées en détail, incluant les mesures de sécurité mises en place.

##### b) Présentation des Dashboards

Les dashboards Jenkins ont été configurés pour présenter de manière claire et intuitive l'état des jobs, permettant une visualisation rapide des résultats.

![alt text](https://github.com/tropizz/PROJET_FINAL_Jenkins/blob/main/screenshots/dashboard_jenkins.png)

![alt text](https://github.com/tropizz/PROJET_FINAL_Jenkins/blob/main/screenshots/repertoire_jenkins_workspace.png)


### Analyse du Script d'Exécution (Pipeline Jenkins)

Voici un extrait du dernier script Jenkins Pipeline utilisé pour orchestrer les différents jobs dans le pipeline :

```groovy
pipeline {
    agent any

    stages {
        stage('List Services') {
            steps {
                script {
                    // Exécute le job list_service
                    build job: 'list_service', wait: true
                }
            }
        }

        stage('Git Operations') {
            steps {
                script {
                    // Exécute le job git
                    build job: 'git', wait: true
                }
            }
        }

        stage('Deploy Service') {
            steps {
                script {
                    // Exécute le job deploy_service
                    build job: 'deploy_service', wait: true
                }
            }
        }

        stage('Deploy Database') {
            steps {
                script {
                    // Exécute le job deploy_bdd
                    build job: 'deploy_bdd', wait: true
                }
            }
        }

        stage('Delete Resources') {
            steps {
                script {
                    // Exécute le job delete_ressources
                    build job: 'delete_ressources', wait: true
                }
            }
        }
    }

    post {
        success {
            echo 'La pipeline s\'est exécutée avec succès.'
        }
        failure {
            echo 'La pipeline a échoué.'
        }
        always {
            echo 'La pipeline est terminée.'
        }
    }
}
```

### Explication de la Pipeline

- **`stage('List Services')`** : Ce stage exécute le job `list_service`, qui liste les services AWS pour s'assurer que les ressources actuelles sont correctement identifiées avant de procéder à d'autres opérations.

- **`stage('Git Operations')`** : Ce stage gère les opérations Git, comme le pull des dernières modifications depuis le dépôt.

- **`stage('Deploy Service')` et `stage('Deploy Database')`** : Ces stages déploient les services et les bases de données en utilisant Ansible via les jobs `deploy_service` et `deploy_bdd`.

- **`stage('Delete Resources')`** : Ce stage supprime les ressources non nécessaires en exécutant le job `delete_ressources`.

![alt text](https://github.com/tropizz/PROJET_FINAL_Jenkins/blob/main/screenshots/output_console_pipeline.png)

---
