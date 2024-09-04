/* groovylint-disable LineLength, NestedBlockDepth */
def services = [
        [path: 'app/ledger/balancereader', image: 'boa-balancereader:v0.1'],
        [path: 'app/ledger/ledgerwriter', image: 'boa-ledgerwriter:v0.1'],
        [path: 'app/ledger/transactionhistory', image: 'boa-transactionhistory:v0.1'],
        [path: 'app/ledger/ledger-db', image: 'boa-ledgerdb:v0.1'],
        [path: 'app/loadgenerator', image: 'boa-loadgenerator:v0.1'],
        [path: 'app/frontend', image: 'boa-frontend:v0.1'],
        [path: 'app/accounts/accounts-db', image: 'boa-accountsdb:v0.1'],
        [path: 'app/accounts/contacts', image: 'boa-contacts:v0.1'],
        [path: 'app/accounts/userservice', image: 'boa-userservice:v0.1']
]
def builtImages = []
pipeline {
    agent {
        docker {
            image 'docker:latest'
        }
    }

    services.each { service ->
        service.image = service.image.replace('v0.1', "v${BUILD_NUMBER}")
    }

    environment {
        DOCKER_ACCESS_KEY_ID  = credentials('docker_key')
        DOCKER_SECRET_ACCESS_KEY  = credentials('docker_secret')
        DOCKER_FILE = credentials('docker_secret_file')
        SUCCESS = 'SUCCESS'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',  url: 'https://github.com/chandhuDev/bank-of-anthos-app'
            }
        }
        stage('SonarQube Check') {
            steps {
                    input(message: 'Enter URL parameter:', parameters: [string(name: 'SONAR_URL')])

                    withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                        services.each { service ->
                        dir(service.path) {
                            echo "Running SonarQube analysis for ${service.path}..."
                            sh "mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}"
                        }
                        }
                    }
            }
        }
        stage('Build Docker images') {
            when {
                expression { currentBuild.previousStageResult == "${SUCCESS}" }
            }
            steps {
                    withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                        script {
                            services.each { item ->
                            def changes = sh(script: "git diff --name-only HEAD~1 HEAD ${item.path}", returnStdout: true).trim()

                            if (changes) {
                                echo "Changes detected in ${item.path}, building image ${item.image}..."
                                dir(item.path) {
                                    sh "docker build -t chandhudev0/${item.image} ."
                                    sh "docker push chandhudev0/${item.image}"
                                }
                                builtImages.push(item.image)
                            } else {
                                echo "No changes detected in ${item.path}, skipping build for ${item.image}."
                            }
                            }
                        }
                    }
            }
        }
        stage('Trivy Check') {
            when {
                expression { currentBuild.previousStageResult == "${SUCCESS}" }
            }
            steps {
                script {
                    builtImages.each { image ->
                        echo "Running Trivy scan on image: chandhudev0/${image}"
                        sh "/home/trivyUser/bin/trivy image --severity CRITICAL,HIGH --format template --template-output table --ignore-unfixed chandhudev0/${image}"
                    }
                }
            }
        }
    }
}
