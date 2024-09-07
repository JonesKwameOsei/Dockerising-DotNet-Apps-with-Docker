# Dockerising-DotNet-Apps-with-Docker
Automating the containerising and running of a .Net application with CI/CD pipeline and Github Actions. 

## Initialize Docker assets
To create the necessary docker assets to containerise the application, I would employ `docker init`.<p>
![alt text](image.png) <p>
![alt text](image-2.png)<p>

The project directory now a dockerfile, docker compose file and a .dockerignore file as below:<p>
![alt text](image-1.png)

**Run The Application**:
To run the application, inside the project folder I will run:
`docker compose up --build`  

The build time is `238.6s`<p>
![alt text](image-3.png)

Application built and running:<p>
![alt text](image-4.png)

The application can be accessed on http://localhost:8080 <p>
![mywebapp](image-6.png)

**Run The Application in the Backgroud**
To run the application in a detach mode, I will use the `-d` flag.
`
docker compose --duild -d 
`

The second build time was significanlty faster. It only `2.7s` as compared to the first build. 
This is beacuse **Docker** employed *build cache* in the second build. It checked whether it can reuse the build intruction from a previous build which it did. This is efficient as it helps **Docker** to skip uunecessary work in the second build.<p>
The layers cached and reused: <p>
![cached layers](image-5.png)<p>
When constructing the second build with Docker, a layer is reused from the build cache in the previous build if the command and its dependent files remain unchanged from the last build. This reuse accelerates the build process as Docker avoids rebuilding the unchanged layer. Hence, the significant decrease of build time. 






