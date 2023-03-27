pipeline { 
    agent any 
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') { 
            steps { 
                echo 'make' 
            }
        }
        stage('Test'){
            steps {
                echo 'make check'
                echo 'reports/**/*.xml' 
            }
        }
        stage('Deploy') {
            steps {
                input 'Does the staging environment look OK?'
                milestone(1)
                echo 'make publish'
            }
        }
    }
}
