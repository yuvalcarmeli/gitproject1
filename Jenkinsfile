pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
               git branch: 'master', url: 'https://github.com/yuvalcarmeli/WorldOfGames.git'
            }
        }

        stage('Build') {
            steps {
                script {
                    sh 'docker-compose build'
                }
            }
        }
        stage('Run') {
            steps {
                script {
                    def containerName = "worldofgames-web-1"
                    def command = "docker ps -f name=${containerName} -a"
                    def commandOutput = sh(script: command, returnStdout: true).trim()
                    echo "Command: ${command}"
                    echo "Output: ${commandOutput}"

                    def isRunning = sh(script: "docker ps -q -f name=${containerName}", returnStdout: true).trim()
                    if (isRunning) {
                        echo "Container ${containerName} is running."
                        sh 'docker-compose down'
                    } else {
                        echo "Container ${containerName} is not running."
                    }
                    sh 'docker-compose up -d'
                }
                script {
                    sleep 15
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    sh '/venv/bin/python -c "import sys; print(sys.version); print(sys.executable)"'
                    dir('/var/jenkins_home/workspace/WorldOfGames/tests') {  
                        def Url = "http://host.docker.internal:8777"
                        def exitCode = sh(script: ". /venv/bin/activate && HTTP_HOST=${Url} python -c 'import e2e; e2e.main_function()'", returnStatus: true)
            
                        if (exitCode != 0) {
                            echo "Tests failed with exit code ${exitCode}"
                            currentBuild.result = 'FAILURE'  
                            error "Test execution failed." 
                        } 
                        else {
                            echo "Tests passed"
                        }
                    }
                }
            }
        }

        stage('Finalize') {
            when {
                expression {
                    currentBuild.result == null || currentBuild.result == 'SUCCESS'
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'DockerHub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_ID')])
                {
                    sh '''
                    docker-compose down
                    docker login -u $DOCKER_ID -p $DOCKER_PASSWORD
                    docker tag yuvalcarmeli/flask_project:latest $DOCKER_ID/flask_project:latest
                    docker push $DOCKER_ID/flask_project:latest'''
                }
            }
        }
    }
}


