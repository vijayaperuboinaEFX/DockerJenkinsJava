pipeline {
    agent any {
        environment {
            registry = "YourDockerhubAccount/YourRepository"
            registryCredential = 'dockerhub_id'
            dockerImageId =""
            JAVA_HOME="/usr/java/jdk1.8.0_181-amd64"
            /* Branch information */
            BRANCH_DEV = 'develop'
            BRANCH_MASTER = 'master'
            BRANCH_HOTFIX = 'hotfix/prod'
            /*Sonar Tool Confiuration*/
            SCANNER_HOME = tool 'Sonar-scanner'

        }
        parameters { string(name: 'DEPLOY_ENV', defaultValue: 'staging', description: '') }
        
        tools {
            maven 'mvn'
            jdk 'java'
        }
    }
    stages {
        stage('Clone Repo') {
             steps {
               git credentialsId: 'git_credentials', url: 'https://github.com/ravdy/hello-world.git'
            }
        }
        stage('Build and test') {
            steps{
                    script {
                        sh 'mvn clean verify -DskipITs=true';
                        dockerImage = docker.build registry + ":$BUILD_NUMBER"
                    }
                junit(allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml')
                jacoco()
            }
        }
        stage('Sonar Analaysis') {
             steps {
                     withSonarQubeEnv(credentialsId: 'sonar-credentialsId', installationName: 'Sonar') {
                     sh '''$SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=projectKey \
                    -Dsonar.projectName=projectName \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/classes/ \
                    -Dsonar.exclusions=src/test/java/****/*.java \
                    -Dsonar.java.libraries=/var/lib/jenkins/.m2/**/*.jar \
                    -Dsonar.projectVersion=${BUILD_NUMBER}-${GIT_COMMIT_SHORT}'''

             }

        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    timeout(time: 3, unit: 'MINUTES') {
                        def qualitygate = waitForQualityGate()
                        if (qualitygate.status != "OK") {      
                            println "Marking the BUILD UNSTABLE because of SonarQube Quality Gate Error"
                            currentBuild.result = 'UNSTABLE'
                        }
                    }
                }
            }
        }

        stage('Deploy Image to Dockerhub') {
            steps{
                script {
                        docker.withRegistry( '', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }   
        stage('Cleaning up') {
            steps{
                    sh "docker rmi $registry:$BUILD_NUMBER"
                }
            }


    }

}