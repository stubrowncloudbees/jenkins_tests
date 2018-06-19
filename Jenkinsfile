import hudson.model.*
import hudson.EnvVars
import groovy.json.JsonSlurperClassic
import groovy.json.JsonBuilder
import groovy.json.JsonOutput
import java.net.URL

currentBuild.displayName = "#" + currentBuild.number + " - " + env.BRANCH_NAME

if (env.BRANCH_NAME == "master") {
    env.ENV_SUFFIX = "mst"
} else {
    env.ENV_SUFFIX = ".qa"
}
echo "the branch name is " + env.BRANCH_NAME
env.DOCKER_VERSION = env.BRANCH_NAME.replaceAll("/", ".") + ".${BUILD_NUMBER}" + env.ENV_SUFFIX

env.DOCKER_IMAGE = "stuartcbrown/jentest:${DOCKER_VERSION}"

pipeline {
    agent { 
        node {
            label 'gen-docker' 
        }
    }


    stages {
        stage('Build'){
            steps {
                container('docker'){
                    script (){
                        docker.withRegistry('https://registry.hub.docker.com','docker_creds') {
                            sh 'rm -rf build.name'
                            sh 'ls && pwd'
                            sh "echo '$DOCKER_IMAGE' > build.name"
                            def dockerImageName = readFile('build.name').trim()
                            //def dockerImage = docker.build(dockerImageName, ".")
                            echo "Docker image nane is ${dockerImageName}"
                            
                            sh "docker build -t ${dockerImageName} ."
                            
                            //sh "docker push ${DOCKER_IMAGE}"
                            echo "running"
                           
                        }
                    }
                }
            }
        }

        stage('Push'){
            steps {
                container('docker'){
                    withDockerRegistry([ credentialsId: "docker_creds", url: "https://registry.hub.docker.com" ]) {
                        sh 'docker push stuartcbrown/jentest:master.15mst'
                    }
                    
                }
            }
        }
    }
    
    post {
        // always, unstable, aborted, failure, success, changed
        success {
            container('docker'){
                    /*script (){
                        docker.withRegistry('https://registry.hub.docker.com','docker_creds') {
                            echo "SUCCESS - about to push ${DOCKER_IMAGE}"
                            sh "docker push ${DOCKER_IMAGE}"
            
                            echo "SUCCESS - available as ${DOCKER_IMAGE}"
                            script () {
                            currentBuild.displayName = currentBuild.displayName + " ${DOCKER_VERSION}"}
                        }
                    }
                    */
               
            echo "done"    
            }
        }
        unstable {
            echo 'UNSTABLE'
        }
        failure {
            echo 'FAILED'
            sh "docker rmi ${DOCKER_IMAGE}"
        }
        always {
            echo 'DONE'
        }
    }
}
