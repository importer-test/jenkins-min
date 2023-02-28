pipeline {
    agent { label 'worker_node1' }
    environment {
        STAGE_VERSION = "0.0.${BUILD_NUMBER}"
        RC_VERSION = "1.0.${BUILD_NUMBER}"
    }
    stages {
        stage('Source') {
            steps {
               cleanWs()
               checkout scm
            }
        }
        stage('Compile') {
            tools {
                gradle 'gradle5'
            }
            steps {
                sh 'gradle -PSTAGE_VERSION=$STAGE_VERSION clean compileJava assemble'
                stash includes: '**/web*.war', name: 'roar'
            }
        }
        stage('Test') {
            steps {
                unstash 'roar'
                git branch: 'test', url: 'https://github.com/importer-test/jenkins-min'
                sh "chmod +x ./test-script.sh"
                sh "./test-script.sh test1 test2"
              }
        }
        stage('Package') {
            steps {
                unstash 'roar'
                git branch: 'test', url: 'https://github.com/importer-test/jenkins-min'
                sh "docker build -f Dockerfile_roar_db_image -t localhost:5000/roar-db:${STAGE_VERSION} ."
                sh "docker build -f Dockerfile_roar_web_image --build-arg warFile=web/build/libs/web-${STAGE_VERSION}*.war -t localhost:5000/roar-web:${STAGE_VERSION} . "
                sh "docker push localhost:5000/roar-db:${STAGE_VERSION}"
                sh "docker push localhost:5000/roar-web:${STAGE_VERSION}"
            }
        }
    }
}
