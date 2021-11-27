# devops_lesson_trial
sudo mkdir -p ~/build/site  # Создаем папку для сборки
sudo chown root:jenkins -R ~/build/ # Даем Jenkins права на запись в этой папке 
sudo chmod 775 -R ~/build/ 
 
sudo rsync -av $WORKSPACE/landing/ ~/build/site/ # Переносим сайт из WORKSPACE Jenkins в сборочную директорию 
 
sudo echo " 
server { 
 
 server_name devops.local; 
 
 access_log /var/log/nginx_access.log; 
 
 error_log /var/log/nginx_error.log; 
 
 root /var/www; 
 
 location / { 
 
 index index.html index.htm index.php; 
 
    }
 
} " > ~/build/site.conf 
 
 
sudo echo "FROM nginx 
RUN apt update && apt-get install -y locales 
 
# Locale 
RUN sed -i -e \ 
 's/# ru_RU.UTF-8 UTF-8/ru_RU.UTF-8 UTF-8/' /etc/locale.gen \ 
 && locale-gen 
 
ENV LANG ru_RU.UTF-8 
ENV LANGUAGE ru_RU:ru 
ENV LC_LANG ru_RU.UTF-8 
ENV LC_ALL ru_RU.UTF-8 
 
RUN mkdir /var/www 
RUN rm -f /etc/nginx/conf.d/default.conf 
COPY site.conf /etc/nginx/conf.d/ 
COPY site/. /var/www/
RUN  chown nginx:nginx -R /var/www/" > ~/build/Dockerfile
 
 
sudo docker rm -f web # Удаляем старый контейнер
 
cd ~/build/ # Помним что сборку нужно запускать из папки в которой находится Dockerfile
 
 
sudo docker build -f ~/build/Dockerfile . -t web/dev # Запускаем сборку образа
 
 
sudo docker run --name web -p 80:80 -d web/dev #Запускаем новый контейнер из собранного образа
 
sudo rm -rf ~/build/ # Убираем за собой
