node {
    stage('Checkout') {
        checkout scm
    }
    
    stage('Build and Deploy') {
        // 1. Ruim oude containers op als die er nog zijn
        sh 'docker rm -f todoapp todoappdb || true'
        
        // 2. Maak een Docker-netwerk aan
        sh 'docker network create todo-net || true'
        
        // 3. Start de MariaDB database container
        sh 'docker run -d --name todoappdb --net todo-net -e MARIADB_ROOT_PASSWORD=sekrit -e MARIADB_DATABASE=todo_db -e MARIADB_USER=todo_usr -e MARIADB_PASSWORD=letmeinplz -p 3306:3306 mariadb:11'
        
        // 4. Bouw de Docker-image van de .NET-applicatie vanuit de TodoApp map
        sh 'docker build -t todoapp-image ./TodoApp'
        
        // 5. Start de .NET-applicatie container gekoppeld aan de database
        sh 'docker run -d --name todoapp --net todo-net -p 8080:8080 -e ConnectionStrings__TodoDb="Server=todoappdb;Port=3306;Database=todo_db;User=todo_usr;Password=letmeinplz;" -e ASPNETCORE_ENVIRONMENT=Development todoapp-image'
    }
}