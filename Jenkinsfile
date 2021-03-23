pipeline {
  agent any
  stages {
    stage('build') {
      steps {
        sh './mvnw build -DskipTests'
      }
    }

    stage('test') {
      parallel {
        stage('test') {
          steps {
            sh './mvnw test'
          }
        }

        stage('fast') {
          steps {
            sh './mvnw test -Dgroups="fast"'
          }
        }

        stage('slow') {
          steps {
            sh './mvnw test -Dgroups="slow"'
          }
        }

      }
    }

  }
}