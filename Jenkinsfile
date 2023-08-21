pipeline {
    agent any
    
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
    }
}
