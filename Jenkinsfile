pipeline {
    agent any

    environment {
        APP_NAME = "simple-node"
        PROJECT = "cicd-demo"
    }

    stages {
        stage('Clone Code') {
            steps {
                script {
                    echo "✅ 1. clone code"
                    sh 'ls -la'
                }
            }
        }

        stage('Build & Deploy') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject(env.PROJECT) {
                            echo "🛠️ 2. Build & Deploy"

                            openshift.newApp("nodejs:18~https://github.com/yourusername/simple-node-app.git", "--name=${APP_NAME}")
                                .narrow()

                            openshift.startBuild("${APP_NAME}").narrow()

                            openshift.poll {
                                return openshift.selector("bc/${APP_NAME}").object().status.phase == "Complete"
                            }
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                def url = openshift.selector("route", "${APP_NAME}").object().spec.host
                echo "🎉done and go : http://${url}"
            }
        }
        failure {
            echo "❌ fail"
        }
    }
}