pipeline{
    agent any
    tools{
        jdk 'jdk17'
        maven 'maven'
    }
    stages{
        stage('checkout'){
            steps{
                git branch: 'main', credentialsId: 'github-login', url: 'https://github.com/lucklaksh/webapp-java-maven'
            }
        }
        
        stage('compile'){
            steps{
                sh "mvn compile"
            }
        }
        
        stage('test'){
            steps{
                sh "mvn test"
            }
        }
        
        stage('filesystem scan'){
            steps{
                sh "trivy fs --format table -o trivy-fs-report.html . "
            }
        }
        /*
        
        stage('sonarqube analysis'){
            steps{
               withSonarQubeEnv('sonar') {
                    sh" mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=webapp \
                        -Dsonar.projectName='webapp' \
                        -Dsonar.host.url=http://54.87.123.157:9000 \
                        -Dsonar.token=sqp_81bd6708303a8fcb59a529e68dda06bd6d5b5578 "  
                }
            }
        }
        
        stage('Quality gate'){
            steps{
                script {
                    // Wait for SonarQube quality gate result
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "Pipeline aborted due to quality gate failure: ${qg.status}"
                    }
                }
            }
        }
        
        */
        
        stage("Build Docker Image"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker login' , toolName: 'docker') {
                        sh "docker build -t lakshmanlucky821/shop:latest ."
                    }
                }
            }
        }
       
       /* 
        stage('Image scan'){
            steps{
                sh "trivy image --format table -o trivy-fs-report.html lakshmanlucky821/shop:latest "
            }
        }
        */
        
        stage("Push Docker Image"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker login' , toolName: 'docker') {
                        sh "docker push  lakshmanlucky821/shop:latest"
                    }
                }
            }
        }
        stage("k8s deployment"){
            steps{
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', serverUrl: 'https://172.31.24.45:6443']]) {
                    sh "kubectl apply -f deployment-service.yml"
                }
            }
        }
        
        stage("Push Docker Image"){
            steps{
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', serverUrl: 'https://172.31.24.45:6443']]) {
                    sh "kubectl get pods -n webapps"
                    sh "kubectl get svc -n webapps"
                }
            }
        }
        
        
        
    }
}
