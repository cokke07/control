pipeline {
    agent any 

    tools { 
        maven 'jenkinsmaven'
        jdk 'java11'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/MiguelAngelRamos/control.git'
            }
        }

        stage('Build') {
            steps {
                script {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Test') {
            steps {
                script {
                    sh 'mvn test'
                }
            }
        }

        stage('Sonar Scanner') {
        steps {
            script {
                def sonarqubeScannerHome = tool name: 'sonar', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                withCredentials([string(credentialsId: 'sonar', variable: 'sonarLogin')]) {
                    sh "${sonarqubeScannerHome}/bin/sonar-scanner -e -Dsonar.host.url=http://SonarQube:9000 -Dsonar.login=${sonarLogin} -Dsonar.projectName=mv-maven -Dsonar.projectVersion=${env.BUILD_NUMBER} -Dsonar.projectKey=GS -Dsonar.sources=src/main/java/com/kibernumacademy/miapp -Dsonar.tests=src/test/java/com/kibernumacademy/miapp -Dsonar.language=java -Dsonar.java.binaries=."
                }
            }
        }
    }

/*         stage('SonarQube Analysis') {
          steps {
            script {
              def sonarqubeScannerHome = tool 'sonar'
              withSonarQubeEnv('sonar') {
                sh "${sonarqubeScannerHome}/bin/sonar-scanner -e -Dsonar.host.url=http://SonarQube:9000 -Dsonar.login=${sonarLogin} -Dsonar.projectName=mv-maven -Dsonar.projectVersion=${env.BUILD_NUMBER} -Dsonar.projectKey=GS"
              }
            }
          }
        }

        stage('Quality Gate') {
          steps {
            timeout(time: 1, unit: 'HOURS') {
              waitForQualityGate abortPipeline: true
            }
          }
        } */


        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonar'
                    withSonarQubeEnv('sonar') {
                    sh "${scannerHome}/bin/sonar-scanner"
                        }
                    }
                }
            }

        stage('nexus') {

                steps {

                    script {

                          dir("target"){

                            def pom = readMavenPom file: "pom.xml"

                                        nexusArtifactUploader(

                                            nexusVersion: '0.0.1-SNAPSHOT',

                                            protocol: 'http',

                                            nexusUrl: 'nexus:8081',

                                            groupId: pom.groupId,

                                            version: pom.version,

                                            repository: 'maven-snapshots',

                                            credentialsId: 'nexusLogin',

                                            artifacts: [

                                                [artifactId: pom.artifactId,

                                                classifier: '',

                                                file: 'ControlInventario-0.0.1-SNAPSHOT.jar',

                                                type: pom.packaging]

                                            ]

                                        )

                                    }

                              }

                     }

            }
    }
    post {
                    always {
                        script {
                            // Check the Quality Gate status
                            def qg = waitForQualityGate()
                            if (qg.status != 'OK') {
                                error "Quality Gate failed: ${qg.status}"
                            }
                        }
                    }
                }
}