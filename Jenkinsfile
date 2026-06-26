pipeline {

    agent {
        label 'DevServer'
    }

    parameters {
        choice(
            name: 'select_environment',
            choices: ['dev', 'prod'],
            description: 'Select deployment environment'
        )
    }

    environment {
        NAME = 'Lalit'
    }

    tools {
        maven 'My Maven'
    }

    stages {

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }

        stage('Test') {
            parallel {

                stage('Test A') {
                    agent {
                        label 'DevServer'
                    }
                    steps {
                        echo "Running Test A"
                        sh 'mvn test'
                    }
                }

                stage('Test B') {
                    agent {
                        label 'DevServer'
                    }
                    steps {
                        echo "Running Test B"
                        sh 'mvn test'
                    }
                }
            }

            post {
                success {
                    sh '''
                        echo "Build artifacts:"
                        ls -l webapp/target
                    '''

                    dir('webapp/target') {
                        stash name: 'maven-build', includes: '*.war'
                    }
                }
            }
        }

        stage('Deploy to Dev') {

            when {
                beforeAgent true
                expression {
                    params.select_environment == 'dev'
                }
            }

            agent {
                label 'DevServer'
            }

            steps {

                // Unstash in Jenkins workspace
                unstash 'maven-build'

                sh '''
                    echo "Current Workspace:"
                    pwd

                    echo "Workspace Files:"
                    ls -l

                    echo "Cleaning deployment directory..."
                    rm -rf /var/www/html/*

                    echo "Copying WAR..."
                    cp *.war /var/www/html/webapp.war

                    echo "Extracting WAR..."
                    cd /var/www/html
                    jar -xvf webapp.war

                    echo "Removing WAR..."
                    rm -f webapp.war

                    echo "Deployment Successful!"
                '''
            }
        }

    }

    post {

        success {
            echo "Pipeline completed successfully."
        }

        failure {
            echo "Pipeline failed."
        }
    }
}