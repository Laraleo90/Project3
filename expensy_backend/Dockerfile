# 1. Start with a base image
FROM node:18

# 2. Set working directory
WORKDIR /app

# 3. Copy package.json and package-lock.json
COPY package*.json ./

# 4. Install dependencies
RUN npm install

# 5. Copy all application files
COPY . .

# 6. Expose the port your app uses
EXPOSE 8706

# 7. Run the app
CMD ["npm", "start"]

