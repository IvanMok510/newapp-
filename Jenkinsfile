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
                    echo "🛠️ 2. 构建并部署应用"

                    def appName = env.APP_NAME
                    def repoUrl = "https://github.com/IvanMok510/newapp-.git"

                    // 检查 BuildConfig 是否存在
                    def bc = openshift.selector('bc', appName).object()
                    if (!bc) {
                        echo "🔧 创建新应用..."
                        // ✅ 使用本地镜像流：nodejs:18
                        openshift.newApp("nodejs:18~${repoUrl}", "--name=${appName}")
                    } else {
                        echo "🔁 应用已存在，跳过创建"
                    }

                    // 重新构建
                    echo "🚀 开始构建..."
                    def build = openshift.startBuild(appName)
                    
                    // 等待构建完成
                    openshift.poll {
                        def latestBuild = openshift.selector('bc', appName).object().status.lastVersion
                        return build.object().metadata.name == "${appName}-${latestBuild}"
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
