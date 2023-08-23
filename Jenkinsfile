pipeline {
    agent any

    triggers {
        pollSCM('H/2 * * * *')
    }

    environment {
        GRADLE_VERSION = '8.1' // Set the desired Gradle version here
        JAVA_VERSION = 'Java17'
        MAVEN_VERSION = '3.9.4'
        FULL_IMAGE_NAME = ''
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
        stage('Unit Test and Integration Test Pet Clinic') {
            steps {
                script {
                    // Run Maven tests in the workspace directory
                    sh "mvn test -X"
                }
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml' // Specify the path to your test result XML files
                }
            }
        }
        stage('Build Pet Clinic Application Docker Image') {
                    steps {
                        script {
                            def imageName = "muktarbek/pet-clinic"
                            def dockerfile = "Dockerfile"

                            // Run Maven to get project version
                            def version = sh(script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true).trim()

                            // Extract version without '-SNAPSHOT' using substring
                            def extract_version = version.substring(0, version.indexOf('-'))

                            // Create image tag using extracted version and BUILD_NUMBER
                            def imageTag = "${extract_version}.${env.BUILD_NUMBER}"

                            def fullImageName = "${imageName}:${imageTag}"

                            echo "Tag extracted version: ${extract_version}"
                            echo "Docker image tag name: ${fullImageName}"

                            FULL_IMAGE_NAME = fullImageName
                            // Build Docker image and tag with version
                            docker.build(fullImageName, "-f ${dockerfile} .")
                        }
                    }
        }
        stage("Pushing docker image to Docker hub") {
            steps {
                script {
                    withDockerRegistry(
                        credentialsId: "docker hub login password", // Replace with your credentials ID
                        url: "https://index.docker.io/v1/"
                    ) {
                        docker.image(env.FULL_IMAGE_NAME).push()
                    }
                }
            }
        }

    }
}
