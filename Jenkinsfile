pipeline {
    agent { nodes {
        labels "jenkins-slave"
    }}
    stages {
        stage('build') {
            steps {
                script{
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    echo 'building...'
                    sh '''    
                        docker login -u ${USERNAME} -p ${PASSWORD}
                        sudo docker build . -t andrewanter/hightech-website:v${BUILD_NUMBER}
                        docker push andrewanter/hightech-website:v${BUILD_NUMBER}
                        echo ${BUILD_NUMBER} > ../build.txt
                    '''
                    }
                }
                
            }
        }
        
        stage('deploy') {
            steps {
                script{
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    echo 'deploying to localhost'
                    sh '''
                        export BUILD_NUMBER=$(cat ../build.txt)
                        mv Deployment/Deployment.yaml Deployment/Deployment.yaml.tmp
                        cat Deployment/Deployment.yaml.tmp | envsubst > Deployment/Deployment.yaml
                        rm -f Deployment/Deployment.yaml.tmp
                        kubectl apply -f Deployment --kubeconfig ${KUBECONFIG} -n hightech-website
                        
                    '''
                    }
                }   
            }
        }
    }
}