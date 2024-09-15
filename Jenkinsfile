pipeline{
    agent{
        label 'slave-node'
    }
        stages{
            stage('Build Application'){
                steps {
                sh 'mvn -f javamavenapp/pom.xml clean package'
                }
            post{
                success{
                    echo "Now Archiving the Artifacts...."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
    }
}