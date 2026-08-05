pipeline {
    agent none
    options {
        buildDiscarder(logRotator(daysToKeepStr:'7', numToKeepStr:'10'))
        disableConcurrentBuilds()
        timestamps()
    }
    stages {
        stage("架构分支信息确认") {
            agent { label 'built-in' }
            steps {
                echo "======================================"
                echo "选定架构: ${params.BUILD_ARCH}"
                echo "选定Git分支/Tag: ${params.BUILD_BRANCH_TAG}"
                echo "======================================"
            }
        }

        stage("编译 + 测试") {
            agent any
            steps {
                script {
                    // 根据参数选择目标节点名称
                    def targetNode
                    switch(params.BUILD_ARCH) {
                        case "hygon":
                            targetNode = "hg"
                            break
                        case "intel":
                            targetNode = "intel-node"
                            break
                        case "amd":
                            targetNode = "amd-node"
                            break
                        default:
                            error("不支持的架构参数：${params.BUILD_ARCH}")
                    }
                    echo "将调度任务至节点：${targetNode}"

                    // 在目标节点执行所有编译流程
                    node(targetNode) {
                        echo "当前执行节点：${env.NODE_NAME}"
                        echo "开始检出分支/tag：${params.BUILD_BRANCH_TAG}"
                        // 动态节点必须手动拉取代码！关键！
                        checkout scm

                        stage("编译构建") {
                            echo "===== 执行编译流程 ====="
                            bat "gcc src\main.c -o main.exe"
                            // Windows构建使用bat命令
                            // bat "build.bat"
                        }
                        stage("功能测试") {
                            echo "===== 执行测试流程 ====="
                            bat "main.exe"
                            // bat "test.bat"
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            echo "流水线执行结束"
        }
        success {
            echo "✅构建成功！架构：${params.BUILD_ARCH}，分支：${params.BUILD_BRANCH_TAG}"
        }
        failure {
            echo "❌构建失败！架构：${params.BUILD_ARCH}，分支：${params.BUILD_BRANCH_TAG}"
        }
    }
}
