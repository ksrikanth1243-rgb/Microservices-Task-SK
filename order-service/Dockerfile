# Use official Node.js LTS Alpine image for a lightweight container
FROM node:18-alpine

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json first to leverage Docker layer caching
COPY package*.json ./

# Install dependencies
RUN npm install --omit=dev

# Copy the rest of the application source code
COPY . .

# Expose the port the Order Service listens on
EXPOSE 3002

# Start the Order Service
CMD ["node", "app.js"]
