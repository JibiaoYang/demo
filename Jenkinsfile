pipeline {
    agent none
    options {
        // 日志保留策略，和页面配置保持一致
        buildDiscarder(logRotator(daysToKeepStr:'7', numToKeepStr:'10'))
        disableConcurrentBuilds()
        timestamps()  // 控制台输出时间戳
    }
    stages {
        stage("架构分支信息确认") {
            agent { label "Built-In Node" } // 在主节点打印参数信息
            steps {
                echo "======================================"
                echo "选定架构: ${params.BUILD_ARCH}"
                echo "选定Git分支/Tag: ${params.BUILD_BRANCH_TAG}"
                echo "======================================"
            }
        }

        stage("编译 + 测试") {
            agent {
                label {
                    switch(params.BUILD_ARCH) {
                        case "hygon":
                            return "hg"    // hygon海光架构，调度到hg节点
                        case "intel":
                            return "intel-node" //后续你创建intel节点修改此处名称
                        case "amd":
                            return "amd-node"   //后续amd节点
                        default:
                            error("不支持的架构参数：${params.BUILD_ARCH}")
                    }
                }
            }
            stages {
                stage("代码自动检出") {
                    steps {
                        echo "当前执行节点：${env.NODE_NAME}"
                        echo "开始检出分支/tag：${params.BUILD_BRANCH_TAG}"
                        // Pipeline script from SCM模式，Jenkins会自动检出对应代码，不需要手动git checkout
                    }
                }
                stage("编译构建") {
                    steps {
                        echo "===== 执行编译流程 ====="
                        // 这里替换成你实际windows编译脚本
                        // bat "build.bat"
                    }
                }
                stage("功能测试") {
                    steps {
                        echo "===== 执行测试流程 ====="
                        // bat "test.bat"
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
