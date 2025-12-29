pipeline {
    agent any

    environment {
        SONAR_HOST = "http://172.31.11.246:9000"
        NEXUS_HOST = "172.31.14.124:8081"
        NEXUS_REPO = "maven-snapshort"
        GROUP_ID = "com.example"
        ARTIFACT_ID = "maven-projects"
        VERSION = "1.0"
        TOMCAT_IP = "172.31.8.183"
    }

    stages {

        stage('Checkout') {
    steps {
        git branch: 'main',
            url: 'https://github.com/Theertha12345/maven-projects.git'
       }
   }


        stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('theertha sonar') {
            sh '''
            sonar-scanner \
              -Dsonar.projectKey=maven-projects \
              -Dsonar.sources=src \
              -Dsonar.java.binaries=target
            '''
        }
    }
}


        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUS_HOST}",
                    repository: "${NEXUS_REPO}",
                    credentialsId: 'nexus-creds',
                    groupId: "${GROUP_ID}",
                    version: "${VERSION}",
                    artifacts: [
                        [artifactId: "${ARTIFACT_ID}",
                         classifier: '',
                         file: "target/${ARTIFACT_ID}.war",
                         type: 'war']
                    ]
                )
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sshagent(['tomcat-ssh']) {
                    sh """
                    ssh ubuntu@${TOMCAT_IP} '
                    docker exec tomcat bash -c "
                    rm -rf /usr/local/tomcat/webapps/${ARTIFACT_ID}*
                    wget http://${NEXUS_HOST}/repository/${NEXUS_REPO}/${GROUP_ID.replace('.','/')}/${ARTIFACT_ID}/${VERSION}/${ARTIFACT_ID}-${VERSION}.war \
                    -O /usr/local/tomcat/webapps/${ARTIFACT_ID}.war
                    "
                    '
                    """
                }
            }
        }
    }
}
