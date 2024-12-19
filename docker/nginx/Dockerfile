FROM nginx:1.26.0-alpine AS webserver


# dd your custom Nginx configuration
COPY docker/nginx/nginx.conf /etc/nginx/nginx.conf

# Expose the default port
EXPOSE 80 443

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
