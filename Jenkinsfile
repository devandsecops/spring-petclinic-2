
pipeline{
    agent any

    stages{
        stage("Build"){
            when {
                branch 'develop'    
            }
            steps{
              {
                   sh "mvn clean install -DskipTests"
               }
            }
        }
        stage("Unit Tests"){
            when {
                branch 'develop'   
            }
            steps{
               sh "mvn test"
            }
        }
        stage("Integration test"){
            when {
                branch 'develop'   
            }
            steps{
               sh "mvn verify -DskipUnitTests"

            }
        }
        stage ("static code analysis"){
            when {
                branch 'develop'   
            }
            steps {
                sh "mvn checkstyle:checkstyle"
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }
         stage('CODE ANALYSIS with SONARQUBE') {
            when {
                branch 'develop'   
            }
            environment {
             scannerHome = tool 'sonarscanner4'
            }
            steps {
                withSonarQubeEnv('sonar-qube') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=petclinic \
                   -Dsonar.projectName=petclinic \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/classes/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''

                // we ferform code analysis using sonarqube by passing the unit test cases result, checkstyle result for uploading and doing code analysis
              }
            timeout(time: 10, unit: 'MINUTES') {
               waitForQualityGate abortPipeline: true
               // here we wait for quality gates from sonar server
            }
          }
        }

        stage ("docker build") {
            when {
                branch 'develop'   
            }
            steps{
                sh "sudo docker build -t sagarppatil27041992/main:'${env.BUILD_NUMBER}' ."
                // we build the docker image of our apllication and tageed that image with build no env variable
            }
        }
        stage('Docker Publish') {
            when {
                branch 'develop'   
            }
            steps {
               withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                   sh "sudo docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                   // make a login to docker hub registry using docker credential
                   sh "sudo docker push sagarppatil27041992/main:'${env.BUILD_NUMBER}' "
                // we push the docker image to dockerhub registry that we build in privious steps 
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'develop'   
            }
            steps {
               withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                   sh "sudo docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                   sh "sudo docker run -d --name java-app-main-${env.BUILD_NUMBER}  -p 3000:8080 sagarppatil27041992/main:'${env.BUILD_NUMBER}' "                // we push the docker image to dockerhub registry that we build in privious steps 
                // we run the docker imaage  that we build in privious steps 

                }
            }
        }
    }

}
