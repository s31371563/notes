# Use a base image with Podman and Java 17 Corretto installed
FROM amazoncorretto:17

# Install necessary dependencies
RUN dnf install -y git maven

# Set the working directory
WORKDIR /app

# Copy the script into the container
COPY update_and_run.sh /app/

# Make the script executable
RUN chmod +x /app/update_and_run.sh

# Expose all ports
EXPOSE 0-65535

# Command to run the script
CMD ["/app/update_and_run.sh"]



#!/bin/bash

# Function to clone or update a Git repository
clone_or_update() {
    local repo_url=$1
    local branch=$2
    local repo_name=$(basename "$repo_url" .git)
    
    if [ -d "$repo_name" ]; then
        # If the repository exists, pull the latest changes
        cd "$repo_name" || exit
        git pull origin "$branch"
    else
        # If the repository does not exist, clone it
        git clone --branch "$branch" "$repo_url"
        cd "$repo_name" || exit
    fi
}

# Clone or update your Spring Boot projects
clone_or_update "https://github.com/yourusername/project1.git" "branch1"
clone_or_update "https://github.com/yourusername/project2.git" "branch2"

# Build the projects with Maven
mvn clean install

# Run the Spring Boot applications
java -jar project1/target/project1.jar &
java -jar project2/target/project2.jar &

# Simple local server to keep the container running
while true; do
    sleep 60
done
