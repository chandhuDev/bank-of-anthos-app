def services = [
        [path: 'app/accounts/accounts-db', image: 'boa-accountsdb:v0'],
        [path: 'app/accounts/contacts', image: 'boa-contacts:v0'],
        [path: 'app/accounts/userservice', image: 'boa-userservice:v0'],
        [path: 'app/frontend', image: 'boa-frontend:v0'],
        [path: 'app/ledger/ledger-db', image: 'boa-ledgerdb:v0'],
        [path: 'app/ledger/ledgerwriter', image: 'boa-ledgerwriter:v0'],
        [path: 'app/ledger/balancereader', image: 'boa-balancereader:v0'],
        [path: 'app/ledger/transactionhistory', image: 'boa-transactionhistory:v0'],
        [path: 'app/loadgenerator', image: 'boa-loadgenerator:v0'],
]
def builtImages = []

pipeline {
    agent any
    tools {
        jdk "jdk17"
        maven 'maven'
    }
    environment {
        SUCCESS = 'SUCCESS'
        sonar = tool 'sonar'
        SONAR_AUTH_TOKEN = 'sonar-cred'
    }

    parameters {
        booleanParam(name: 'FULL_BUILD', defaultValue: false, description: 'Force a full build of all Docker images')
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    services.each { service ->
                        service.image = service.image.replace('v0', "v0.${BUILD_NUMBER}")
                    }
                }
                git branch: 'master', url: 'https://github.com/chandhuDev/bank-of-anthos-app.git'
            }
        }

        stage('SonarQube Check') {
            when {
                expression {
                    return !currentBuild.previousBuild?.result?.equals('SUCCESS')
                }
            }
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        withSonarQubeEnv('sonar') {
                            services.each { service ->
                                echo "Running SonarQube analysis for ${service.path}..."
                                dir("${service.path}") {
                                    if (fileExists('pom.xml')) {
                                        echo "Detected Java project in ${service.path}, running SonarQube analysis with Maven..."
                                        sh "mvn clean verify && mvn clean install"
                                        sh "mvn clean package && mvn sonar:sonar -Dsonar.login=${SONAR_AUTH_TOKEN}"
                                    } else if (fileExists('requirements.txt')) {
                                        echo "Detected Python project in ${service.path}, running SonarQube analysis with sonar-scanner..."
                                        sh """
                                        ${sonar}/bin/sonar-scanner \
                                        -Dsonar.projectKey=jenkins \
                                        -Dsonar.sources=. \
                                        """
                                    } else {
                                        echo "No recognized project type in ${service.path}, skipping SonarQube analysis."
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }

        stage('Build Docker images') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                echo "in docker folder"
                withCredentials([usernamePassword(credentialsId: 'docker-cred', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "echo ${DOCKER_USERNAME} and ${DOCKER_PASSWORD}"
                    sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"

                    script {
                        def forceFullBuild = params.FULL_BUILD
                        services.each { item ->
                            if (forceFullBuild) {
                                echo "Full build requested, building image ${item.image}..."
                                dir("${item.path}") {
                                    sh "docker build -t chandhudev0/${item.image} ."
                                    sh "docker push chandhudev0/${item.image}"
                                }
                                builtImages.push("${item.image}")
                            } else {
                                def changes = sh(script: "git diff --name-only HEAD~1 HEAD ${item.path}", returnStdout: true).trim()
                                if (changes) {
                                    echo "Changes detected in ${item.path}, building image ${item.image}..."
                                    dir("${item.path}") {
                                        sh "docker build -t chandhudev0/${item.image} ."
                                        sh "docker push chandhudev0/${item.image}"
                                    }
                                    builtImages.push("${item.image}")
                                } else {
                                    echo "No changes detected in ${item.path}, skipping build for ${item.image}."
                                }
                            }
                        }
                    }
                }
            }
        }

        stage('Trivy Check') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                script {
                    builtImages.each { image ->
                        echo "${image}"
                        echo "Running Trivy scan on image: chandhudev0/${image}"
                        sh "/usr/local/bin/trivy image --severity CRITICAL,HIGH --format template --template-output table --ignore-unfixed chandhudev0/${image}"
                    }
                }
            }
        }
    }
}

def hasPreviousSonarSuccess() {
    def prevBuild = currentBuild.previousBuild
    if (prevBuild != null && prevBuild.result == 'SUCCESS') {
        def sonarStage = prevBuild.rawBuild.getAction(hudson.tasks.junit.TestResultAction)
        if (sonarStage != null && sonarStage.getResult() == 'SUCCESS') {
            echo "Previous SonarQube analysis was successful, skipping this stage."
            return true
        }
    }
    return false
}
