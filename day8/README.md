# cloud4c-jenkinsb2

### Adjusting security analysis

<img src="sec1.png">

### jenkinsfile for detecting private key

```
pipeline {
    agent any

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
                sh ' docker run -it --rm  trufflesecurity/trufflehog:latest  github --repo https://github.com/redashu/ashu-cisco-webUI.git --json     >testkey.txt'
                // checking presence
                sh 'cat testkey.txt | grep -i private'
            }
        }
    }
}

```

### adding docker pipeline build method

```
pipeline {
    agent any

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
    }
}

```

## adding SASt pipeline in the image

```
pipeline {
    agent any

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
    }
}

```

### adding compose for health page testing 

```
pipeline {
    agent any

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
    }
}

```

### adding CI pipeline for pushing image

```
pipeline {
    agent any

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
    }
}

```
