# Use Node 18 for Angular application
FROM node:18

# Set the timezone
ENV TZ=Asia/Kolkata
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Set working directory
WORKDIR /app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application
COPY . .

# Expose port for ng serve
EXPOSE 4200

# Run ng serve
CMD ["npm", "run", "start"]


# Navigate to the directory containing the Dockerfile
cd /path/to/angular/project

# Build the Docker image
podman build -t angular-app:dev .

# Run the container
podman run -d --name angular-app -e TZ=Asia/Kolkata -p 4200:4200 angular-app:dev





# Use OpenJDK 17 for Spring Boot application
FROM openjdk:17-jdk

# Set the timezone
ENV TZ=America/New_York
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Copy the Spring Boot jar file
COPY target/your-app.jar /app.jar

# Expose port 8080
EXPOSE 8080

# Run the Spring Boot application
ENTRYPOINT ["java", "-jar", "/app.jar"]


# Navigate to the directory containing the Dockerfile
cd /path/to/springboot/project

# Build the Docker image
podman build -t spring-boot-app:latest .

# Run the container
podman run -d --name spring-boot-app -e TZ=America/New_York -p 8080:8080 spring-boot-app:latest


podman run -d --name mysql-db -e TZ=UTC -e MYSQL_ROOT_PASSWORD=rootpassword -e MYSQL_DATABASE=yourdb -p 3306:3306 mysql:8

