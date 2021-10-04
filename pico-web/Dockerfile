FROM nginx:stable-alpine
COPY dist/ /usr/share/nginx/html
RUN set -ex \
 && sed -r -i '/location\s+\/\s+\{/a try_files $uri $uri/ /index.html;' /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
