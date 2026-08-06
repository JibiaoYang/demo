pipeline {
    agent none
    options {
        buildDiscarder(logRotator(daysToKeepStr: '7', numToKeepStr: '10'))
        disableConcurrentBuilds()
        timestamps()
    }
    stages {
        stage("架构分支信息确认") {
            agent { label 'built-in' }
            steps {
                echo "======================================"
                echo "选定架构: ${params.BUILD_ARCH}"
                echo "选定Git分支: ${params.BUILD_BRANCH_TAG}"
                echo "======================================"
            }
        }
        stage("编译 + 测试") {
            agent any
            steps {
                script {
                    // 根据CPU架构选择Jenkins节点
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
                    // 动态切换执行节点
                    node(targetNode) {
                        echo "当前执行节点：${env.NODE_NAME}"
                        echo "开始检出Git分支：${params.BUILD_BRANCH_TAG}"
                        // 根据参数checkout指定分支
                        checkout([
                            $class: 'GitSCM',
                            branches: [[name: "${params.BUILD_BRANCH_TAG}"]],
                            userRemoteConfigs: [[url: scm.userRemoteConfigs[0].url]]
                        ])
                        stage("编译构建") {
                            echo "===== 当前Git分支：${params.BUILD_BRANCH_TAG} ====="
                            // 根据Git分支选择编译代码
                            switch(params.BUILD_BRANCH_TAG) {
                                case "main":
                                    echo "执行main分支编译"
                                    bat "gcc src/main.c -o main.exe"
                                    break
                                case "dev":
                                    echo "执行dev分支编译"
                                    bat "gcc src/dev.c -o dev.exe"
                                    break
                                default:
                                    error("当前分支没有对应编译规则：${params.BUILD_BRANCH_TAG}")
                            }
                        }
                        stage("功能测试") {
                            echo "===== 执行测试流程 ====="
                            switch(params.BUILD_BRANCH_TAG) {
                                case "main":
                                    echo "运行main测试"
                                    bat "main.exe"
                                    break
                                case "dev":
                                    echo "运行dev测试"
                                    bat "dev.exe"
                                    break
                                default:
                                    error("当前分支没有测试规则：${params.BUILD_BRANCH_TAG}")
                            }
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
            echo """
            ✅ 构建成功！
            架构：${params.BUILD_ARCH}
            Git分支：${params.BUILD_BRANCH_TAG}
            """
        }
        failure {
            echo """
            ❌ 构建失败！
            架构：${params.BUILD_ARCH}
            Git分支：${params.BUILD_BRANCH_TAG}
            """
        }
    }
}
