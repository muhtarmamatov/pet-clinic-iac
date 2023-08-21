pipeline {
    agent any

    triggers {
        pollSCM('H/2 * * * *')
    }

    environment {
        GRADLE_VERSION = '8.1' // Set the desired Gradle version here
        JAVA_VERSION = 'Java17'
        MAVEN_VERSION = '3.3.9'
    }

    
    tools {
        gradle "${GRADLE_VERSION}"
        jdk "${JAVA_VERSION}"
        maven "${MAVEN_VERSION}"
    }
    
    stages {
        stage('Check Tools') {
            steps {
                script {
                    def requiredTools = ['mvn', 'gradle', 'docker', 'java']
                    def missingTools = []
                    
                    for (tool in requiredTools) {
                        def toolExists = sh(script: "command -v $tool", returnStatus: true) == 0
                        if (!toolExists) {
                            missingTools.add(tool)
                        }
                    }
                    
                    if (missingTools) {
                        error "Required tools not found: ${missingTools.join(', ')}"
                    } else {
                        echo "All required tools are installed."
                    }
                }
            }
        }
        stage('Checkout Pet Clinic application') {
            steps {
                // Clean workspace before checking out code
                deleteDir()
                checkout(
                        scm: [
                            $class: 'GitSCM',
                            branches: [
                                [name: '*/main'],
                                [name: '*/Development'],
                                [name: '*/Test'],
                                [name: '*/Production']
                            ],
                            userRemoteConfigs: [
                                [
                                    credentialsId: 'jenkins_ci.hl.airmanas.llc',
                                    url: 'git@github.com:muhtarmamatov/spring-petclinic.git'
                                ]
                            ]
                        ]
                    )
            }
        }
    }
}
