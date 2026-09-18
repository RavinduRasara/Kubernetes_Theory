
# Start with lightweight Alpine (Nginx is already installed inside this!)
FROM nginx:alpine

# Set the working directory to Nginx's default web folder
WORKDIR /usr/share/nginx/html

# Copy your HTML file into that folder
COPY index.html .

# Tell Docker that the container listens on port 80 at runtime
EXPOSE 80

# Tell Docker what to run when it starts
# "daemon off;" keeps Nginx running in the foreground so the container doesn't exit.bcz The container stays alive ONLY as long as the main command is running in the foreground. The container will die in less than a second. When you try to open http://localhost:8080 in your browser
CMD ["nginx", "-g", "daemon off;"]

