node {
    stage('Checkout') {
        checkout scm
    }
    stage('Build and Deploy') {
        sh 'docker compose down || true'
        sh 'docker compose up -d --build'
    }
}