pipeline {
    agent { node {
        label "jenkins-slave"
    }}
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
                        mv Deployment/Deployment.yaml Deployment/Deployment.yaml.tmp
                        cat Deployment/Deployment.yaml.tmp | envsubst > Deployment/Deployment.yaml
                        rm -f Deployment/Deployment.yaml.tmp
                    '''
                        withCredentials( [file(credentialsId: 'gcloud-serviceaccountkey', variable: 'serviceaccountkey' )] ) {
                            sh '''
                            gcloud auth activate-service-account jenkins-auth@hightech-website.iam.gserviceaccount.com --key-file=${serviceaccountkey} --project=hightech-website
                            gcloud container clusters get-credentials hightech-website-gke --zone us-east1-b --project hightech-website
                            kubectl apply -f Deployment/ --kubeconfig ${KUBECONFIG}
                            kubectl get ingress -n hightech-website
                            '''
                        }
                        
                    }
                }   
            }
        }
    }
}