pipeline{
    envoronment{
        Project-URL = "https://github.com/RahulAggarwal-DevOps/node-todo-cicd.git"
        Image = "node-app"
        Tag = "latest"
        SQ-SCANNER = tool "sonar"
    }
    agent{
        node{
            label "dev"
        }
    }
    stages{
        stage("Code Clone"){
            steps{
                echo "Clone the code"
                git url: ${project-URL}, branch: "master"
            }
        }
        stage("Analyse the code with SonarQube"){
            steps{
                withSonarQubeEnv("sonar"){
                    sh "${SQ-SCANNER}/bin/sonar-scanner -Dsonar.projectName=NodeToDoProject" -Dsonar.projectKey=NodeToDoProject -X"
                }
            }
        }
        stage("Quality Gates with SonarQube"){
            steps{
                timeout(time: 1, unit: "MINUTES"){
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage("Scan the dependencies for Vulnerabilities, using OWASP DC, and save the Report with a Publisher"){
            steps{
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'OWASP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Code Build & Test with Docker"){
            steps{
                sh "docker build -t ${Image}:${Tag} ."
            }
        }
        stage("Scan the Docker image with Trivy"){
            steps{
                sh "trivy image ${Image}:${Tag}" 
            }
        }
        stage("Push To DockerHub"){
            steps{
                withCredentials([usernamePassword(
                    credentialsId:"dockerHubCreds",
                    usernameVariable:"dockerHubUser", 
                    passwordVariable:"dockerHubPass")]){
                sh 'echo $dockerHubPass | docker login -u $dockerHubUser --password-stdin'
                sh "docker image tag ${Image}:${Tag} ${env.dockerHubUser}/${Image}:${Tag}"
                sh "docker push ${env.dockerHubUser}/${Image}:${Tag}"
                }
            }
        }
        stage("Deploy the Application"){
            steps{
                sh "docker compose down && docker compose up -d --build"
            }
        }
    }
}
