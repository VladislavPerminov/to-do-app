pipeline {
    agent {
        label 'agent_node'
    }
    
    environment {
        PAT_DOCKERHUB = credentials('PAT_Dockerhub')
    }
 
    stages {
        stage('Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/VladislavPerminov/to-do-app.git'
            }
        }
        stage('Build') {
            steps {
                sh '''
                    npm install
                    npm run build
                '''
            }
        }
        stage('Test') {
            steps {
                sh 'npm run test'
            }
        }
        
        stage('Scan') {
            steps { 
                withSonarQubeEnv(installationName: 'sq1') {
                    sh ''' sonar-scanner \
                        -Dsonar.projectKey=to-do-app \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://172.22.0.5:9000 \
                        -Dsonar.token=sqp_6d596aea1ec6b8ae1d1990917f56b04cda2386af 
                        '''
                }
            }
        }
        
        stage('QualiteGate') {
            steps{
                timeout(time:4, unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: true 
                }
            }    
        }
        
        stage('Delivery') {
            steps {
                sh '''
                    docker login -u steppenwol -p ${PAT_DOCKERHUB}
                    docker build . -t steppenwol/to-do-app:${BUILD_ID} -t steppenwol/to-do-app:latest
                    docker push steppenwol/to-do-app:${BUILD_ID} && docker push steppenwol/to-do-app:latest
                   '''
            }
        }
    }
    
    post {
        failure{
            mail bcc: '', body: 'pas de chance', cc: '', from: '', replyTo: '', subject: 'Fail', to: 'admin@admin.com'
          }
        success{
            mail bcc: '', body: 'bravo', cc: '', from: '', replyTo: '', subject: 'Sucess', to: 'admin@admin.com'
           }
        }
}