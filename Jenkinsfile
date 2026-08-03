pipeline {
    agent any
    stages {
        stage('拉取代码') {
            steps {
                checkout scm
            }
        }
        stage('编译') {
            steps {
                sh 'gcc main.c -o main_app'
            }
        }
        stage('运行') {
            steps {
                sh './main_app'
            }
        }
    }
    post {
        success { echo "✅ main 分支构建成功" }
        failure { echo "❌ main 分支构建失败" }
        always  { cleanWs() }
    }
}
