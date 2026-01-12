pipeline {
    agent { label 'slave1' }

    tools {
        maven 'Maven-3.8.9'
        jdk 'JDk-17'
    }
    parameters {
        choice(
            name: 'buildOnly',
            choices: 'no\nyes',
            description: 'this will only build the applicaiton'
            )
        choice(
            name: 'dockerPush',
            choices: 'no\nyes',
            description: 'this will build and push the image to registry'
            )
        choice(
            name: 'deployToDev',
            choices: 'no\nyes',
            description: 'deploy to dev'
            )
        choice(
            name: 'deployToTest',
            choices: 'no\nyes',
            description: 'deploy to Test'
            )
        choice(
            name: 'deployToStage',
            choices: 'no\nyes',
            description: 'deploy to Stage'
            )
        choice(
            name: 'deployToProd',
            choices: 'no\nyes',
            description: 'deploy to Prod'
            )
    }

    environment {
        APPLICATION_NAME = "user"
        // SONAR_EUREKA2_URL = "http://54.252.136.132:9000"
        // SONAR_EUREKA2_TOKEN = credentials('eureka2_token')
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging ()     
        DOCKER_HUB = "docker.io/swarna441"   
        DOCKER_CREDS = credentials('docker_creds')
        //JFROG_DOCKER_REPO = "abc.jfrog.io"
    }

    stages {
        stage('mvnBuild') {
            when {
                anyOf {
                    expression {
                        params.buildOnly == 'yes'
                        params.dockerPush == 'yes'
                    }
                }
            }
            steps {
              script {
                buildApp().call()
              }
            }
        }
        // stage ('sonarqube'){
        //     steps{
        //         echo " starting sonar scans"
        //         //to make the sonarqube stage fail if there are code smells >3
        //         //to access 'withSonarQubeEnv' ,in UI settings---/var/lib/jenkins --------add sonarqube servers (url/token)  
        //         withSonarQubeEnv('SonarQube'){
        //              sh """
        //                 mvn clean verify sonar:sonar \
        //                     -Dsonar.projectKey=i27-eureka2 \
        //                     -Dsonar.host.url=${env.SONAR_EUREKA2_URL} \
        //                     -Dsonar.login=${env.SONAR_EUREKA2_TOKEN}
        //                 """
        //     }
        //         timeout (time: 5, unit: 'MINUTES'){
        //             script {
        //                 //to access this add in sonarqube/webhook (add jenkins master url/creds)
        //                 waitForQualityGate abortPipeline: true
        //             }
        //         }
                    
        //     }
              
        // }
        
        stage ('DockerBuildPushImage'){
             when {
                anyOf {
                    expression {
                        params.dockerPush == 'yes'
                    }
                }
            }
            steps {
                //i27-eureka2-0.0.1-SNAPSHOT.jar
               
                script {
                      dockerBuildandPush().call()
                }
                
             }
         }

         stage ('DeploytoDev'){
             when {
                anyOf {
                    expression {
                        params.deployToDev == 'yes'
                       
                    }
                }
            }
            steps {
                echo "deploy to dev"
                //sh "docker run --name ${env.APPLICATION_NAME}-dev -d -p 5761:8761 -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
                //echo "it will fail now as running the same port to create container"
               script{
                     imageValidation().call()
                    dockerDeploy('dev', '5761').call()
               } 
            }
         }
        stage ('DeploytoTest'){
            when {
                anyOf {
                    expression {
                        params.deployToTest == 'yes'
                       
                    }
                }
            }
            steps {
                echo "deploy to Test"
                
               script{
                // echo "${GIT_COMMIT}"
                //  sh "docker login -u ${env.DOCKER_CREDS_USR} -p ${env.DOCKER_CREDS_PSW}"

                //     def image = "${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
                //     def imageExists = sh(
                //             script: "docker pull ${image}",
                //             returnStatus: true
                //         ) == 0

                //     if (!imageExists) {
                //     dockerBuildandPush().call()
                //     } else{
                //     echo "image found"
                //     }
                    imageValidation().call()

                    dockerDeploy('Test', '6761').call()
               } 
            }
         } 
        stage ('DeploytoStage'){
            when {
                anyOf {
                    expression {
                        params.deployToStage == 'yes'
                       
                    }
                }
                anyOf {
                    branch 'release*'
                    tag pattern: "v\\d{1,2}\\.\\d{1,2}\\.\\d{1,2}", comparator: "REGEXP"
                }
            }
            steps {
                echo "deploy to Stage"
              
               script{
                    imageValidation().call()
                    dockerDeploy('Stage', '7761').call()
               } 
            }
         } 
         stage ('Deploytoprod'){
            when {
                anyOf {
                    expression {
                        params.deployToProd == 'yes'
                       
                    }
                }
                  anyOf {
                     tag pattern: "v\\d{1,2}\\.\\d{1,2}\\.\\d{1,2}", comparator: "REGEXP"
                }
            }
            steps {
                timeout(time: 300, unit: 'SECONDS'){
                input message : "deploying ${env.APPLICATION_NAME} to production ?", ok: 'yes' , submitter: 'sre, swarna'
            }
               script{
                    dockerDeploy('prod', '8761').call()
               } 
            }
         } 

     }
    //  post {
    //     always {
    //         mail bcc: '',
    //          body: 'this is test email',
    //          cc: '',
    //          from: '',
    //          replyTo: '',
    //          subject: '',
    //          to: 'swarna.varsha100@gmail.com'
    //     }
    //  }
 }

def dockerBuildandPush(){
    return{
        script {
               sh """                
            ls -la
                cp ${WORKSPACE}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd
                echo "existing jar format : i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"  
                """
               echo "docker build image"
               sh "docker build --no-cache --build-arg JAR_PATH=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} ./.cicd" 
                
                echo "docker login before push to dockerhub"
                sh "docker login -u ${env.DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}"

                echo "docker push to dockerhub"
                sh "docker push ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
        }
    }
    
}
def buildApp(){
    return {
                echo "building ${env.APPLICATION_NAME} applicationda"
                sh "mvn package -DskipTests=true"
                archiveArtifacts artifacts: 'target/*.jar'
    }

}

 def dockerDeploy(envDeploy, port){
    return {
      echo "deploying to $envDeploy"
      script {
                    try {
                      //stop the container 
                     sh "docker stop ${env.APPLICATION_NAME}-$envDeploy"
                    //remove the container 
                     sh "docker rm ${env.APPLICATION_NAME}-$envDeploy"
                    } catch (err) {
                        echo "error caught :$err"
                    }
                   
                    //create the conatiner again
                    sh "docker run --name ${env.APPLICATION_NAME}-$envDeploy -d -p ${port}:8232 -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
                }
    }
 }

def imageValidation(){
    return {
        try {
             sh " docker pull ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
             println("image pulled successfully")
        } catch (Exception e) {
         buildApp().call()
         dockerBuildandPush().call()
        }
       
    }
}

//mail post actions


//dev = 5761
//test=6761
//stage= 7761
//prod = 8761