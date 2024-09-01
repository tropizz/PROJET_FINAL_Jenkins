# PROJET_FINAL_Jenkins
Projet final jenkins et ansible avec AWS
-----------------------------------------
Jeremie EDOUARD & Tomi VAILLANT
----------------------------------------

### Compte Rendu de Pratique : Jenkins et Ansible avec AWS

#### Contexte du Projet

Dans ce projet, nous avons configuré un environnement automatisé pour la gestion des ressources AWS, en utilisant **Jenkins** pour orchestrer les déploiements et les tâches, et **Ansible** pour gérer les configurations des ressources AWS. Nous avons installé Jenkins et Ansible sur deux machines virtuelles distinctes, hébergées localement sur un PC, avec une connexion à nos comptes AWS via l'interface en ligne de commande (CLI).

Tous les fichiers YAML utilisés pour ce projet ont été créés dans GitHub. Jenkins les a ensuite récupérés à partir de ce dépôt.

#### Objectifs

- **Automatiser** la gestion des ressources AWS.
- **Orchestrer** les tâches via Jenkins.
- **Gérer** les configurations des ressources AWS avec Ansible.
- **Assurer** la sécurité et l'efficacité du système.

### Étapes du Projet

#### 1. Préparation des Ressources

Pour ce projet, nous avons configuré deux VM localement sur un de nos poste (VM-ANSIBLE & VM-JENKIS) . Chaqu'une d'elle hébergeant respectivement son service. Les ressources AWS ont été configurées via des scripts Ansible, et la sécurité des données sensibles a été assurée par AWS. La VM-Ansible communiquais avec le cloud AWS via le Amazon CLI.

#### 2. Création des Jobs Jenkins

Tous nos jobs Jenkins ont été configurés de la même manière. Le dépôt GitHub était indiqué, puis dans la partie "script shell", nous avons mis en place une suite de commandes suivant un schéma similaire pour chaque job. Ce schéma consiste à envoyer les fichiers YAML souhaités sur la machine Ansible via SCP, à les déplacer dans un répertoire "playbook", puis à lancer la commande `ansible-playbook`.

![alt text](https://github.com/tropizz/PROJET_FINAL_Jenkins/blob/main/screenshots/configuration_git.png)

Dans certains scripts shell, nous avons dû introduire directement des identifiants et des mots de passe en clair. Nous avons utilisé cette solution pour pallier certains bugs et problèmes de mise en place. Nous restons conscients qu'il est préférable de ne pas laisser d'informations d'identification en clair.

##### a) Job pour lister les services AWS

Le job "list_service" est le premier script exécuté dans le pipeline. Il utilise un playbook Ansible pour lister les ressources AWS, notamment les instances EC2, les buckets S3, et les groupes de sécurité.

Ce job est crucial car il permet de vérifier l'état actuel des ressources avant de procéder à d'autres opérations, comme le déploiement ou la suppression de ressources. Le playbook Ansible utilise les modules `aws_ec2_info` et `aws_s3_bucket_info` pour récupérer ces informations.

Fichier .yml utilisé : /lister_ressources/liste_ressources.yml

![alt text](https://github.com/tropizz/PROJET_FINAL_Jenkins/blob/main/screenshots/script_job_list_service.png)

##### b) Job pour créer de nouvelles ressources

Nous avons développé un playbook Ansible pour déployer des ressources AWS, incluant la création d'instances EC2 et de buckets S3. Ce playbook est exécuté via le job "deploy_bdd" dans Jenkins.

Fichier .yml utilisé : creer_ressources/deploy_ressources.yml

![alt text](https://github.com/tropizz/PROJET_FINAL_Jenkins/blob/main/screenshots/script_job_deploy_bdd.png)

##### c) Job pour modifier ou supprimer des ressources

Nous avons développé des scripts Ansible pour mettre à jour ou supprimer des ressources AWS, tels que la suppression d'instances EC2. Ces scripts sont exécutés via le job "delete_ressources" dans Jenkins.

Fichier .yml utilisé : suppr_ressources/suppr_ressources.yml

![alt text](https://github.com/tropizz/PROJET_FINAL_Jenkins/blob/main/screenshots/script_job_delete_ressources.png)

#### 3. Intégration d'Ansible

##### a) Développement de playbooks Ansible

Nous avons créé des playbooks Ansible pour gérer les configurations des instances EC2, comme l'installation de MySQL et d'Apache.

Fichier .yml utilisé : config_instances/install_mysql.yml & config_instances/install_apache2.yml

Pour cette utilisation d'Ansible, nous avons dû configurer d'autres paramètres supplémentaires directement dans les fichiers de configuration de la VM Ansible.

Fichier "ansible.cfg" :  
![alt text](https://github.com/tropizz/PROJET_FINAL_Jenkins/blob/main/screenshots/fichier_ansible_cfg.png)

Fichier "hosts" :  
![alt text](https://github.com/tropizz/PROJET_FINAL_Jenkins/blob/main/screenshots/fichier_ansible_hosts.png)

Le fichier `hosts` permet de regrouper un ou plusieurs serveurs par utilisateur pour ensuite déployer certains services uniquement sur certains "hosts" machines. Par exemple, déployer le service Apache2 uniquement sur les serveurs web !

##### b) Automatisation via Jenkins

Nous avons configuré Jenkins pour exécuter ces playbooks Ansible automatiquement selon des déclencheurs spécifiques, garantissant une gestion continue et automatisée des configurations.

#### 4. Présentation des Résultats

##### a) Documentation Technique

Nous avons documenté en détail la méthodologie suivie, les configurations des jobs Jenkins, des playbooks Ansible, et les ressources AWS, incluant les mesures de sécurité mises en place.

##### b) Présentation des Dashboards

Nous avons configuré les dashboards Jenkins pour présenter de manière claire et intuitive l'état des jobs, permettant une visualisation rapide des résultats.

![alt text](https://github.com/tropizz/PROJET_FINAL_Jenkins/blob/main/screenshots/dashboard_jenkins.png)

![alt text](https://github.com/tropizz/PROJET_FINAL_Jenkins/blob/main/screenshots/repertoire_jenkins_workspace.png)

### Analyse du Script d'Exécution (Pipeline Jenkins)

Voici un extrait du dernier script Jenkins Pipeline que nous avons utilisé pour orchestrer les différents jobs dans le pipeline :

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
