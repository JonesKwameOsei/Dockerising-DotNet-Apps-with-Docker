# Dockerising-DotNet-Apps-with-Docker
Automating the containerising and running of a .Net application with CI/CD pipeline and Github Actions. 

## Initialize Docker assets
To create the necessary docker assets to containerise the application, I would employ `docker init`.<p>
![alt text](image.png) <p>
![alt text](image-2.png)<p>

The project directory now has a dockerfile, docker compose file and a .dockerignore file as below:<p>
![alt text](image-1.png)

**Run The Application**:<p>
To run the application, inside the project folder I will run:

`
docker compose up --build
`  

The build time is `238.6s`<p>
![alt text](image-3.png)

Application built and running:<p>
![alt text](image-4.png)

The application can be accessed on http://localhost:8080 <p>
![mywebapp](image-6.png)

**Run The Application in the Background**<p>
To run the application in a detach mode, I will use the `-d` flag.

`
docker compose --duild -d 
`

The second build time was significanlty faster. It only `2.7s` as compared to the first build. 
This is beacuse **Docker** employed *build cache* in the second build. It checked whether it can reuse the build intruction from a previous build which it did. This is efficient as it helps **Docker** to skip unnecessary work in the second build.<p>
The layers cached and reused: <p>
![cached layers](image-5.png)<p>
When constructing the second build with Docker, a layer is reused from the build cache in the previous build if the command and its dependent files remain unchanged from the last build. This reuse accelerates the build process as Docker avoids rebuilding the unchanged layer. Hence, the significant decrease of build time. 

**Confirm build on the cli**:
To comfirm if our service, web server is built, I will run:

`
docker compose ls
`

![image](https://github.com/user-attachments/assets/1f4e3262-4352-4410-a956-06f5e1f6f8c2)

Checking containers running: 

`
docker container ls
`
![image](https://github.com/user-attachments/assets/4c94a4fd-ecb3-4257-9625-b1032e0a81fe)

## Update The Application 

To update th .NET application:
1. Stash any previous changes. 

`
git stash -u
`

2. Checkout the new branch with to update the application. 

```
git checkout -b add-db
```

Outcome:<p>
![image](https://github.com/user-attachments/assets/01d356ab-c14f-4627-8cc2-438b1ba656b3)

### Add a local database and persist data
I will use a container to set up local database. This would be done by updating the compose.yaml file to define a database service and a persistent volume. This db service utilise a postgresSQL database container. 

in the main branch, the compose.yaml file had only one service, the server, configured as below:

```
services:
  server:
    build:
      context: .
      target: final
    ports:
      - 8080:80
```

Within the `Add-db` branch, the service has been updated with an addition of a db service as should below: 
```
 depends_on:
      db:
        condition: service_healthy
  db:
    image: postgres
    restart: always
    user: postgres
    secrets:
      - db-password
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=example
      - POSTGRES_PASSWORD_FILE=/run/secrets/db-password
    expose:
      - 5432
    healthcheck:
      test: [ "CMD", "pg_isready" ]
      interval: 10s
      timeout: 5s
      retries: 5
volumes:
  db-data:
secrets:
  db-password:
    file: db/password.txt
```

The full configuration would now look like this:

```
services:
  # Web Server or frontend service
  server:
    build:
      context: .
      target: final
    ports:
      - 8080:80
  # db service added
    depends_on:
      db:
        condition: service_healthy
  db:
    image: postgres
    restart: always
    user: postgres
    secrets:
      - db-password
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=example
      - POSTGRES_PASSWORD_FILE=/run/secrets/db-password
    expose:
      - 5432
    healthcheck:
      test: [ "CMD", "pg_isready" ]
      interval: 10s
      timeout: 5s
      retries: 5
volumes:
  db-data:
secrets:
  db-password:
    file: db/password.txt
```

> **Note**: The Postgres database has its password stored in a file called password.txt within db directory.  
**Build the Updated Applicatiocation**:

Having configured the .NET app with a local db, the new app has a db service attached to it. I will opnce again apply the new changed by running:

```
docker compose up --build
```

![image](https://github.com/user-attachments/assets/6074a580-d336-40d4-8f83-2dfdfb4936e5)<p>


The new build took `13.1s`. The build time increased as compared to the last build becuase the build instruction had changed. **Docker** could not find a build history for the updated app, which is the database added. 

**Confirmed build**:<p>
Accessing web app from the internet.<p>
![image](https://github.com/user-attachments/assets/91c75aa0-18b4-488e-8b0e-51d8d527d43f)<p>

**Run Updated App in Detach Mode**:

Let's build the app to run in the background.
`
docker compose up --build -d 
`

The second build took only 2.6 seconds to build. Again, docker has cached the previous build and used same in the new build since configurations have not changed. <p>
![image](https://github.com/user-attachments/assets/88770be3-2f93-447d-b63c-bf88b76790ff)<p>

Let's confirm from the command line to see the services and containers running. <p>
```
docker compose ls           # Outputs number services created

docker container ls         # Outputs number of containers running 
```
![image](https://github.com/user-attachments/assets/c8214aa1-a16c-4bda-81d4-37b44e452b1a)<p>
![image](https://github.com/user-attachments/assets/2e85aeca-e91f-4916-99f0-31e6eb638001)<p>

Let check the logs of the Postgres DB. This, i will run `docker compose logs -f db`.<p>

![image](https://github.com/user-attachments/assets/674e50c4-da06-409b-a7c4-af8cff7d0c0d)<p>

## Adding Records to the Database
From the updated web server accessed above, we could see simple wep application with the line of text, `Student name is`.  The application does not show a name because the database is empty. To resolve this, I will access the database and add records. To get to the database and run commands inside it, I will need to run `docker exec` command. The `id` of the postgres database is needed into get to the container. 

```
docker container ls    # to get the id
```
![image](https://github.com/user-attachments/assets/2b504c8f-ba5b-44ea-a80f-3222d471a1b7)<p>

Having obtained the id of the postgres container, I can now `exec` into the container. 
```
docker exec -it a3f985971878 psql -d example -U postgres
```
![image](https://github.com/user-attachments/assets/4a05679d-c3b5-4e0e-a457-15675629959e)<p>

Insert some records into the database:<p>

```
example=# INSERT INTO "Students" ("ID", "LastName", "FirstMidName", "EnrollmentDate") VALUES (DEFAULT, 'Jones', 'Osei', '2015-11-25');
```
![image](https://github.com/user-attachments/assets/ebf3a4ab-bdf6-419c-a799-9e4a2ba92f7a)<p>

## Verifying That The Data Persists in the Database
Now I will open the web app in a browser to see if the student name has now been populated. <p>
![image](https://github.com/user-attachments/assets/294e48e3-08f7-422f-b430-76a5d889eaed)<p>

Indeed, the data has been persisted. Will the data persist if the container is accidentally removed or crashed? 
```
docker compose rm        # remove all the containers

docker compose up --build    # to run the app again
```
![image](https://github.com/user-attachments/assets/2c3e190b-f9f1-42ee-a968-34a66d1f4374)<p>

![image](https://github.com/user-attachments/assets/785822b8-a871-4e29-9e7b-ea67bc18c774)<p>

The data has been persisted even after removing the containers. <p>
![image](https://github.com/user-attachments/assets/978bffbf-b669-49d2-b957-76c64428e582)<p>











