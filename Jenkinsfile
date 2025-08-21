pipeline {
    agent any
    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branch to build')
    }

    environment {
        APPLICATION_NAME = 'python-app'  
        GIT_REPO = 'git@github.com:decoder-cicd-demo/ocp-sample-flask-s2i.git'  
        GIT_BRANCH = 'main'
        PROJECT_NAME = 'crt-openshiftdemo-dev'  // Your OpenShift project
        TEMPLATE_NAME = 'python-app' 
        PORT = 8080  
    }
    stages {
        // stage('Get Latest Code') {
        //     steps {
        //         git branch: "${GIT_BRANCH}", url: "${GIT_REPO}", credentialsId: 'example'
        //     }
        // }
        // stage('Install Dependencies') {
        //     steps {
        //         sh '''
        //         python -m venv venv
        //         . venv/bin/activate
        //         pip install --upgrade pip
        //         pip install -r requirements.txt
        //         deactivate
        //         '''
        //     }
        // }
        stage('Create Image Builder') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject("${PROJECT_NAME}") {
                            if (openshift.selector("bc", "${TEMPLATE_NAME}").exists()) {
                                openshift.selector("bc", "${TEMPLATE_NAME}").delete()
                            }
                            if (openshift.selector("is", "${TEMPLATE_NAME}").exists()) {
                                openshift.selector("is", "${TEMPLATE_NAME}").delete()
                            }
                            openshift.newBuild("openshift/python:3.12-ubi9~${GIT_REPO}", "--name=${TEMPLATE_NAME}")
                        
                        }
                    }
                    sh "oc set build-secret --source bc/${TEMPLATE_NAME} github -n ${PROJECT_NAME}"
                }
            }
        }
        stage('Build Image') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject("${PROJECT_NAME}") {
                            openshift.selector("bc", "${TEMPLATE_NAME}").startBuild("--wait=true")
                        }
                    }
                }
            }
        }
        stage('Deploy to Project') {
            steps {
                sh """
                oc delete deployment/${TEMPLATE_NAME} svc/${TEMPLATE_NAME} route/${TEMPLATE_NAME} -n ${PROJECT_NAME} --ignore-not-found=true
                oc new-app ${TEMPLATE_NAME}:latest -n ${PROJECT_NAME}
                oc expose svc/${TEMPLATE_NAME} --port=${PORT} -n ${PROJECT_NAME}
                """
                script {
                    openshift.withCluster() {
                        openshift.withProject("${PROJECT_NAME}") {
                            def deployment = openshift.selector("deployment", "${TEMPLATE_NAME}")
                            deployment.rollout().status()
                        }
                    }
                }
            }
        }
        stage('Scale Deployment') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject("${PROJECT_NAME}") {
                            openshift.selector('deployment', "${TEMPLATE_NAME}").scale('--replicas=1')  
                        }
                    }
                }
            }
        }
    }
}
