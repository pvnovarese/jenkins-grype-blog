pipeline {
  environment {
    // shouldn't need REGISTRY variable unless you're using something other than dockerhub
    REGISTRY = 'registry.hub.docker.com'
    // change this CREDENTIAL to the ID of whatever jenkins credential has your registry user/pass
    CREDENTIAL = 'docker-hub'
    // set USER to your docker hub ID (or user for proviate registry)
    USER = 'pvnovarese'
    // you probably don't need to change this
    REPOSITORY = 'jenkins-grype-blog'
  }
  agent any
  stages {
    stage('Checkout SCM') {
      steps {
        checkout scm
      }
    }
    stage('Build image and tag with build number') {
      steps {
        script {
          dockerImage = docker.build USER + "/" + REPOSITORY + ":${BUILD_NUMBER}"
        }
      }
    }
    stage('Analyze with grype') {
      steps {
        // run grype with json output, use jq just to get severities, 
        // concatenate all onto one line, if we find High or Critical 
        // vulnerabilities, fail and kill the pipeline
        //
        // set -o pipefail enables the entire command to return the failure 
        // in grype and still get the count of vulnerability types
        // 
        // you can change this from "high" to "critical" if you want to see 
        // the command succeed since dvwa doesn't (as of today) have any 
        // critical vulns in it, just a bunch of highs
        //
        sh 'set -o pipefail ; /var/jenkins_home/grype -f critical -q -o json ${USER}/${REPOSITORY}:${BUILD_NUMBER} | jq .matches[].vulnerability.severity | sort | uniq -c'
      }
    }
    stage('Re-tag as prod and push stable image to registry') {
      steps {
        script {
          docker.withRegistry('', CREDENTIAL) {
            dockerImage.push('prod') 
            // dockerImage.push takes the argument as a new tag for the image before pushing
          }
        }
      }
    }
  }
}
