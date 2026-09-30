node {
    stage('Checkout') {
        checkout scm
    }
    
    stage('Build and Deploy') {
        // 1. Ruim oude containers én het oude datavolume op
        sh 'docker rm -f todoapp todoappdb || true'
        sh 'docker volume rm mariadb-data || true'
        
        // 2. Maak een Docker-netwerk aan
        sh 'docker network create todo-net || true'
        
        // 3. Start MariaDB mét automatische schema-initialisatie vanuit schema.sql
        sh 'docker run -d --name todoappdb --net todo-net -e MARIADB_ROOT_PASSWORD=sekrit -e MARIADB_DATABASE=todo_db -e MARIADB_USER=todo_usr -e MARIADB_PASSWORD=letmeinplz -v mariadb-data:/var/lib/mysql:Z -v $(pwd)/TodoApp/schema.sql:/docker-entrypoint-initdb.d/schema.sql:ro -p 3306:3306 mariadb:11'
        
        // 4. Bouw de Docker-image van de .NET-applicatie
        sh 'docker build -t todoapp-image ./TodoApp'
        
        // 5. Start de .NET-applicatie op poort 8081
        sh 'docker run -d --name todoapp --net todo-net -p 8081:8080 -e ConnectionStrings__TodoDb="Server=todoappdb;Port=3306;Database=todo_db;User=todo_usr;Password=letmeinplz;" -e ASPNETCORE_ENVIRONMENT=Development todoapp-image'
    }
}