pipeline {
    agent { label "vinod" }

    stages {
        stage("Code") {
            steps {
                echo "Cloning the repository..."
                git url: "https://github.com/Gursukhab/django_app_todo.git", branch: "main"
            }
        }

        stage("Build") {
            steps {
                echo "Building the Docker image..."
                sh "docker build -t appy:latest ."
            }
        }

        stage("Push to Docker Hub") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhubcred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        def imageName = "${DOCKER_USER}/apply:latest"
                        sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}"
                        sh "docker tag appy:latest ${imageName}"
                        sh "docker push ${imageName}"
                    }
                }
            }
        }

        stage("Deploy") {
            steps {
                script {
                    def existingContainer = sh(script: "docker ps -aq -f name=elastic_mcnulty", returnStdout: true).trim()
                    if (existingContainer) {
                        echo "Stopping & removing old container..."
                        sh "docker stop elastic_mcnulty || true"
                        sh "docker rm elastic_mcnulty || true"
                    }

                    echo "Running new container..."
                    sh "docker run -d -p 8000:8000 --name elastic_mcnulty appy:latest"

                    echo "Waiting for container to be ready..."
                    sleep 5

                    echo "Applying migrations..."
                    sh "docker exec elastic_mcnulty python manage.py makemigrations tasks || true"
                    sh "docker exec elastic_mcnulty python manage.py migrate || true"
                }
            }
        }
    }
}
