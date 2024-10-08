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
            stage("Creating Tomcat image."){
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
            stage('Deploying it to staging environment'){
                agent{
                    label 'slave-node'
                }
                steps{
                    echo "Running app on staging ENV"
                    sh '''
                    docker stop tomcatInstanceStaging || true
                    docker rm tomcatInstanceStaging || true
                    docker run -itd --name tomcatInstanceStaging -p 8084:8080 localtomcatimg:$BUILD_NUMBER
                    sh '''
                }
                //rename the warfilename in dockerfile to root directory.
            }
            stage('Deploying in Production Environment'){
                agent{
                    label 'slave-node'
                }
                steps{
                    timeout(time:1 ,unit:'DAYS'){
                        input message:'Approve production Deployment?'
                    }
                    sh '''
                        docker stop tomcatInstanceStaging || true
                        docker rm tomcatInstanceStaging || true
                        docker run -itd --name tomcatInstanceStaging -p 8084:8080 localtomcatimg:$BUILD_NUMBER
                    sh '''
                }
            }
        }

        post{
            always { 
            mail to: 'aluii07sh@gmail.com',
            subject: "Job '${JOB_NAME}' (${BUILD_NUMBER}) is waiting for input",
            body: "Please go to ${BUILD_URL} and verify the build"
            }
            success {
                mail bcc: '', body: """Hi Team,
                Build #$BUILD_NUMBER is successful, please go through the url

                $BUILD_URL
                and verify the details.

                Regards,
                    DevOps Team""", cc: '', from: '', replyTo: '', subject: 'BUILD SUCCESS NOTIFICATION', to: 'devopsuryaraj@gmail.com'
                }
                
            failure {
                mail bcc: '', body: """Hi Team,            
                Build #$BUILD_NUMBER is unsuccessful, please go through the url
                $BUILD_URL
                and verify the details.
                Regards,
                DevOps Team""", cc: '', from: '', replyTo: '', subject: 'BUILD FAILED NOTIFICATION', to: 'aluii07sh@gmail.com'
            }
        }
}
