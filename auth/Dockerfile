FROM node:18-alpine
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm ci --only=production || npm install --production
COPY . .
EXPOSE 3002
CMD ["node", "index.js"]
