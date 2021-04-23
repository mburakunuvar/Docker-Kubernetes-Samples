FROM node:14

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

# --build-arg
ARG DEFAULT_PORT=80
# TO MAKE FIXED ENV VARIABLES FLEXIBLE AT BUILD TIME
# to be able to build different images in a flexible way without having 
# to change the Dockerfile all the time.
# $ docker build -t feedback-node:dev --build-arg DEFAULT_PORT=8000

# WITHOUT ARG 
# ENV PORT  80 

# WITH ARG
ENV PORT $DEFAULT_PORT
# CAN BE CONFIGURED AT RUN TIME
# $ docker run... --env PORT=8000...
# $ docker run... -e var1 -e var2... 
# $ docker run ... --env-file ./.env ... 


EXPOSE $PORT

# VOLUME [ "/app/node_modules" ]
# will be overwritten by CLI
# And this can also help with performance and efficiency.
# VOLUME [ "/temp" ]

# named volumes will be necessary for permanent data 
# anonymous volume exists as long as container exists
# VOLUME ["app/feedback"]

# CMD [ "node", "server.js" ]
CMD [ "npm", "start" ]