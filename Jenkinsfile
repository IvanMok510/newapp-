pipeline {
    agent any

    environment {
        APP_NAME = "simple-node"
        PROJECT = "cicd-demo"
    }

    stages {
        stage('Clone Code') {
            steps {
                git 'https://github.com/IvanMok510/newapp-.git'
            }
        }

        stage('Build & Deploy') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject(env.PROJECT) {
                            echo "🛠️ 2. 构建并部署应用"

                            // 假设已经存在 nodejs:18 的镜像流
                            def appName = env.APP_NAME
                            def repoUrl = "https://github.com/IvanMok510/newapp-.git"

                            // 检查 BuildConfig 是否存在
                            def bc = openshift.selector('bc', appName).object()
                            if (!bc) {
                                echo "🔧 创建新应用..."
                                openshift.newApp("nodejs:18~${repoUrl}", "--name=${appName}")
                            } else {
                                echo "🔁 应用已存在，跳过创建"
                            }

                            // 强制重新构建
                            openshift.startBuild(appName)
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
                echo "🎉 应用已部署！访问: http://${url}"
            }
        }
        failure {
            echo "❌ 部署失败，请查看日志"
        }
    }
}
