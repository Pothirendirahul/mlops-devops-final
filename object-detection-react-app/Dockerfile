# Build stage
FROM node:18-alpine as builder

WORKDIR /app
COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html

# Remove the nginx.conf copy line or fix the path
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]