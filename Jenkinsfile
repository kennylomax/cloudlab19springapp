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
            stash(name: 'client', includes: '**/dist/*')
          }
        }

        stage('docker') {
          steps {
            sh 'echo "Docker file"'
            stash(name: 'dockerfile', includes: 'Dockerfile')
            sh './mvnw install -DskipTests'
            stash(includes: '**/target/*', name: 'target')
          }
        }

      }
    }

    stage('Tests') {
      parallel {
        stage('Chrome') {
          agent {
            docker {
              image 'selenium/standalone-chrome:latest'
            }

          }
          steps {
            sh '''echo \'mvn test -Dbrowser=chrome\'
'''
          }
        }

        stage('Firefox') {
          agent {
            docker {
              image 'selenium/standalone-firefox:latest'
            }

          }
          steps {
            sh 'echo \'mvn test -Dbrowser=firefox\''
          }
        }

      }
    }

    stage('QA') {
      agent {
        docker {
          image 'tomcat:8.0-jre8'
          args '-u 0:0 -p 11080:8080'
        }

      }
      steps {
        unstash 'server'
        unstash 'client'
        sh '''APP_DIR=/usr/local/tomcat/webapps
rm -rf $APP_DIR/ROOT
cp target/server.war $APP_DIR/server.war
mkdir -p $APP_DIR/ROOT
cp dist/* $APP_DIR/ROOT
/usr/local/tomcat/bin/startup.sh

'''
        input(message: 'Ok for QA?', ok: 'Go go go!')
      }
    }

    stage('Deploy') {
      steps {
        unstash 'client'
        unstash 'server'
        unstash 'dockerfile'
        sh '''echo "deploying client:"
ls -alFh dist
echo "deploying server:"
ls -alFh target'''
        sh '''echo "Deploy bild docker thingy"
docker build -t kenlomax/spring-boot-api-example:0.1.0-SNAPSHOT .
docker ps'''
        unstash 'target'
      }
    }

  }
}