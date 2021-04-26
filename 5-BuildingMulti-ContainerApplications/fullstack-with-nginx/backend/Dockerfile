FROM node

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 80

ENV MONGODB_USERNAME=default-username
ENV MONGODB_PASSWORD=default-passoword

# CMD [ "node", "app.js" ]
CMD ["npm", "start"]