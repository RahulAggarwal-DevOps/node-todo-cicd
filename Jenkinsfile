pipeline{
    environment{
        Project_URL = "https://github.com/RahulAggarwal-DevOps/node-todo-cicd.git"
        Image = "node-app"
        Tag = "latest"
        SQ_SCANNER = tool "sonar"
    }
    agent{
        node{
            label "dev"
        }
    }
    stages{
        stage("Code Clone"){
            steps{
                echo "Clone the code "
                git url: "${Project_URL}", branch: "master"
            }
        }
        stage("Analyse the code with SonarQube"){
            steps{
                withSonarQubeEnv("sonar"){
                    sh "${SQ_SCANNER}/bin/sonar-scanner -Dsonar.projectName=NodeToDoProject -Dsonar.projectKey=NodeToDoProject -X"
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
                sh "whoami"
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
                    credentialsId:"DockerHubCreds",
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
                sh "docker compose down"
                sh "docker compose up -d --build"
            }
        }
    }
}
