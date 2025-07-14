# Dockerfile for Node.js API application
# This Dockerfile sets up a Node.js environment for the API application

# Use the official Node.js image from Docker Hub rather than getting ubuntu and then installing node and npm
# Using a specific version to ensure consistency
FROM node:19.6-alpine

# Set the working directory inside the container
# This is where the application code will be copied to and run from
WORKDIR /usr/src/app

# Set environment variables
# NODE_ENV is set to 'production' to optimize the application for production use
ENV NODE_ENV=production

# Copy package.json and package-lock.json (if available) to the working directory
# This allows Docker to cache the npm install step, so it doesn't run every time the code changes
# This is important for performance, as it avoids reinstalling dependencies unnecessarily
COPY package*.json ./

# Install the application dependencies
# This will install all the dependencies listed in package.json
# Install using npm ci to ensure a clean and reproducible installation
# The --only=production flag ensures that only production dependencies are installed, ignoring devDependencies
# The --mount=type=cache option is used to cache the npm modules, improving build performance
# This cache will be stored in the specified target directory, which is /usr/src/app/.
RUN --mount=type=cache,target=/usr/src/app/.npm \
    npm set cache /usr/src/app/.npm && \
    npm ci --only=production

# Setting the user to 'node' for security reasons
# This ensures that the application runs with a non-root user, which is a best practice for security
# The 'node' user is created by the official Node.js image
USER node

# Copy the application source code into the container
# This copies all files from the current directory (where the Dockerfile is located) to the working directory in the container
# The --chown=node:node option sets the ownership of the files to the 'node' user and group
COPY --chown=node:node ./src .

# Expose the port that the application will run on
# This is the port that the Node.js application will listen on
EXPOSE 3000

# Setting the commands to run the application
# This command will start the Node.js application when the container is run
CMD [ "node", "index.js" ]
