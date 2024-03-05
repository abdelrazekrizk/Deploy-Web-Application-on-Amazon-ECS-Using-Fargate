# Use an official Node.js with version 14 as the base image
FROM node:14

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy the configuration file to the /usr/src/app directory inside the container.
COPY package*.json ./

# installing the Node.js dependencies specified in the package.json
RUN npm install

# Copy the rest of the application code to the working directory
COPY . .

# Expose a port for the application to listen on (if applicable)
EXPOSE 3000

# Define the command to run the application
CMD ["node", "server.js"]
