pipeline {
    agent any
    
    environment{
        sonar = tool name: 'sonar_msbuild'
        registry = 'shivamsahni/basicmath'
        properties = null
        docker_port = null
        username = 'shivamsahni'
    }
    
    options {
      timestamps()
      timeout(activity: true, time: 1)
    }
    
    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/shivamsahni/dotnetsample4devops.git'
            }
        }        
        stage('restore') {
            steps {
                echo "Build ${JOB_NAME} number: ${BUILD_NUMBER}"
                echo 'Restore nuget Packages'
                bat 'dotnet restore'
            }
        }
        stage('SonarQube Start') {
            steps {
                echo 'SonarQube Analysis'
                withSonarQubeEnv('sonar'){
                    bat "${sonar}\\SonarScanner.MSBuild.exe begin /k:shivamnagp2 /n:shivamnagp /v:1.0"
                }
            }
        }
        stage('clean') {
            steps {
                echo 'Clean before build'
                bat 'dotnet clean'
            }
        }
        stage('build') {
            steps {
                echo 'Build Code'
                bat 'dotnet build'
            }
        } 
        stage('Automated Unit Testing') {
            steps {
                echo 'Run Unit Tests'
                bat 'dotnet test BasicMathTests\\BasicMathTests.csproj -l:trx;LogFileName=BasicMathTestResults.xml'
            }
        }
        stage('SonarQube Stop') {
            steps {
                echo 'Stop SonarQube Analysis'
                withSonarQubeEnv('sonar'){
                    bat "${sonar}\\SonarScanner.MSBuild.exe end"
                }
            }
        }
        stage('Create Docker Image'){
            steps{
                echo "Docker Image creation step"
                bat "dotnet publish -c Release"
                bat "docker build -t localimageof_${username} --no-cache -f Dockerfile ."
            }
        }
        stage('Move Image to Docker Hub'){
            steps{
                echo "Move Image to a registry, Docker Hub"
                bat "docker tag localimageof_${username} ${registry}:${BUILD_NUMBER}"
                withDockerRegistry([credentialsId: 'DockerHub', url: ""]){
                    bat "docker push ${registry}:${BUILD_NUMBER}"
                }
            }
        }
        stage('Docker Deployment'){
            steps{
                echo "Docker Deployment by using docker hub's image"
                bat "docker run --name basicmathapp ${registry}:${BUILD_NUMBER}"
            }
        }
        
    }
    post{
        always{
            echo 'Test Report Generation...'
            xunit([MSTest(deleteOutputFiles: true, failIfNotNew: true, pattern: 'BasicMathTests\\TestResults\\BasicMathTestResults.xml', skipNoTestFiles: true, stopProcessingIfError: true)])
        }
        
    }
}
