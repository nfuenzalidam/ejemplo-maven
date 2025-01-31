import groovy.json.JsonSlurperClassic

def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}
pipeline {
    agent any
    environment {
        NEXUS_USER         = credentials('nexus-user')
        NEXUS_PASSWORD     = credentials('nexus-password')
    }
    stages {
        stage('Paso 1: Compliar') {
            steps {
                script {
                    sh "echo 'Compile Code!'"
                    // Run Maven on a Unix agent.
                    sh 'mvn clean compile -e'
                }
            }
        }
        stage('Paso 2: Testear') {
            steps {
                script {
                    sh "echo 'Test Code!'"
                    // Run Maven on a Unix agent.
                    sh 'mvn clean test -e'
                }
            }
        }
        stage('Paso 3: Build .Jar') {
            steps {
                script {
                    sh "echo 'Build .Jar!'"
                    // Run Maven on a Unix agent.
                    sh 'mvn clean package -e'
                }
            }
            post {
                //record the test results and archive the jar file.
                success {
                    archiveArtifacts artifacts:'build/*.jar'
                }
            }
        }
        stage('Paso 4: Análisis SonarQube') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh "echo 'Calling sonar Service in another docker container!'"
                    // Run Maven on a Unix agent to execute Sonar.
                    sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=github-sonar'
                   // sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=github-sonar -Dsonar.host.url=https://sonarqube.planeta0.com'
                }
            }
        }

        stage('Paso 5: Subir a Nexus') {
            steps {
                //archiveArtifacts artifacts:'build/*.jar'
                nexusPublisher nexusInstanceId: 'nexus',
                nexusRepositoryId: 'devops-usach',
                    packages: [
                    [$class: 'MavenPackage',
                       mavenAssetList: [
                            [classifier: '',
                            extension: 'jar',
                            filePath: 'build/DevOpsUsach2020-0.0.1.jar']
                            ],
                        mavenCoordinate: [
                            artifactId: 'DevOpsUsach2020',
                            groupId: 'com.devopsusach2020',
                            packaging: 'jar',
                            version: '0.0.3']
                    ]
                    ]
            }
        }
        stage(" Paso 6: Download: Nexus"){
            steps {
                sh "echo 'fase success'"
                sh "pwd"
                sh ' curl -X GET -u $NEXUS_USER:$NEXUS_PASSWORD "http://nexus:8081/repository/devops-usach/com/devopsusach2020/DevOpsUsach2020/0.0.3/DevOpsUsach2020-0.0.3.jar" -O'
                sh "ls -l"
            }
        }
        stage(" Paso 7: Levantar Springboot APP"){
            steps {
               sh 'nohup java -jar DevOpsUsach2020-0.0.3.jar & >/dev/null'
            }
        }
        stage('Paso 8: Dormir(Esperar 60sg, que levante sprint boot) ') {
            steps {
                sh 'sleep 60'
            }
        }
        stage("Paso 9: Curl"){
            steps {
               sh "curl -X GET 'http://localhost:8085/rest/mscovid/test?msg=testing'"
            }
        }
    }
    post {
        always {
            sh "echo 'fase always executed post'"
        }
        success {
            sh "echo 'fase success'"
        }
        failure {
            sh "echo 'fase failure'"
        }
    }
}