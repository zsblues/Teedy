pipeline {
    agent any

    tools {
        maven 'maven-3.9'
        jdk 'jdk11'
    }

    environment {
        MAVEN_OPTS = '-Xmx1024m -XX:MaxPermSize=256m'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo '=== 代码检出完成 ==='
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean compile -DskipTests'
                echo '=== Maven编译完成 ==='
            }
        }

        stage('PMD Code Analysis') {
            steps {
                sh 'mvn pmd:pmd'
                echo '=== PMD代码检查完成 ==='
            }
            post {
                always {
                    archiveArtifacts artifacts: '**/target/pmd.xml', allowEmptyArchive: true
                }
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn test'
                echo '=== 测试执行完成 ==='
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Generate Test Reports') {
            steps {
                sh 'mvn surefire-report:report'
                echo '=== 测试报告生成完成 ==='
            }
        }

        stage('Generate JavaDoc') {
            steps {
                sh 'mvn javadoc:aggregate'
                echo '=== JavaDoc文档生成完成 ==='
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
                echo '=== 项目打包完成 ==='
            }
        }

        stage('Archive Artifacts') {
            steps {
                echo '=== 归档构建产物 ==='
                sh '''
                    mkdir -p artifacts
                    find . -name "*.jar" -path "*/target/*" ! -name "*-sources.jar" ! -name "*-javadoc.jar" -exec cp {} artifacts/ \\;
                    find . -name "*.war" -path "*/target/*" -exec cp {} artifacts/ \\;
                    find . -name "surefire-report.html" -path "*/target/site/*" -exec cp {} artifacts/ \\;
                    if [ -d "target/site/apidocs" ]; then
                        cp -r target/site/apidocs artifacts/javadoc
                        cd artifacts/javadoc && jar cf ../javadoc.jar * && cd ../..
                    fi
                '''
                archiveArtifacts artifacts: 'artifacts/**', allowEmptyArchive: true
            }
        }
    }

    post {
        success {
            echo '流水线执行成功!'
        }
        failure {
            echo '流水线执行失败，请检查日志。'
        }
        always {
            cleanWs()
        }
    }
}
