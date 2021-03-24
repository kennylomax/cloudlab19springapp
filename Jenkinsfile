pipeline {
  agent any
  stages {
    stage('Build') {
      parallel {
        stage('Server') {
          agent {
            docker {
              image 'maven:3.5-jdk-8-slim'
            }

          }
          steps {
            sh '''echo "Building server"
mvn -version
mkdir -p target
touch "target/server.war"'''
            stash(name: 'server', includes: '**/*.war')
          }
        }

        stage('Client') {
          agent {
            docker {
              image 'node:6'
            }

          }
          steps {
            sh '''echo "Building client"
npm install --save react
mkdir -p dist
cat > dist/index.html << EOF
hello!
EOF
touch "dist/client.js"
'''
          }
        }

      }
    }

  }
}