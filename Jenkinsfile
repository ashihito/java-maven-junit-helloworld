pipeline {
    options{
        timeout(time:1,unit:'HOURS')
    }
    environment {
        docker_image_name = "java8-maven3-junit5"
    }
    
    agent {
    dockerfile {
        additionalBuildArgs '--no-cache=true --build-arg "JENKINS_USER_ID=112" --build-arg "JENKINS_GROUP_ID=117"'
        args '-v ${PWD}/.m2:/usr/share/maven/.m2'
        dir '.'
        filename 'Dockerfile'
        label env.docker_image_name
        }
    }
    stages {
        stage('maven execution') {
            steps {
                script {
                    dir('.') {
                        sh 'set HTTP_PROXY=$HTTP_PROXY'
                        sh 'set HTTPS_PROXY=$HTTP_PROXY'
                        sh 'mvn clean package site'
                    }
                }
            }
        }
        stage ('Analysis') {
            def mvnHome = tool 'mvn-default'
 
            sh "${mvnHome}/bin/mvn -batch-mode -V -U -e checkstyle:checkstyle spotbugs:spotbugs"
 
            def checkstyle = scanForIssues tool: [$class: 'CheckStyle'], pattern: '**/target/checkstyle-result.xml'
            publishIssues issues:[checkstyle]

            def spotbugs = scanForIssues tool: [$class: 'SpotBugs'], pattern: '**/target/spotbugsXml.xml'
            publishIssues issues:[spotbugs]
        }
    }

}
