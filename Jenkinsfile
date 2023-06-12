pipeline {
    agent any
    stages {
        stage('build') {
            steps {
                script{
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    echo 'building...'
                    sh '''    
                        docker login -u ${USERNAME} -p ${PASSWORD}
                        docker build . -t andrewanter/hightech-website:v${BUILD_NUMBER}
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
                        mv Deployment/deploy.yaml Deployment/deploy.yaml.tmp
                        cat Deployment/deploy.yaml.tmp | envsubst > Deployment/deploy.yaml
                        rm -f Deployment/deploy.yaml.tmp
                        kubectl apply -f Deployment --kubeconfig ${KUBECONFIG} -n hightech-website
                        
                    '''
                    }
                }   
            }
        }
    }
}