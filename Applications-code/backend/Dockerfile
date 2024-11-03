# Base image
FROM node:14-alpine

# Create app directory
WORKDIR /app

# Copy package.json and install dependencies
COPY package.json ./
RUN npm install

# Copy the backend code
COPY . .

# Expose the backend port
EXPOSE 3000

# Run the app
CMD ["npm", "start"]
