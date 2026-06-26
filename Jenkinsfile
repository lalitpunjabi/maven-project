pipeline {

    agent {
        label 'DevServer'
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    parameters {
        choice(
            name: 'select_environment',
            choices: ['dev', 'prod'],
            description: 'Select Deployment Environment'
        )
    }

    environment {
        NAME = "Lalit"
        DEPLOY_DIR = "/var/www/html"
    }

    tools {
        maven 'My Maven'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Load Groovy Script') {
            steps {
                script {
                    def util = load "script.groovy"
                    util.hello()
                }
            }
        }

        stage('Build') {
            steps {
                sh '''
                    echo "Building Application..."
                    mvn clean package -DskipTests=true
                '''
            }
        }

        stage('Test') {
            parallel {

                stage('Test A') {
                    steps {
                        sh '''
                            echo "Running Test A"
                            mvn test
                        '''
                    }
                }

                stage('Test B') {
                    steps {
                        sh '''
                            echo "Running Test B"
                            mvn test
                        '''
                    }
                }

            }

            post {
                success {

                    sh '''
                        echo "Artifacts Generated:"
                        ls -lh webapp/target
                    '''

                    dir('webapp/target') {
                        stash name: 'maven-build', includes: '*.war'
                    }
                }
            }
        }

        stage('Deploy to Development') {

            when {
                beforeAgent true
                expression {
                    params.select_environment == 'dev'
                }
            }

            steps {

                unstash 'maven-build'

                sh '''
                    echo "Current Workspace"
                    pwd

                    echo "Workspace Files"
                    ls -lh

                    WAR_FILE=$(ls *.war)

                    echo "WAR File = $WAR_FILE"

                    echo "Cleaning Deployment Directory..."
                    rm -rf ${DEPLOY_DIR}/*

                    echo "Copying WAR..."
                    cp "$WAR_FILE" ${DEPLOY_DIR}/

                    cd ${DEPLOY_DIR}

                    echo "Extracting WAR..."
                    jar -xvf "$WAR_FILE"

                    rm -f "$WAR_FILE"

                    echo "Development Deployment Successful"
                '''
            }
        }

        stage('Production Approval') {

            when {
                beforeAgent true
                expression {
                    params.select_environment == 'prod'
                }
            }

            steps {
                timeout(time: 5, unit: 'DAYS') {
                    input(
                        message: 'Deploy application to Production?',
                        ok: 'Deploy'
                    )
                }
            }
        }

        stage('Deploy to Production') {

            when {
                beforeAgent true
                expression {
                    params.select_environment == 'prod'
                }
            }

            agent {
                label 'ProdServer'
            }

            steps {

                unstash 'maven-build'

                sh '''
                    echo "Workspace"
                    pwd

                    ls -lh

                    WAR_FILE=$(ls *.war)

                    echo "WAR File = $WAR_FILE"

                    rm -rf ${DEPLOY_DIR}/*

                    cp "$WAR_FILE" ${DEPLOY_DIR}/

                    cd ${DEPLOY_DIR}

                    jar -xvf "$WAR_FILE"

                    rm -f "$WAR_FILE"

                    echo "Production Deployment Successful"
                '''
            }
        }
    }

    post {

        always {
            cleanWs()
        }

        success {
            echo "Pipeline completed successfully."
        }

        failure {
            echo "Pipeline failed."
        }

        unstable {
            echo "Pipeline is unstable."
        }

        aborted {
            echo "Pipeline was aborted."
        }
    }
}