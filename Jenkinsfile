pipeline {
    agent {
        label 'AGENT-1'
    }
    options{
        disableConcurrentBuilds()
        timeout(time:10, unit: 'MINUTES')
    }
    environment{
        appVersion = ''
        region = 'us-east-1'
        account_id = '537124943253'
        project = 'expense'
        environment = 'dev'
        component  = 'frontend'
    }

    stages {
        stage('Read the Version') {
            steps {
               script{      //script means groovy scripts
                def packageJson = readJSON file: 'package.json'
                appVersion = packageJson.version
                echo "App version: ${appVersion}"
               }
            }
        }
        // stage('NodeJs'){
        //     steps{
        //         sh """
        //         sudo dnf module disable nodejs -y
        //         sudo dnf module enable nodejs:20 -y
        //         sudo dnf install nodejs -y
        //         """
        //     }
        // }
        stage('Install dependencies') {
            steps {
               sh 'npm install'
            }
        }
        stage('Build docker image') {
            steps {                         //dot means current directory
                // withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                //     sh """
                //     docker build -t kdprasad028/backend:${appVersion} .
                //     docker images
                //     """
                // }
                withAWS(region: 'us-east-1', credentials: 'aws-creds') {    //please ADD credentials in jenkins site
                    sh """
                    aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.us-east-1.amazonaws.com

                    docker build -t ${account_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${environment}/${component}:${appVersion} .

                    docker images

                    docker push ${account_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${environment}/${component}:${appVersion}

                    """
                }
            }
        }
        stage('Deploy $component'){
            steps{
                withAWS(region: 'us-east-1', credentials: 'aws-creds'){
                    sh """
                    aws eks update-kubeconfig --region ${region} --name ${project}-${environment}

                    cd helm     

                    sed -i 's/IMAGE_VERSION/${appVersion}/g' values-${environment}.yaml 

                    helm upgrade --install ${component} -n ${project} -f values-${environment}.yaml .  

                    """
                    //dot means current folder of Dockerfile exists
                }
            }
        }
    }
    post {
        always{
            echo "This is ALWAYS section, always say hello"
            deleteDir()
        }

        success{
            echo "This section print when pipeline is success, I am SUCCESS"
        }

        failure{
            echo "this section failed when pipeline is FAILED. I am FAIL"
        }
    }
}