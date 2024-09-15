pipeline{
    agent{
        label 'slave-node'
    }
        stages{
            stage('Build Application'){
                steps {
                sh 'mvn -f pom.xml clean package'
                }
            post{
                success{
                    echo "Now Archiving the Artifacts...."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
            }
            stage("Deploying Tomcat Image"){
                agent{
                    label 'slave-node'
                }
                steps{
                    copyArtifacts filter: '**/*.war', fingerprintArtifacts: true, projectName: env.JOB_NAME, selector: specific(env.BUILD_NUMBER)
                    echo "Building docker image"
                    sh '''
                    original_pwd=$(pwd -P)
                    docker build -t localtomcatimg:$BUILD_NUMBER .
                    cd $original_pwd
                    sh '''
                }
            }
    }
}
