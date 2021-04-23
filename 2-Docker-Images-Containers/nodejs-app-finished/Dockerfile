FROM node:12

WORKDIR /app

# relative path is WORKDIR
# COPY . ./ 
COPY package.json /app

RUN npm install

COPY . /app

#only for documentation purpose
EXPOSE 80

CMD ["node", "server.js"]