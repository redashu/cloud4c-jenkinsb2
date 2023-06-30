# cloud4c-jenkinsb2

### Rev 1

<img src="rev1.png">

### revision 2

<img src="rev2.png">

### user and notification service 

<img src="ut.png">

### RBAC 

<img src="rbac.png">

## Notification & alert 

### SMTP 

<img src="smtp.png">

### jenkisfile job

```
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
        stage('testing docker connection'){
            steps {
                echo 'testing connection of docker'
                sh 'docker version'
                sleep 3 
                // i want to send an email notication about docker successfull connection
                mail bcc: '', body: '''yeahhhh ohhhhhhhhhhh!!
we got docker connection established hurrrahh''', cc: 'navinyab95@gmail.com', from: '', replyTo: '', subject: 'docker testing status', to: 'ashutoshh@delvex.io'
                
            }
        }
    }
}


```

### putting email notification in post section 

```
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
        stage('testing docker connection'){
            steps {
                echo 'testing connection of docker'
                sh 'docker version'
                sleep 3 
                // i want to send an email notication about docker successfull connection
            }
        }
        stage('testing kubernetes api-resources'){
            steps {
                echo 'lets check ingress controller deployment status'
                sh 'kubectl  check ingress -n ingress-nginx'
            }
        }
    }
    post {
        success {
            echo 'hey got it'
        }
        failure {
            echo 'drafting email'
            script {
                def subject1 = "Build Status: ${currentBuild.currentResult}"
                def body1 = """
                            <p>Build Status: ${currentBuild.currentResult}</p>
                            <p>Build Number: ${env.BUILD_NUMBER}</p>
                            <p>Build URL: ${env.BUILD_URL}</p>
                           """
                def to1 = "ashutoshh@delvex.io"
                def from1 = "asashuthegreat4@gmail.com"
                mail (
                    to: to1,
                    from: from1,
                    subject: subject1,
                    body: body1,
                    )
            }
        }
    }
}

```

### easiest way to integration jenkins + k8s

```
root@ip-172-31-49-102 ~]# grep jenkins /etc/passwd
jenkins:x:995:993:Jenkins Automation Server:/var/lib/jenkins:/bin/bash
[root@ip-172-31-49-102 ~]# 
[root@ip-172-31-49-102 ~]# 
[root@ip-172-31-49-102 ~]# 
[root@ip-172-31-49-102 ~]# mkdir  /var/lib/jenkins/.kube
[root@ip-172-31-49-102 ~]# cd   /var/lib/jenkins/.kube
[root@ip-172-31-49-102 .kube]# ls
[root@ip-172-31-49-102 .kube]# wget  http://3.130.106.53/admin.conf 
--2023-06-30 13:04:05--  http://3.130.106.53/admin.conf
Connecting to 3.130.106.53:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5640 (5.5K) [text/plain]
Saving to: 'admin.conf'

100%[===================================================================================================================>] 5,640       --.-K/s   in 0s      

2023-06-30 13:04:05 (390 MB/s) - 'admin.conf' saved [5640/5640]

[root@ip-172-31-49-102 .kube]# ls
admin.conf
[root@ip-172-31-49-102 .kube]# mv admin.conf config
[root@ip-172-31-49-102 .kube]# 



```

### testing connection 

```
pipeline {
    agent {
        label 'master'
    }

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
        stage('testing docker connection'){
            steps {
                echo 'testing connection of docker'
                sh 'docker version'
                sleep 3 
                // i want to send an email notication about docker successfull connection
            }
        }
        stage('testing kubernetes api-resources'){
            steps {
                echo 'lets check ingress controller deployment status'
                sh 'kubectl  get nodes'
            }
        }
    }
    post {
        success {
            echo 'hey got it'
        }
        failure {
            echo 'drafting email'
            script {
                def subject1 = "Build Status: ${currentBuild.currentResult}"
                def body1 = """
                            <p>Build Status: ${currentBuild.currentResult}</p>
                            <p>Build Number: ${env.BUILD_NUMBER}</p>
                            <p>Build URL: ${env.BUILD_URL}</p>
                           """
                def to1 = "ashutoshh@delvex.io"
                def from1 = "asashuthegreat4@gmail.com"
                mail (
                    to: to1,
                    from: from1,
                    subject: subject1,
                    body: body1,
                    )
            }
        }
    }
}

```

## adding k8s cd section in security pipelien 

```
pipeline {
    agent {
        label 'master'
    }

    stages {
        stage('cloning github source code') {
            steps {
                echo 'taking sample code to tesing'
                git 'https://github.com/redashu/ashu-cisco-webUI.git'
                // when we run any shell command it return exit code 0 on success 
                // verification also 
                sh 'ls | grep html'
            }
        }
        stage('doing SAST using trufflehog'){
            steps {
                echo 'using trufflehog to test private key presence'
                // using docker based truffleHog 
                sh ' docker run --rm  trufflesecurity/trufflehog:latest  github --repo https://github.com/redashu/ashu-cisco-webUI.git --json     >testkey.txt'
                // checking presence
                sh 'cat testkey.txt | grep -i private '
                sh 'exit 0'
            }
        }
        // building docker image using docker pipeline
        stage('building image'){
            steps {
                echo 'usign docker pipeline to build image'
                script {
                    def imageName = "dockerashu/ashusec"
                    def imageTag  = "version$BUILD_NUMBER"
                    // docker build function 
                    docker.build(imageName + ":" + imageTag, " -f Dockerfile .")
                }
                // verify image
                sh 'docker images  | grep ashusec'
            }
        }
        // checking SAST after build 
        stage('using Trivy to scan docker images for critical security severity'){
            steps {
                echo 'Using trivy'
                sh 'trivy image dockerashu/ashusec:version$BUILD_NUMBER >critic.txt'
                sh 'cat critic.txt | grep -i critical'
                
            }
        }
        // creating contaienr and accessing web page
        stage('using compose for doing this'){
            steps {
                echo 'using compose'
                sh 'docker-compose down'
                sh 'docker-compose up -d --build'
                sh 'docker-compose ps'
                sh 'curl -f http://localhost:1990/health.html'
            }
        }
        // lets push image to docker hub using docker pipeline 
        stage('pushing image to docker hub usign docker pipeline'){
            steps {
                echo 'pushing image'
                script {
                    def imageName = "dockerashu/ashusec"
                    def imageTag  = "version$BUILD_NUMBER"
                    def hubCred   = "b174d3db-9c22-420f-acbd-d1fbc2fbd40b"
                    // calling function
                    docker.withRegistry('https://registry.hub.docker.com',hubCred){
                        docker.image(imageName + ":" + imageTag).push()
                    }
                }
            }
        }
        stage('check connection and deploy deploymet'){
            steps {
                echo 'checking connection with k8s control plane'
                sh 'kubectl cluster-info'
                // creating namespace 
                sh 'kubectl apply -f  https://raw.githubusercontent.com/redashu/cloud4c-jenkins-cd/master/ns.yaml'
                sh 'kubectl get ns | grep ashu-final-app'
                // deploy the deployment 
                sh 'kubectl apply -f https://raw.githubusercontent.com/redashu/cloud4c-jenkins-cd/master/deploy.yaml'
                // verify deployment and pod status
                sh 'kubectl -n ashu-final-app get deploy | grep ashu-appweb'
                // wait and check pod status
                sleep 10
                sh 'kubectl -n ashu-final-app get pods | grep -i running'
            }
        }
    }
}

```

### final 

```
pipeline {
    agent {
        label 'master'
    }

    stages {
        stage('cloning github source code') {
            steps {
                echo 'taking sample code to tesing'
                git 'https://github.com/redashu/ashu-cisco-webUI.git'
                // when we run any shell command it return exit code 0 on success 
                // verification also 
                sh 'ls | grep html'
            }
        }
        stage('doing SAST using trufflehog'){
            steps {
                echo 'using trufflehog to test private key presence'
                // using docker based truffleHog 
                sh ' docker run --rm  trufflesecurity/trufflehog:latest  github --repo https://github.com/redashu/ashu-cisco-webUI.git --json     >testkey.txt'
                // checking presence
                sh 'cat testkey.txt | grep -i private '
                sh 'exit 0'
            }
        }
        // building docker image using docker pipeline
        stage('building image'){
            steps {
                echo 'usign docker pipeline to build image'
                script {
                    def imageName = "dockerashu/ashusec"
                    def imageTag  = "version$BUILD_NUMBER"
                    // docker build function 
                    docker.build(imageName + ":" + imageTag, " -f Dockerfile .")
                }
                // verify image
                sh 'docker images  | grep ashusec'
            }
        }
        // checking SAST after build 
        stage('using Trivy to scan docker images for critical security severity'){
            steps {
                echo 'Using trivy'
                sh 'trivy image dockerashu/ashusec:version$BUILD_NUMBER >critic.txt'
                sh 'cat critic.txt | grep -i critical'
                
            }
        }
        // creating contaienr and accessing web page
        stage('using compose for doing this'){
            steps {
                echo 'using compose'
                sh 'docker-compose down'
                sh 'docker-compose up -d --build'
                sh 'docker-compose ps'
                sh 'curl -f http://localhost:1990/health.html'
            }
        }
        // lets push image to docker hub using docker pipeline 
        stage('pushing image to docker hub usign docker pipeline'){
            steps {
                echo 'pushing image'
                script {
                    def imageName = "dockerashu/ashusec"
                    def imageTag  = "version$BUILD_NUMBER"
                    def hubCred   = "b174d3db-9c22-420f-acbd-d1fbc2fbd40b"
                    // calling function
                    docker.withRegistry('https://registry.hub.docker.com',hubCred){
                        docker.image(imageName + ":" + imageTag).push()
                    }
                }
            }
        }
        stage('check connection and deploy deploymet'){
            steps {
                echo 'checking connection with k8s control plane'
                sh 'kubectl cluster-info'
                // creating namespace 
                sh 'kubectl apply -f  https://raw.githubusercontent.com/redashu/cloud4c-jenkins-cd/master/ns.yaml'
                sh 'kubectl get ns | grep ashu-final-app'
                // deploy the deployment 
                sh 'kubectl apply -f https://raw.githubusercontent.com/redashu/cloud4c-jenkins-cd/master/deploy.yaml'
                // verify deployment and pod status
                sh 'kubectl -n ashu-final-app get deploy | grep ashu-appweb'
                // wait and check pod status
                sleep 10
                sh 'kubectl -n ashu-final-app get pods | grep -i running'
            }
        }
        stage('creating service and ingress'){
            steps {
                echo 'creating service'
                sh 'kubectl apply -f https://raw.githubusercontent.com/redashu/cloud4c-jenkins-cd/master/service.yaml'
                sleep 3
                sh 'kubectl apply -f https://raw.githubusercontent.com/redashu/cloud4c-jenkins-cd/master/ingress.yaml'
                // verify 
                sh 'kubectl -n ashu-final-app get svc,ing'
            }
        }
        stage('upgrading app'){
            steps {
                echo 'upgrading'
                sh "kubectl -n ashu-final-app set image deployment ashu-appweb ashuweb=dockerashu/ashusec:version$BUILD_NUMBER"
            }
        }
    }
}

```
