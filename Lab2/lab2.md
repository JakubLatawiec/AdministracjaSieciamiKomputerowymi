# ASK LAB 2
## Automatyzacja przy pomocy docker compose [4 pkt]
### Przygotowany plik compose.yaml:
``` yaml
services:
	#Kontener bazy danych:
	db:
		build:
			context: ./db  #Folder z plikami
			dockerfile: Dockerfile  #Nazwa pliku budującego obraz
		networks:
			backend: #Przypisanie do sieci "backend"
				ipv4_address: 172.38.117.10  #Przypisanie statycznego adresu ip
		environment:
			- MYSQL_ROOT_PASSWORD="Zaq12wsx"  #Przesłanie zmiennej środowiskowej z hasłem dla root

	#Kontener API:
	backend:
		build:
			context: ./server  #Folder z plikami
			dockerfile: Dockerfile  #Nazwa pliku budującego obraz
		networks:
			backend: #Przypisanie do sieci "backend"
				ipv4_address: 172.38.117.5  #Przypisanie statycznego adresu ip
			frontend: #Przypisanie do sieci "frontend"
				ipv4_address: 172.58.117.5  #Przypisanie statycznego adresu ip
		env_file:
			- ./server/.env  #Przesłanie pliku ze zmiennymi środowkowymi
		volumes:
			- ./server/src:/app/src  #Przypisanie wolumena
		depends_on:
			- db  #Poczekanie na załadowanie kontenera bazy danych

	#Kontener aplikacji React:
	frontend:
		build:
			context: ./client  #Folder z plikami
			dockerfile: Dockerfile  #Nazwa pliku budującego obraz
		ports:
			- 3333:3000  #Upublicznienie portu 3000 i przekierowanie go na port 3333
		networks:
			frontend: #Przypisanie do sieci "frontend"
				ipv4_address: 172.58.117.10  #Przypisanie statycznego adresu ip
		environment: #Ustawnienie zmiennych środowiskowych aplikacji
			- CI=true
			- DANGEROUSLY_DISABLE_HOST_CHECK=true
		volumes: #Podpięcie wolumenów
			- ./client/src:/app/src
			- ./client/public:/app/public 
```
Do poprawnego działania pliku compose.yaml, należy wykonać następny krok.

## Custom networking w dockerze [2 pkt]
### Uzupełnienie pliku compose.yaml:
``` yaml
...
# Wcześniej uzupełniona sekcja "services"
...

#Tworzenie customowych sieci
networks:
	frontend: #Sieć o nazwie frontend
		ipam:
			driver: default
			config:
				- subnet: 172.58.117.0/24  #Adresacja sieci
	backend: #Sieć o nazwie backend
		ipam:
			driver: default
			config:
				- subnet: 172.38.117.0/24  #Adresacja sieci
```

### Zbudowanie obrazów i kontenerów przy użyciu docker compose:
```
docker compose -f compose.yaml up -d
```
### Wynik:

## Produkcyjny Dockerfile apki klienckiej [2 pkt]
### Plik Dockerfile.Production tworzący obraz aplikacji w wersji produkcyjnej:
``` dockerfile
#Użyj gotowego obrazu node w wersji LTS
FROM  node:lts  as  build

#Ustaw domyślny folder aplikacji w kontenerze
WORKDIR  /app

#Skopiuj pliki zależności aplikacji
COPY  package*.json  ./
COPY  *.config.js  ./
COPY  *.json  ./

#Zainstaluj zależności
RUN  npm  install

#Skopiuj pliki źródłowe aplikacji
COPY  .  /app

#Zbuduj aplikację w wersji produkcyjnej
RUN  npm  run  build

#Użyj gotowego obrazu nginx
FROM  nginx

#Skopiuj plik konfiguracyjny nginx
COPY  ./nginx.conf.template  /etc/nginx/nginx.conf

#Skopiuj zbudowane pliki aplikacji
COPY  --from=build  /app/build  /usr/share/nginx/html

#Uruchom serwer
CMD  ["nginx",  "-g",  "daemon  off;"]
```
### Uzupełniony plik nginx.conf.template:
```
worker_processes auto;

events {
    worker_connections 8000;
    multi_accept on;
}

http {
    # What types to include
    include /etc/nginx/mime.types;
    # Which is the default
    default_type application-octet-stream;

    upstream backend {
        server 172.58.117.5:8080;
    }

    server {
        # Listen on port 80
        listen 80;
        # Logs dir
        access_log /var/log/nginx/access.log;

        recursive_error_pages   on;

        # all of the status defined at http.cat AND allowed by nginx to be error_page'd

        error_page 404 /status-error.html;

        location /status-error.html {
            resolver 1.1.1.1 ipv6=off;
            proxy_ssl_server_name on;
            proxy_pass https://http.cat/$status;
            internal;
        }

        # Root directory
        root /usr/share/nginx/html;
        # Index files
        index index.html;

        # React content
        location / {
            # First attempt to serve request as file,
            # then as directory,
            # then fall back to redirecting to index.html
            try_files $uri $uri/ =404; #/index.html;
        }

        # Proxy to API server
        location /api {
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-NginX-Proxy true;
            proxy_pass http://backend;
            proxy_ssl_session_reuse off;
            proxy_set_header Host $http_host;
            proxy_cache_bypass $http_upgrade;
            proxy_redirect off;
        }
    }
}

```
### Budowanie obrazu (w folderze client):
```
docker build -t nginx-app:0.1 -f Dockerfile.Production .
```
Zbudowany obraz zostanie wykorzystany w pliku compose-production.yaml w następnym punkcie.

## Prodykcyjna wersja compose.yaml [2 pkt]
### Stworzony plik compose.production.yaml:
``` yaml
services:
	frontend:
		image: nginx-app:0.1  #Wykorzystanie zbudowanego wcześniej obrazu
	ports:
		- "80:80"  #Udostępnienie i przekierowanie portu 80
	environment:
		- NGINX_ENVSUBST_OUTPUT_DIR="/etc/nginx/nginx.conf"  #Zmienna środowiskowa ze ścieżką konfiguracji
```
### Uruchomienie aplikacji w wersji produkcyjnej:
```
docker compose -f compose.yaml -f compose.production.yaml up -d
```
Kontener "frontend" z pliku compose.yaml zostanie nadpisany kontenerem o tej samej nazwie z pliku compose-production.yaml.

### Wynik:
