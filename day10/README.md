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

