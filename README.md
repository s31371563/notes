#!/bin/bash

set -e

# Define project and repository information
PROJECT1_REPO="https://github.com/your_username/project1.git"
PROJECT2_REPO="https://github.com/your_username/project2.git"
BRANCH_NAME="main"

# Clone or update repositories
clone_or_update_repo() {
    REPO_URL=$1
    REPO_NAME=$(basename $REPO_URL .git)

    if [ -d $REPO_NAME ]; then
        echo "Updating $REPO_NAME repository..."
        cd $REPO_NAME
        git pull origin $BRANCH_NAME
        cd ..
    else
        echo "Cloning $REPO_NAME repository..."
        git clone -b $BRANCH_NAME $REPO_URL
    fi
}

# Install Maven, MySQL, DynamoDB, and Java 17 inside the container
install_dependencies() {
    CONTAINER_NAME=$1

    echo "Installing dependencies in $CONTAINER_NAME..."
    podman exec -it $CONTAINER_NAME /bin/bash -c "apt-get update && apt-get install -y maven mysql-client dynamodb-local openjdk-17-jdk"
}

# Build projects with Maven
build_projects() {
    PROJECT_DIR=$1

    echo "Building Spring Boot project in $PROJECT_DIR..."
    podman exec -it $PROJECT_DIR-container mvn -f /app/pom.xml clean install
}

# Remove and recreate the container
remove_and_recreate_container() {
    CONTAINER_NAME=$1
    IMAGE_NAME="multi-project-image"

    # Stop and remove the existing container
    podman stop $CONTAINER_NAME || true
    podman rm $CONTAINER_NAME || true

    # Run a new container
    echo "Running a new container..."
    podman run --name $CONTAINER_NAME --link mysql-container:mysql --link dynamodb-container:dynamodb -p 8081:8080 -p 8082:8081 -d $IMAGE_NAME
}

# Run Spring Boot projects with java -jar command
run_projects() {
    CONTAINER_NAME=$1

    echo "Running Spring Boot projects in $CONTAINER_NAME..."
    podman exec -it $CONTAINER_NAME /bin/bash -c "java -jar /app/project1/target/your_project1_jar_file.jar &"
    podman exec -it $CONTAINER_NAME /bin/bash -c "java -jar /app/project2/target/your_project2_jar_file.jar &"
}

# Clone or update both projects
clone_or_update_repo $PROJECT1_REPO
clone_or_update_repo $PROJECT2_REPO

# Create a common Dockerfile
cat <<EOL > Dockerfile
FROM adoptopenjdk:17-jdk-hotspot

WORKDIR /app

COPY . .

CMD ["bash"]
EOL

# Build Podman image
echo "Building Podman image..."
podman build -t multi-project-image .

# Run MySQL and DynamoDB containers
echo "Starting MySQL container..."
podman run --name mysql-container -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=projectdb -p 3306:3306 -d docker.io/library/mysql:latest

echo "Starting DynamoDB container..."
podman run --name dynamodb-container -p 8000:8000 -d amazon/dynamodb-local

# Install dependencies and copy repositories into the container
install_dependencies mysql-container
install_dependencies dynamodb-container
podman cp project1 mysql-container:/app
podman cp project2 dynamodb-container:/app

# Check if the container exists
CONTAINER_NAME="multi-project-container"
if podman inspect -t container $CONTAINER_NAME &> /dev/null; then
    echo "Updating existing container $CONTAINER_NAME"
    # Build and run projects in the existing container
    build_projects project1
    build_projects project2
    run_projects $CONTAINER_NAME
else
    # Run a new container
    remove_and_recreate_container $CONTAINER_NAME
    # Build and run projects in the new container
    build_projects project1
    build_projects project2
    run_projects $CONTAINER_NAME
    echo "Setup completed successfully!"
fi
