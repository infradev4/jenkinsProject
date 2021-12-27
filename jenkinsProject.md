# Mini-projet Jenkins: Static Website

## Énoncé
```txt
1) Forkez ce repo afin d’en disposer librement.
https://github.com/diranetafen/static-website-example.git

2) Créer un fichier Dockerfile permettant de conteneuriser cette application.

3) Déployez un serveur Jenkins (from scratch) et non à l’aide de docker sur une instance AWS ec2.

4) Créez un jenkinsfile permettant de builder tester et déployer automatiquement cette application sur une machine cloud Ec2 (prenom-ec2-prod).

5) Réalisez dans votre pipeline le test de bon fonctionnement de votre pplication une fois celle-ci déployée en environnement de prod (Ec2-cloud).

6) Réalisez des différentes notifications de fin de build vers Github et aussi vers votre slack. 
NB: Utilisez un code avec structure conditionnelle pour les notifications, comme celui utilisé dans la shared Library mais cette fois-ci sans utiliser de shared library, le code devra être saisi en dur dans votre Jenkinsfile.

7) Rédigez un rapport décrivant entièrement les actions réalisées avec capture d’écran que vous sauvegarderez dans l’intranet partagé de l’établissement.
```

## Jenkins Master 
### Création d'une instance Ec2

Caractéristiques :
```txt
Ubuntu Server 20.04 LTS (HVM), SSD Volume Type
t2.large 
8Go RAM 
20Go SSD
```

## Exécuter des commandes au lancement
```txt
Lorsque vous lancez une instance dans Amazon EC2, vous avez la possibilité de transmettre les données des utilisateurs vers l'instance qui peut être utilisée pour exécuter des scripts après le démarrage de l'instance.
Dans mon cas, je lance différentes installations pour mon Master jenkins
```

### Liste des installations:
* Jenkins
* Java
* Docker

### Script d'installation:
```sh
#!/bin/bash
#install Jenkins
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y

#Installation of Java
sudo apt-get install -y jenkins
sudo apt install -y openjdk-11-jdk

#Start Jenkins
sudo systemctl daemon-reload
sudo systemctl start jenkins

#Install docker
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
sudo usermod -aG docker ubuntu
```

* Nom de mon instance Ec2 : `oussama-JenkinsScratch-Projet`
* ID de mon instance Ec2 :	    `i-00084f2f372aa63a5`
* IPv4 publique de mon instance Ec2 :	`54.157.132.177`

## Déverrouillage de Jenkins
http://54.157.132.177:8080/

![Getting Started](/assets/setup-jenkins-01-unlock-jenkins-page.jpg)

```txt
L'assistant de configuration post-installation démarre.
Lorsque vous accédez pour la première fois à une nouvelle instance Jenkins, vous êtes invité à la déverrouiller à l'aide d'un mot de passe généré automatiquement. Accédez à http://54.157.132.177:8080 et attendez que la page Déverrouiller Jenkins apparaisse.

À partir de la sortie du journal de la console Jenkins, copiez le mot de passe alphanumérique généré automatiquement.
```

* La commande suivante imprimera le mot de passe sur la console:
```bash
=>sudo cat /var/lib/jenkins/secrets/initialAdminPassword
398355dfd08644f1b060fab6ebb4b42f
```
nom:	oussama
mdp:	xxxxxxx
mail:	oussama@oussama.com

# GitHub

## Forker et cloner le projet
forker le projet sur notre repos Git : https://github.com/diranetafen/static-website-example

![Getting Started](/assets/a.png)
![Getting Started](/assets/b.png)

## Créer un Dockerfile dans mon depot GitHub
Ces lignes de code représentent l'image que nous allons utiliser en copiant le contenu du répertoire courant dans le conteneur.
```sh
# nous allons utiliser une image de base nginx
FROM nginx
# copie le contenu du répertoire courant dans le conteneur.
COPY ./ /usr/share/nginx/html
```
![Getting Started](/assets/c.png)

## Instance de production
### Création d'une instance Ec2 de production

* Caractéristiques :
```txt
Ubuntu Server 20.04 LTS (HVM)
8Go SSD Volume Type
t2.micro 
```

* Nom de mon instance Ec2 : `oussama-ec2-prod`
* IPv4 publique de mon instance Ec2 :	`34.207.190.81`

### Liste d'installation:
* Docker

### Script d'installation:

Exécuter le script après le démarrage de l'instance:

```sh
#!/bin/bash
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
usermod -aG docker ubuntu
```

# Pipeline as code

##jenkinsProject

### Création de notre Pipline

Description: 
```txt
Pipeline avec un Jenkinsfile permettant de builder tester et déployer automatiquement l'application Static Website Example sur une machine cloud Ec2.
```
(Cocher) GitHub project : https://github.com/infradev4/jenkinsProject.git
![Getting Started](/assets/d.png)
(Cocher) GitHub hook trigger
![Getting Started](/assets/e.png)
Avancé

* Pipline script from SCM
* SCM : GIT
* Repoistory URL : https://github.com/infradev4/jenkinsProject.git
* Crédential (aucun) car repo public
![Getting Started](/assets/f.png)
* Lancer notre build

# Comment intégrer Jenkins à GitHub
## Installer le pugin : GitHub Integration

https://www.guru99.com/jenkins-github-integration.html
https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/Jenkins-with-GitHub-Integration-Guide
https://www.blazemeter.com/blog/how-to-integrate-your-github-repository-to-your-jenkins-project

Les étapes requises pour réaliser l'intégration de Jenkins avec GitHub sont :
* Installez le plugin d'intégration Jenkins GitHub 
* Utilisez les URL GitHub pour extraire le code source dans Jenkins 
* Utilisez les webhooks GitHub pour déclencher les tâches de build Jenkins

Administrer Jenkins → Plugin → Installer → GitHub Integration Plugin
![Getting Started](/assets/g.png)

## Rajouter 2 crédentials:

* Administrer Jenkins → Manage Credentials → Stores scoped to Jenkins → Global → Add Credential
    * Portée : Global
    * Type : SSH Username with private key
	* ID: ec2_production_private_key
	* Description: key for ac2 aws production
	* Username: ubuntu
        * Secret : key ssh

#Pour trouver le token dans Dockerhub:

Account Settings → Security        

    * Portée : Global
    * Type : Secret text
        * Secret : Token de docker Hub
        * ID: dockerhub_password


Mettre l'URL coté Jenkins :
Administrer Jenkins → Configurer le system → Jenkins Location
http://54.157.132.177:8080/

Relier Jenkins à GitHub pour notifier 
[ Jenkins ] → Administrer Jenkins → Plugin → Embeddable-build-status
![Getting Started](/assets/h.png)
[ Jenkins ] → jenkinsProject → Embeddable Build Status → Links / Markdown / unprotected

[![Build Status](http://54.157.132.177:8080/buildStatus/icon?job=jenkinsProject)](http://54.157.132.177:8080/job/jenkinsProject/)

## Slack
* Créer un compte slack
![Getting Started](/assets/i.png)
* Créer un workSpace AJC-JENKINS
* ajouter un nouveau canal : ajc-formation-devops
* ajouter une application : CI/CD Jenkins
![Getting Started](/assets/k.png)
* token : xxxxxxxxx



![Getting Started](/assets/l.png)
![Getting Started](/assets/m.png)
![Getting Started](/assets/n.png)
![Getting Started](/assets/o.png)
###Configuration sur Jenkins
[ Jenkins ] → Administrer Jenkins → Plugin → Slack Notification Plugin

[ Jenkins ] → configurer le système → Slack
Créer le Credential slack_token
![Getting Started](/assets/p.png)
![Getting Started](/assets/q.png)

Rendre partie pipline manuelle
[ Jenkins ] → Administrer Jenkins → Plugin → Build Pipline


##Créer un nouveau projet (activer écoute webhook)
[ Jenkins ] → Nouveau Item

Relier GitHub a notre Jenkins
[ GitHub ] : Projet → Settings → Webhooks → add webhook
![Getting Started](/assets/Addwebhook.png)


Set URL de Jenkins: http://54.157.132.177:8080/github-webhook/

![Getting Started](/assets/jenkinsProject.png)
![Getting Started](/assets/Welcome.png)

# JenkinsFile
```Ruby
pipeline {

    environment {
        IMAGE_NAME = "staticwebsite"
        USERNAME = "devoupssama"
        CONTAINER_NAME = "webapp"
        EC2_PRODUCTION_HOST = "54.242.116.111"
	}

    agent none

    stages{

       stage ('Build Image'){
           agent any
           steps {
               script{
                   sh 'docker build -t $USERNAME/$IMAGE_NAME:$BUILD_TAG .'
               }
           }
       }

       stage ('Run test container') {
           agent any
           steps {
               script{
                   sh '''
                       docker stop $CONTAINER_NAME || true
                       docker rm $CONTAINER_NAME || true
                       docker run --name $CONTAINER_NAME -d -p 5000:80 $USERNAME/$IMAGE_NAME:$BUILD_TAG
                       sleep 6
                   '''
               }
           }
       }

       stage ('Test container') {
           agent any
           steps {
               script{
                   sh '''
                       curl http://localhost:5000 | grep -iq "Dimension"
                   '''
               }
           }
       }

       stage ('clean env and save artifact') {
           agent any
           environment{
               PASSWORD = credentials('dockerhub_password')
           }
           steps {
               script{
                   sh '''
                       docker login -u $USERNAME -p $PASSWORD
                       docker push $USERNAME/$IMAGE_NAME:$BUILD_TAG
                       docker stop $CONTAINER_NAME || true
                       docker rm $CONTAINER_NAME || true
                       docker rmi $USERNAME/$IMAGE_NAME:$BUILD_TAG
                   '''
               }
           }
       }

        stage('Deploy app on EC2-cloud Production') {
        agent any
        when{
            expression{ GIT_BRANCH == 'origin/master'}
        }
        steps{
            withCredentials([sshUserPrivateKey(credentialsId: "ec2_production_private_key", keyFileVariable: 'keyfile', usernameVariable: 'NUSER')]) {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    script{ 
                            sh'''
                                ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${EC2_PRODUCTION_HOST} docker stop $CONTAINER_NAME || true
								ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${EC2_PRODUCTION_HOST} docker rm $CONTAINER_NAME || true
								ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${EC2_PRODUCTION_HOST} docker run --name $CONTAINER_NAME -d -p 5000:80 $USERNAME/$IMAGE_NAME:$BUILD_TAG
                            '''
                        }
                    }
                }
            }
        }
       
    }
    post {
        success{
            slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        failure {
            slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
    }
}
```