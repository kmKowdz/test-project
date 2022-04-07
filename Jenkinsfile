stage('build'){
           agent {
   
               dockerfile {
                   filename 'Dockerfile'
                   dir 'build'
                   args '-v $PWD:/home/codefiles'
                           }
               }
           steps{
               sh label: '', script: 'cd /home/codefiles'

               git changelog: false, credentialsId: <git credential>, poll: false, url: <git url for the code to be deployed>
               sh label: '', script: 'ls -a'
               sh label: '', script: 'mvn package'
           }
           post {
               success {
                       archiveArtifacts artifacts: 'target/*.war', fingerprint: true
                       }
                   failure {                        
                       mail to: <email recipient>, subject: "Build Failed: ${currentBuild.fullDisplayName}",body: "This is the build ${env.BUILD_URL}"
                       cleanWs()
                       } 
               }    
           
       }

 stage('test'){
            
            steps{
                sh label:'',script:"docker container run -itd --name webdocker -p 80:8094 awsacdev/ubuntu_tomcat:1.0"
                sh label:'',script:"docker cp ${env.JENKINS_HOME}/jobs/${currentBuild.projectName}/builds/${currentBuild.number}/archive/target/. webdocker:/servers/tomcat8/webapps"
                }
            post {
                success{
                    echo 'success'
                    mail to: <email recipient>,subject: "Deployed to Test: ${currentBuild.fullDisplayName}",body: "This is the build ${env.BUILD_URL}\n Test with this URL:${env.JENKINS_URL}Devops_maven_1-1.0.0/  \n\n Abort URL: ${env.BUILD_URL}stop"
                    }
                failure{
                    mail to: <email recipient>, subject: "Failed to deploy to test: ${currentBuild.fullDisplayName}",body: "This is the build ${env.BUILD_URL}"
                    sh label:'',script:"docker container rm -f webdocker"
                    cleanWs()
                }
            }

        }

stage('Approval Step'){
            steps{
                
                //----------------send an approval prompt-------------
                script {
                   env.APPROVED_DEPLOY = input message: 'User input required',
                   parameters: [choice(name: 'Deploy?', choices: 'no\nyes', description: 'Choose "yes" if you want to deploy this build')]
                       }
                //-----------------end approval prompt------------
            }
        }

stage('deploy'){
           
            when {
                environment name:'APPROVED_DEPLOY', value: 'yes'
            }

            steps{               
                sh label:'',script:"docker container rm -f webdocker"
                echo 'deploy stage'
                git changelog: false, credentialsId: '<GIT credential>', poll: false, url: '<CHEF GIT URL>'
                sh label: '', script: "ls -a"
               
                ----------------------------------------------------------------------------------------
                ----------------------------------more code lines---------------------------------------
                ----------------------------------------------------------------------------------------
                
                //---bootstrap node
                sh label:'',script:"knife bootstrap ${env.EC2DNS} --ssh-user <USER> --sudo --yes --ssh-identity-file <KEY_FILE> --node-name prodnode --run-list 'role[prodserver]'"
                //----end bootstrap node


            }
            post{
                ----------------------------------------------------------------------------------------
                ----------------------------------more code lines---------------------------------------
                ----------------------------------------------------------------------------------------
                success{
                    mail to: '<recipient>',subject: "Deployment to PROD succeeded: ${currentBuild.fullDisplayName}",body: "This is the build ${env.BUILD_URL}. The application is live at ${env.EC2DNS}/Devops_maven_1-1.0.0/"
                    cleanWs()
                }
            }
        }
        
stage('abortdeploy'){
           when{
               environment name:'APPROVED_DEPLOY',value:'no'
           }
           steps{
               sh label:'',script:"docker container rm -f webdocker"
               mail to:'<recipient>',subject:'Deployment Aborted',body:"The deployment has been aborted. Here are the details: Project Name: ${currentBuild.projectName} Build #: ${currentBuild.number}"
               }
           post {
               always{
                   cleanWS()
               }
           }
       }