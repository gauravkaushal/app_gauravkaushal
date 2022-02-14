pipeline {
    agent any
    
    environment {
        scannerHome = tool name: 'sonar_scanner_dotnet'
        username = 'gauravkaushal'
        registry = 'app_gauravkaushal/app/gauravk'
        dockerHubRepo ='gauravkaushal/app_gauravkaushal'
    }
        
    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'Github', url: 'https://github.com/gauravkaushal/app_gauravkaushal.git/', branch: 'main'
            }
        }
        
        stage('Restore packages'){
            steps{
                echo "Restore nuget package for - ${BRANCH_NAME} branch"
                bat "dotnet restore ${workspace}\\nagp-devops-us.sln"
            }
        }
        
        stage('Start SonarQube Analysis'){
            when {
                branch "main"
            }
            steps{
                echo "Start SonarQube Analysis"
                withSonarQubeEnv("Test_Sonar") {
                 bat "dotnet sonarscanner begin /k:sonar-${username} -d:sonar.cs.opencover.reportsPaths=test-project/coverage.opencover.xml -d:sonar.cs.xunit.reportsPaths='test-projects/TestFileReport.xml'"   
                }
            }
        }
        
        stage('Clean Build'){
            steps{
                echo "Code clean for - ${BRANCH_NAME} branch"
                bat "dotnet clean ${workspace}\\nagp-devops-us.sln"
            }
        }
        
        stage('Build Solution'){
            steps{
                bat "dotnet build ${workspace}\\nagp-devops-us.sln -c Release"
            }
        }
        
        stage('Test Execution'){
            steps{
                bat "dotnet test ${workspace}\\test-project\\test-project.csproj /p:CollectCoverage=true /p:CoverletOutputFormat=opencover -l:trx;LogFileName=TestFileReport.xml"
            }
        }
        
        stage('Stop SonarQube Analysis'){
             when {
                branch "main"
            }
            steps{
                withSonarQubeEnv("Test_Sonar"){
                    bat "dotnet sonarscanner end"
                }
            }
        }
        
        stage ("Release artifact") {
            when {
                branch "development"
            }

            steps {
                echo "Release artifact step"
                bat "dotnet publish -c Release -o ${registry}"
            }
        }
        
        stage ("Docker Image") {
            steps {
                script {
                    if (BRANCH_NAME == "main") {
                        bat "dotnet publish -c Release -o ${registry}"
                    }
                }
                bat "docker build -t i-${username}-${BRANCH_NAME}:${BUILD_NUMBER} --no-cache -f Dockerfile ."
            }
        }
        
        stage ("Docker image push") {
            steps {
                withDockerRegistry([ credentialsId: "dockergrv-tkn", url: "" ]) {
                     bat "docker tag i-${userName}-${BRANCH_NAME}:${BUILD_NUMBER} ${dockerHubRepo}:i-${userName}-${BRANCH_NAME}-${BUILD_NUMBER}"
                     bat "docker tag i-${userName}-${BRANCH_NAME}:${BUILD_NUMBER} ${dockerHubRepo}:i-${userName}-${BRANCH_NAME}-latest"
                     bat "docker push ${dockerHubRepo}:i-${userName}-${BRANCH_NAME}-${BUILD_NUMBER}"
                     bat "docker push ${dockerHubRepo}:i-${userName}-${BRANCH_NAME}-latest"
                }
            }
        }
        
        stage ("Docker image deploy") {
            steps {                
                bat "docker run --name c-${userName}-${BRANCH_NAME} -d -p ${port}:80 ${dockerHubRepo}:i-${userName}-${BRANCH_NAME}-latest"
            }
        }
    }
}
