

## PROXY

### Docker-compose

Creamos el `docker-compose.ylm` con la siguiente estructura: 

```
services:

  nginx:
    image: ubuntu/nginx # imagen de Nginx
    container_name: nginx_server # nombre del contenedor
    # ports:
    #  - "8081:80" # mapeo de puertos  HTTP
    volumes:
      - ./nginx/sites-available:/etc/nginx/sites-available # archivos de configuración de hosts virtuales
      - ./nginx/website:/var/www/html/ # directorio de los sitios web
    restart: always # reinicio automático
    networks:
      - webnet # red de contenedores

  apache:
    image: ubuntu/apache2 # imagen de Apache
    container_name: apache_server # nombre del contenedor
    # ports:
    #  - "8082:80" # mapeo de puertos  HTTP
    volumes:
      - ./apache/sites-available:/etc/apache2/sites-available # archivos de configuración de hosts virtuales
      - ./apache/website:/var/www/html/ # directorio de los sitios web
      - ./apache/htpasswd/.htpasswd:/etc/apache2/.htpasswd # archivo de contraseñas (en este caso lo uso)
    restart: always # reinicio automático
    networks:
      - webnet # red de contenedores

  proxy:
    image: ubuntu/nginx # imagen de Nginx
    container_name: proxy_server # nombre del contenedor
    ports:
      - "80:80" # mapeo de puertos  HTTP
      - "443:443" # mapeo de puertos  HTTPS
    volumes:
      - ./proxy/conf/nginx.conf:/etc/nginx/nginx.conf # archivo de configuración principal
      - ./proxy/certs:/etc/nginx/certs # directorio de certificados (hechos con openssl)
    restart: always # reinicio automático
    depends_on:
      - apache # dependencia de Apache
      - nginx # dependencia de Nginx
    networks:
      - webnet # red de contenedores

networks:
  webnet:
  
```


## NGINX
Para Nginx hacemos lo siguiente: 

### Websites

Creamos esta estructura con sus paginas por defecto:

```
websites/
├── index.html
└── mario.com/
    ├── index.html
    └── error/
        ├── 404.html
        └── 500.html
```

### Sites available
En la carpeta `sites-available` agregamos el `default`:
```
server {
    listen 80;
    listen [::]:80;

    root /var/www/html/; # Ruta de la carpeta raíz del dominio
    index index.html; # Archivo por defecto

    # Configuración para la raíz
    location / {
        try_files $uri $uri/ /index.html; # Intenta servir el archivo solicitado, si no existe, sirve
    }

    # Configuración para /mario
    location /mario {
        alias /var/www/html/mario.com;
    }

    # Personalizar la página de error 404
    error_page 404 /404.html;
    location = /404.html {
        root /var/www/html/mario.com/errors; # Ruta donde se encuentra el archivo de error
        internal; # Asegura que la página de error no sea accesible directamente
    }

    # la página de error 500
    error_page 500 502 503 504 /500.html;
    location = /500.html {
        root /var/www/html/mario.com/errors; # Ruta donde se encuentra el archivo de error
        internal; # Asegura que la página de error no sea accesible directamente
    }
}
```

## APACHE
En primer lugar creamos esta estructura:

```
apache/
├── htpasswd/
│   └── .htpasswd           # Archivo para almacenar usuarios y contraseñas para autenticación HTTP
├── sites-available/
│   └── 000-default.conf    # Archivo de configuración para el sitio web por defecto
└── website/
    └── dedom/
        ├── errors/
        │   ├── 401.html    # Página de error "No autorizado"
        │   ├── 403.html    # Página de error "Prohibido"
        │   ├── 404.html    # Página de error "No encontrado"
        │   └── 500.html    # Página de error "Error interno del servidor"
        ├── privado/
        │   ├── index.html  # Página principal dentro de la carpeta privada
        │   └── .htaccess   # Archivo de configuración para proteger la carpeta privada
        └── index.html      # Página principal del sitio web
```

### .htpasswd
### sites-available
### websites
#### .htacces