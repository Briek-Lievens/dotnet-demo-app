node {
    stage('Checkout') {
        checkout scm
    }
    
    stage('Build and Deploy') {
        // 1. Ruim oude containers en volumes op
        sh 'docker rm -f todoapp todoappdb || true'
        sh 'docker volume rm mariadb-data || true'
        
        // 2. Maak een Docker-netwerk aan
        sh 'docker network create todo-net || true'
        
        // 3. Start de MariaDB database container (zonder problematische file mount)
        sh 'docker run -d --name todoappdb --net todo-net -e MARIADB_ROOT_PASSWORD=sekrit -e MARIADB_DATABASE=todo_db -e MARIADB_USER=todo_usr -e MARIADB_PASSWORD=letmeinplz -p 3306:3306 mariadb:11'
        
        // 4. Wacht even tot MariaDB volledig is opgestart
        sh 'sleep 10'
        
        // 5. Initialiseer de database met schema.sql vanuit de workspace via docker exec
        sh 'docker exec -i todoappdb mariadb -h localhost -utodo_usr -pletmeinplz todo_db < TodoApp/schema.sql'
        
        // 6. Bouw de Docker-image van de .NET-applicatie
        sh 'docker build -t todoapp-image ./TodoApp'
        
        // 7. Start de .NET-applicatie op poort 8081
        sh 'docker run -d --name todoapp --net todo-net -p 8081:8080 -e ConnectionStrings__TodoDb="Server=todoappdb;Port=3306;Database=todo_db;User=todo_usr;Password=letmeinplz;" -e ASPNETCORE_ENVIRONMENT=Development todoapp-image'
    }
}