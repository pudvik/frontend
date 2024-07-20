pipeline {
    agent {
        label 'AGENT-1'
    }
    options {
        timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    
    environment {
        def appVersion = ''
        nexusUrl = '35.153.142.179:8081'
        region = 'us-east-1'
        account_id = '798283511816'


    }
    stages {
        stage('read the version') {
            steps {
                script {
                    def packagejson = readJSON file: 'package.json'
                    appVersion = packagejson.version
                    echo "applicatio version : $appVersion"

                }
                
            }
        }
        
        stage('build') {
            steps {
                sh """
                
                zip -q -r frontend-${appVersion}.zip * -x Jenkinsfile -x frontend-${appVersion}.zip
                ls -ltr
                """
                
            }
        }

        stage('docker build'){
            steps{
                sh """
                    aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.${region}.amazonaws.com

                    docker build -t ${account_id}.dkr.ecr.${region}.amazonaws.com/expense-frontend:${appVersion} .

                    docker push ${account_id}.dkr.ecr.${region}.amazonaws.com/expense-frontend:${appVersion}
                """
            }
        }

        stage('deploy'){
            steps{
                sh """
                    aws eks update-kubeconfig --region us-east-1 --name expense-dev
                    cd helm
                    sed -i 's/IMAGE_VERSION/${appVersion}/g' values.yaml
                    helm upgrade frontend .
                """
            }
        }

        /* stage('Nexus Artifact Upload'){
            steps{
                script{
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${nexusUrl}",
                        groupId: 'com.expense',
                        version: "${appVersion}",
                        repository: "frontend",
                        credentialsId: 'nexus-auth',
                        artifacts: [
                            [artifactId: "frontend" ,
                            classifier: '',
                            file: "frontend-" + "${appVersion}" + '.zip',
                            type: 'zip']
                        ]
                    )
                }
            }
        } */

        // stage('Deploy'){
        //     steps{
        //         script{
        //             def params = [
        //                 string(name: 'appVersion', value: "${appVersion}")
        //             ]
        //             build job: 'frontend-deploy', parameters: params, wait: false
        //         }
        //     }
        // }


        
        
        
    }

    post { 
        always { 
            echo 'I will always say Hello again!'
        }
        success { 
            echo 'I will run when pipeline is success'
        }
        failure { 
            echo 'I will run when pipeline is failure'
        }
    }
}