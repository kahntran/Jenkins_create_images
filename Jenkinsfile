pipeline {
	options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }
    post { always { deleteDir() } }
    agent any
    stages {
        stage('Deploy') {
            when { expression { params.DEPLOY_JOB_NAME == null } }
            steps {
                echo "OK"
            }
        }
        stage('Docker') {
            when { expression { params.DEPLOY_JOB_NAME != null } }
            steps {                
                script {
                    def JOB_TAG_SECTIONS = params.DEPLOY_JOB_NAME.split('/')
                    def NAME = JOB_TAG_SECTIONS[0]
                    def BRANCH = JOB_TAG_SECTIONS[1]
                    def REGISTRY_URL = "docker-registry.example.lara/${NAME}:${BRANCH}"
                    def DOCKER_ARGS = "-t ${REGISTRY_URL}"

                    if (params.ADDITIONAL_ARGUMENTS != null) {
                        DOCKER_ARGS = "${DOCKER_ARGS} ${params.ADDITIONAL_ARGUMENTS}"
                    }

                    echo """
==> Configuration
--> Names
Job name: ${params.DEPLOY_JOB_NAME}
Project name: ${NAME}
Branch: ${BRANCH}
Repository URL: ${REGISTRY_URL}
Arguments: ${params.ADDITIONAL_ARGUMENTS}

--> Directories:
Working directory: ${params.SOURCE_DIRECTORY}
                    
                    """

                    if ("${BRANCH}" =~ /^(master|develop|feature*)/) {
                        if (fileExists("${params.SOURCE_DIRECTORY}/dockerfiles/build")) {
                            echo "--> Found custom build script @ ${params.SOURCE_DIRECTORY}/dockerfiles/build"
                            echo "--> This script will have to tag and push otherwise nothing gets saved!"
                            sh "chmod +x ${params.SOURCE_DIRECTORY}/dockerfiles/build"
                            sh "${params.SOURCE_DIRECTORY}/dockerfiles/build ${params.SOURCE_DIRECTORY} ${NAME} ${BRANCH}"

                        } else if (fileExists("${params.SOURCE_DIRECTORY}/Dockerfile")) {                        
                            echo "--> Found project Dockerfile"
                            echo "--> Building image tagged ${REGISTRY_URL}"
                            sh "docker image build ${DOCKER_ARGS} ${params.SOURCE_DIRECTORY}" 
                            
                            echo "--> Pushing tag to registry ${REGISTRY_URL}"
                            sh "docker push ${REGISTRY_URL}"

                            if ("${BRANCH}" == "master") {
                                def LATEST_REGISTRY_URL = "docker-registry.example.lara/${NAME}:latest"
                                def LATEST_DOCKER_ARGS = "-t ${LATEST_REGISTRY_URL}"

                                if (params.ADDITIONAL_ARGUMENTS != null) {
                                    LATEST_DOCKER_ARGS = "${LATEST_DOCKER_ARGS} ${params.ADDITIONAL_ARGUMENTS}"
                                }

                                echo "--> Building image tagged ${LATEST_REGISTRY_URL}"                            
                                sh "docker image build ${LATEST_DOCKER_ARGS} ${params.SOURCE_DIRECTORY}" 

                                echo "--> Pushing tag to registry ${LATEST_REGISTRY_URL}"
                                sh "docker push ${LATEST_REGISTRY_URL}" 
                            }

                        } else {
                            error("No supported method of making a Docker image was found in this build")
                        }
                    } else {              
                        echo "--> ${BRANCH} is an unsupported branch name for automation."
                    }
                }
            }
        }
    }
}

