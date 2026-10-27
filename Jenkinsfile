node {
    stage('Checkout') {
        checkout scm
    }
    
    stage('Build') {
        // Bouw de Docker-image van de .NET-applicatie
        sh 'docker build -t todoapp-image ./TodoApp'
    }
    
    stage('Deploy Staging') {
        // 1. Ruim oude staging containers en volumes op
        sh 'docker rm -f todoapp todoappdb || true'
        
        // 2. Maak een Docker-netwerk aan
        sh 'docker network create todo-net || true'
        
        // 3. Start de MariaDB database container voor staging
        sh 'docker run -d --name todoappdb --net todo-net -e MARIADB_ROOT_PASSWORD=sekrit -e MARIADB_DATABASE=todo_db -e MARIADB_USER=todo_usr -e MARIADB_PASSWORD=letmeinplz -p 3306:3306 mariadb:11'
        
        // 4. Wacht tot MariaDB klaar is
        sh '''
            echo "Wachten tot MariaDB gereed is..."
            until docker exec todoappdb mariadb-admin ping -h 127.0.0.1 -utodo_usr -pletmeinplz --silent; do
                sleep 2
            done
        '''
        
        // 5. Initialiseer de database
        sh 'docker exec -i todoappdb mariadb --protocol=tcp -h 127.0.0.1 -utodo_usr -pletmeinplz todo_db < TodoApp/schema.sql'
        
        // 6. Start de .NET-applicatie op staging (poort 8081)
        sh 'docker run -d --name todoapp --net todo-net -p 8081:8080 -e ConnectionStrings__TodoDb="Server=todoappdb;Port=3306;Database=todo_db;User=todo_usr;Password=letmeinplz;" -e ASPNETCORE_ENVIRONMENT=Development todoapp-image'
    }

    stage('Deploy Production') {
        // Publish de .NET applicatie lokaal in de build container/server
        sh 'dotnet publish TodoApp/TodoApp.csproj -c Release -o ./publish'
        
        // Kopieer de bestanden naar de productie-webserver (dmz010)
        // (Zorg dat de SSH-sleutel van Jenkins bekend is op dmz010, of gebruik de juiste vagrant-gebruiker)
        sh 'scp -o StrictHostKeyChecking=no -r ./publish/* vagrant@192.0.2.10:/var/www/html/'
    }
}