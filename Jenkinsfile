pipeline {
    agent any
   stages {
       /*stage('SCM') {
           steps {
               git branch: 'devops' url: 'https://github.com/foo/bar.git'
            }
         }  */
        stage('build && SonarQube analysis') {
            steps {
                withSonarQubeEnv(installationName: 'sq1') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube'
                    
                }

                timeout(time: 2, unit: 'MINUTES') {
                  script { 
                       def qg = waitForQualityGate()
                       if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                       }
                    }   
                }
            }
        }    
        stage('docker build & push to nexus'){
            agent { label 'windows'}
            environment { 
                docker_creds = credentials('nexus-creds') 
                }
            steps {
                bat '''
                    docker build -t 192.168.0.170:8083/springboot:${env.BUILD_ID}
                    docker login -u admin -p ${docker_creds} 192.168.0.170:8083
                    docker push 192.168.0.170:8083/docker-private/springboot:${env.BUILD_ID} 
                    docker rmi 192.168.0.170:8083/springboot:${env.BUILD_ID}
                    docker logout 192.168.0.170:8083
                '''
            }    
        }
        // stage('scema validation') {
        //     agent { label 'windows'}
        //     steps {
        //         dir('kubernetes') {
        //            bat 'helm datree test myapp'
        //         }
        //     }

        // }
    }
}
