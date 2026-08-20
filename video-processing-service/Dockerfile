# #Start with a base image...we are starting with the Node18 run time and its misleading 
# #because this is not just including node js but it is built on top of a linux docker image
# #docker is essentially running an operating system
# #docker images can be built on top of each other

# #FUN INCEPTION POINTER
# #So we are starting with this node:18 base image which is itself built on top of a linux image
# #and now we are building our own image on top of this
# FROM node:18

# #Docker image can be used to instantiate containers
# #Docker file is a blue print for a docker image
# #Docker image is a blue print for a container
# #container is going to run our code within its own isolated environment with its own file system
# #
# #Set the working directory in the container to /app
# WORKDIR /app

# #Copy package.json and package-lock.json into the working directory
# COPY package*.json ./

# #INSTALL ffmpeg in the container
# RUN apt-get update && apt-get install -y ffmpeg

# #Install any needed packages needed inside package.json
# #This must come after package.json so that it knows which files are supposed to be installed.
# RUN npm install

# #Copy all of the source code inside the docker image
# #we don't need to copy over node modules and they should be added to a docker ignore file since
# #we are already running npm install
# COPY . .

# #Make the port 3000 available outside this container
# EXPOSE 3000

# #Define the command to run your app using CMD (only one CMD allowed)
# #Since docker is a production env deployment and we have not 
# CMD ["npm", "start"]


####################################################################################
# Stage 1: Build stage
FROM node:18 AS builder

# Set the working directory in the container to /app
WORKDIR /app

# Copy package.json and package-lock.json into the working directory
COPY package*.json ./

# Install any needed packages specified in package.json
RUN npm install

# Bundle app source inside the docker image
COPY . .

# Build the app
RUN npm run build

# Stage 2: Production stage
FROM node:18

# Install ffmpeg in the container
RUN apt-get update && apt-get install -y ffmpeg

# Set the working directory
WORKDIR /app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install only production dependencies
RUN npm install --only=production

# Copy built app from the builder stage
COPY --from=builder /app/dist ./dist

# Make port 3000 available to the world outside this container
EXPOSE 3000

# Define the command to run your app using CMD which defines your runtime
CMD [ "npm", "run", "serve" ]
