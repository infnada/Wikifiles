---
title: Dockerfile
description: 
published: true
date: 2019-03-09T16:40:02.729Z
tags: 
---

> File validator https://www.fromlatest.io/

- Add vs Copy => Add para tar.gz

# Buenas prácticas al escojer una imagen de Docker
- Oficial preferiblemente.
- Que no este basado en binarios producidos por terceros no conocidos.
- Ha de tener DockerFile!.
- Deberiamos tener un registro propio curado de contenedores.
- La version utilizada en el FROM debe ser fijada (Almenos hasta la Major), evitará que tu contenedor deje de funcionar si en una version superior hay incompatibilidades.
- Cada vez que se usa RUN, se crea una capa.
- Cuando usas un segundo FROM con referencia al primero, el producto del primer FROM es temporal.

# Ejemplos simples

```
FROM library/debian:latest
#        Default ENTRYPOINT -> /bin/sh -c
ENTRYPOINT [ "/bin/ping", "-c", "5" ]
CMD [ "8.8.8.8" ]
```

```
FROM debian:9
RUN apt-get update && \
    apt-get -y upgrade &&\
    apt-get -y install libapache2-mod-php7.0 php-mysql php-gd php-redis wget unzip && \
    apt-get clean && \
    rm -fr /var/lib/apt/lists/* && \
    wget -P /var/www/html/ https://wordpress.org/latest.zip && \
    unzip /var/www/html/latest.zip -d /var/www/html/ && \
    rm -fr /var/www/html/latest.zip && \
    cp /var/www/html/wordpress/wp-config-sample.php /var/www/html/wordpress/wp-config.php && \
    cp -r /var/www/html/wordpress/* /var/www/html && \
    ln -sf /dev/stdout /var/log/apache2/access.log && \
    ln -sf /dev/stderr /var/log/apache2/error.log && \
    sed -i "s/^session.save_handler.*/session.save_handler = redis/" /etc/php/7.0/apache2/php.ini && \
		sed -i "/^session.save_handler/i session.save_path = \"tcp://redis\"" /etc/php/7.0/apache2/php.ini

EXPOSE 80

ENTRYPOINT ["/usr/sbin/apachectl"]
CMD ["-DFOREGROUND"]
```