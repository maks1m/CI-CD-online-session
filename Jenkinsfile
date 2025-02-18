pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        script {
          checkout scm
          def customImage = docker.build("${registry}:${env.BUILD_ID}")
        }

      }
    }

    stage('unit-test') {
      steps {
        script {
          docker.image("${registry}:${env.BUILD_ID}").inside {c ->
          sh 'python app_test.py'
        }
      }

    }
  }

  stage('http-test') {
    steps {
      script {
        docker.image("${registry}:${env.BUILD_ID}").withRun('-p 9005:9000') {c ->
        sh "sleep 5; curl -i http://localhost:9005/test_string"
      }
    }

  }
}

stage('Publish') {
  steps {
    script {
      docker.withRegistry('', 'dockerhub-id') {
        docker.image("${registry}:${env.BUILD_ID}").push('latest')
      }
    }

  }
}

stage('Deploy') {
  steps {
    sh 'docker stop flask-test-app || true; docker rm flask-test-app; true; docker run -d --name flask-test-app -p 9000:9000 maximmro/flask-test-app:latest'
  }
}

stage('Validate') {
  steps {
    sh 'curl localhost:9000/abc'
  }
}

}
environment {
registry = 'maximmro/flask-test-app'
}
}