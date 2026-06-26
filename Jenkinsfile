pipeline
{

agent {
  label 'DevServer'
}

tools {
  maven 'My Maven'
}

stages{

    stage('build')
    {
        steps {
            sh 'mvn clean package'
        }

        post {
            success {
                archiveArtifacts artifacts: '**/target/*.war'
            }
        }

    }

}

}