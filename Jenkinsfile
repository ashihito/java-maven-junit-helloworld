pipeline {
    options{
        timeout(time:1,unit:'HOURS')
    }
    environment {
        docker_image_name = "java8-maven3-junit5"
        reportDir = 'build/reports'
        javaDir = 'src/main/java'
        resourcesDir = 'src/main/resources'
        testReportDir = 'build/test-results/test'
        jacocoReportDir = 'build/jacoco' 
        javadocDir = 'build/docs/javadoc'
        libsDir = 'build/libs'
        appName = 'SampleApp'
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
            steps {
                parallel(
                    'test exec' : {
                        sh 'mvn test'

                        dir(reportDir) {
                            step([
                                $class: 'CheckStylePublisher',
                                pattern: "checkstyle/*.xml"
                            ])
                            step([
                                $class: 'FindBugsPublisher',
                                pattern: "findbugs/*.xml"
                            ])
                            step([
                                $class: 'PmdPublisher',
                                pattern: "pmd/*.xml"
                            ])
                            step([
                                $class: 'DryPublisher',
                                pattern: "cpd/*.xml"
                            ])

                            archiveArtifacts "checkstyle/*.xml"
                            archiveArtifacts "findbugs/*.xml"
                            archiveArtifacts "pmd/*.xml"
                            archiveArtifacts "cpd/*.xml"
                        }
                    },
                    'file': {
                        stepcounter outputFile: 'stepcount.xls', outputFormat: 'excel', settings: [
                            [key:'Java', filePattern: "${javaDir}/**/*.java"],
                            [key:'SQL', filePattern: "${resourcesDir}/**/*.sql"],
                            [key:'HTML', filePattern: "${resourcesDir}/**/*.html"],
                            [key:'JS', filePattern: "${resourcesDir}/**/*.js"],
                            [key:'CSS', filePattern: "${resourcesDir}/**/*.css"]
                        ]
                        archiveArtifacts "stepcount.xls"
                    }
                )
            }
        }
    }

}
