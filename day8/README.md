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
