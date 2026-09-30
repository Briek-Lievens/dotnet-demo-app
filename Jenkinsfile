node {
    stage('Checkout') {
        checkout scm
    }
    stage('Build and Deploy') {
        sh 'docker compose down --remove-orphans || true'
        sh 'docker compose up -d --build'
    }
}