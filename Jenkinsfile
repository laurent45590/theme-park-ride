pipeline {
    agent any
    stages {
        stage('SCM') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            parallel {
                stage('clean Compile') {
                    agent {
                        docker {
                            image 'maven:3.6.0-jdk-8-alpine'
                            args '-v /root/.m2/repository:/root/.m2/repository'
                            // to use the same node and workdir defined on top-level pipeline for all docker agents
                            reuseNode true
                        }
                    }
                    steps {
                        sh ' mvn clean compile'
                    }
                }
                stage('CheckStyle') {
                    agent {
                        docker {
                            image 'maven:3.6.0-jdk-8-alpine'
                            args '-v /root/.m2/repository:/root/.m2/repository'
                            reuseNode true
                        }
                    }
                    steps {
                        sh ' mvn checkstyle:checkstyle'
                    }
                    post {
                        always {
                            recordIssues enabledForFailure: true, tool: checkStyle()
                        }
                    }
                }
            }
        }
        stage('Unit Tests') {
            when {
                anyOf { branch 'master'; branch 'develop' }
            }
            agent {
                docker {
                    image 'maven:3.6.0-jdk-8-alpine'
                    args '-v /root/.m2/repository:/root/.m2/repository'
                    reuseNode true
                }
            }
            steps {
                sh ' mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/**/*.xml'
                }
            }
        }
        stage('Integration Tests') {
            when {
                anyOf { branch 'master'; branch 'develop' }
            }
            agent {
                docker {
                    image 'maven:3.6.0-jdk-8-alpine'
                    args '-v /root/.m2/repository:/root/.m2/repository'
                    reuseNode true
                }
            }
            steps {
                sh 'mvn verify -Dsurefire.skip=true'
            }
            post {
                always {
                    junit 'target/failsafe-reports/**/*.xml'
                }
                success {
                    stash(name: 'artifact', includes: 'target/*.jar')
                    stash(name: 'pom', includes: 'pom.xml')
                    archiveArtifacts 'target/*.jar'
                }
            }
        }
    }
}
