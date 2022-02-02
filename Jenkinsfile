pipeline {
    agent any
    
    environment {
        scannerHome = tool name: 'sonar_scanner_dotnet'
        username = 'gauravkaushal'
        registry = 'gauravkaushal/app_gauravkaushal'
    }
        
    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'Github', url: 'https://github.com/gauravkaushal/app_gauravkaushal.git/', branch: 'main'
            }
        }
        
        stage('Restore packages'){
            steps{
                bat "dotnet restore ${workspace}\\nagp-devops-us.sln"
            }
        }
        
        stage('Start SonarQube Analysis'){
            steps{
                echo "Start SonarQube Analysis"
                withSonarQubeEnv("Test_Sonar") {
                 bat "dotnet sonarscanner begin /k:sonar-${username} -d:sonar.cs.opencover.reportsPaths=test-project/coverage.opencover.xml -d:sonar.cs.xunit.reportsPaths='test-projects/TestFileReport.xml'"   
                }
            }
        }
        
        stage('Clean Build'){
            steps{
                bat "dotnet clean ${workspace}\\nagp-devops-us.sln"
            }
        }
        
        stage('Build SOlution'){
            steps{
                bat "dotnet build ${workspace}\\nagp-devops-us.sln"
            }
        }
        
        stage('Test Execution'){
            steps{
                bat "dotnet test ${workspace}\\test-project\\test-project.csproj /p:CollectCoverage=true /p:CoverletOutputFormat=opencover -l:trx;LogFileName=TestFileReport.xml"
            }
        }
        
        stage('Stop SonarQube Analysis'){
            steps{
                withSonarQubeEnv("Test_Sonar"){
                    bat "dotnet sonarscanner end"
                }
            }
        }
    }
}
