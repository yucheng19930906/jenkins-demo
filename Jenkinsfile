pipeline{
    agent any
    stages{
        stage('Prepare') {
            steps{
                echo "1.Prepare Stage"
                script {
                    build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                    if (env.BRANCH_NAME != 'master') {
                        build_tag = "${env.BRANCH_NAME}-${build_tag}"
                    }
            }
        }
        stage('Test') {
            steps{
                echo "2.Test Stage"
            }
        }
        stage('Build') {
            steps{ 
                echo "3.Build Docker Image Stage"
                sh "docker build -t registry.cn-shanghai.aliyuncs.com/dataany/demo:${build_tag} ."
            }
        }
        stage('Push') {
            steps{
                echo "4.Push Docker Image Stage"
                withCredentials([usernamePassword(credentialsId: 'demo-docker-user', passwordVariable: 'Password', usernameVariable: 'User')]) {
                sh "docker login -u ${User} -p ${Password} registry.cn-shanghai.aliyuncs.com"
                sh "docker push registry.cn-shangai.aliyuncs.com/dataany/demo::${build_tag}"
                }
            }
            
        }
        stage('Deploy') {
            steps{
                echo "5. Deploy Stage"
                if (env.BRANCH_NAME == 'master') {
                    input "确认要部署线上环境吗？"
                }
                sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s.yaml"
                sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s.yaml"
                sh "kubectl apply -f k8s.yaml --record"
            }
        }
}
