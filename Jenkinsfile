pipeline{
    agent none

    environment{
        REGISTRY_NAME = 'moorthy265'
        BUILD_NO = "${BUILD_NUMBER}"
    }

    stages{
        stage("Checkout"){
            agent{
                label 'ec2-build-runner'
            }
            steps{
                checkout([$class: 'GitSCM', 
                    branches: [[name: '*/main']], 
                    extensions: [], 
                    userRemoteConfigs: [[
                        credentialsId: 'Gitlogin', 
                        url: 'https://github.com/Moorthy-M/k8s-helm-gitops-project' 
                    ]]
                ])
            }
        }

        stage("Code Lint"){
            agent{
                label 'ec2-build-runner'
            }
            steps{
                dir('src'){
                    sh 'pnpm install --frozen-lockfile'
                    sh 'pnpm -r --if-present lint'
                }
            }
        }
                
        stage("Unit Test"){
            steps{
                dir('src') {
                    sh 'pnpm test'
                }
            }
        }

        stage("Sonar Scanner"){
            agent{
                label 'ec2-build-runner'
            }
            steps {
                withSonarQubeEnv('SonarQube-Scan') {
                    sh 'sonar-scanner'
                }
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Image Build"){
            steps{
                dir('src') {
                    sh "docker build -t ${REGISTRY_NAME}/docmost:${BUILD_NO}"
                }
            }
        }
    }
}