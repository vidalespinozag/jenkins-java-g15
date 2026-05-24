pipeline {
    agent any

    tools {
        jdk 'JDK-17'
    }

    triggers {
        githubPush()
    }

    environment {
        APP_NAME    = 'todos-api'
        JAR_VERSION = '0.0.1-SNAPSHOT'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'git log -1 --format="%h %s"'
            }
        }

        stage('Build') {
            steps {
                sh 'chmod +x mvnw'
                sh './mvnw clean compile -B'
            }
        }

        stage('Test') {
            steps {
                sh './mvnw test -B'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                sh './mvnw package -DskipTests -B'
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts(
                    artifacts: "target/${APP_NAME}-${JAR_VERSION}.jar",
                    fingerprint: true
                )
            }
        }
    }

    post {
        success {
            echo "Pipeline succeeded. Artifact: target/${APP_NAME}-${JAR_VERSION}.jar"
            withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                sh """
                    curl -s -X POST \
                      -H "Authorization: token \$GITHUB_TOKEN" \
                      -H "Content-Type: application/json" \
                      -d '{"state":"success","context":"Jenkins CI","description":"Build passed"}' \
                      "https://api.github.com/repos/\$(echo \$GIT_URL | sed 's|.*github.com/||;s|.git\$||')/statuses/\${GIT_COMMIT}"
                """
            }
        }
        failure {
            echo "Pipeline FAILED — review the stage logs above for details."
            withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                sh """
                    curl -s -X POST \
                      -H "Authorization: token \$GITHUB_TOKEN" \
                      -H "Content-Type: application/json" \
                      -d '{"state":"failure","context":"Jenkins CI","description":"Build failed"}' \
                      "https://api.github.com/repos/\$(echo \$GIT_URL | sed 's|.*github.com/||;s|.git\$||')/statuses/\${GIT_COMMIT}"
                """
            }
        }
    }
}