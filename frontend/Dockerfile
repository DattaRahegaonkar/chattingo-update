# ------------------------ Stage 1 -----------------------------

# base image for building the React app
FROM node:18-slim AS builder

# set working directory for the app
WORKDIR /app

# copy packages files into the  container working directory
COPY package.json package-lock.json ./

# install app dependencies
RUN npm install

# copy the rest of the app source code into the container working directory
COPY . .

# build the React app for production
RUN npm run build

# ------------------------ Stage 2 -----------------------------

# base image for serving the React app
FROM nginx:alpine

# copy the built app from the builder stage to the nginx html directory
COPY --from=builder /app/build /usr/share/nginx/html

# Copy custom nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

# expose port 80 to the outside world
EXPOSE 80

# start nginx server
CMD ["nginx", "-g", "daemon off;"]