# ASK LAB1
## 1. Przygotowanie platformy [1 pkt]
### Sprawdzenie poprawności działania docker'a:
``` 
docker ps
```
Wynik:
![alt text](https://github.com/JakubLatawiec/AdministracjaSieciamiKomputerowymi/blob/main/Lab1/Screenshots/gitclone.png?raw=true)

### Klonowanie repozytorium
```
git clone https://github.com/phajder/ask-node-react-docker
```
Wynik:
![alt text](https://github.com/JakubLatawiec/AdministracjaSieciamiKomputerowymi/blob/main/Lab1/Screenshots/dockerps.png?raw=true)

## 2. Baza danych [2 pkt]
### Pobranie obrazu z DockerHub:
```
docker pull mysql:latest
```
### Tworzenie własnego obrazu
Plik Dockerfile:
``` docker
#Użyj gotowego obrazu mysql w najnowszej wersji
FROM  mysql:latest

#Inicjalizuj bazę danymi z pliku db.sql
COPY  db.sql  /docker-entrypoint-initdb.d
```
Budowanie obrazu (bedąc w folderze db):
```
docker build -t mysql-db:0.1 .
```
### Tworzenie sieci
Stworzenie sieci typu bridge dla bazy danych i api
```
docker network create backend
```
Stworzenie sieci typu bridge dla api i strony React
```
docker network create frontend
```

### Tworzenie kontenera:
- ---name (nazwa kontenera)
- ---network (przypisanie do istniejącej sieci
- --d (uruchomienie kontenera w tle)
- --p (przekierowanie portu kontenera i działającej w nim aplikacji)
- --e (zmienne środowiskowe)
```
docker run --name mysql-db --network backend -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD="Zaq12wsx" mysql-db:0.1
```
### Weryfikacja bazy:
``` docker
docker exec -it mysql-db mysql -u"dockerdb" -p"Zaq12wsx"
```
Wynik:
![alt text](https://github.com/JakubLatawiec/AdministracjaSieciamiKomputerowymi/blob/main/Lab1/Screenshots/showdatabases.png?raw=true)

## 3. Aplikacja backendowa [2 pkt]
### Pobranie obrazu z DockerHub:
```
docker pull node:lts
```
### Tworzenie własnego obrazu
Plik Dockerfile:
``` docker
#Użyj gotowego obrazu node w wersji LTS
FROM  node:lts

#Ustaw domyślny folder aplikacji w kontenerze
WORKDIR  /app

#Skopiuj pliki aplikacji
COPY  .  .

#Zainstaluj odpowiednie biblioteki
RUN  npm  install

#Uruchom aplikacje
CMD  ["npm",  "run",  "dev"]
```
Plik .dockerignore:
``` docker
#Tutaj należy dodać pliki/katalogi pomijane przy wykonywaniu polecenia COPY
src
```
Budowanie obrazu (bedąc w folderze server):
```
docker build -t node-backend:0.1 .
```
### Tworzenie kontenera:
- ---name (nazwa kontenera)
- ---network (przypisanie do istniejącej sieci
- --d (uruchomienie kontenera w tle)
- --p (przekierowanie portu kontenera i działającej w nim aplikacji)
- --e (zmienne środowiskowe)
- --v (przypisanie voluminu)
```
docker run --name node-backend --network backend -d -p 8080:8080 -e .env -v $(pwd)/src:/app/src node-backend:0.1
```
Dodanie do sieci z frontendem:
```
docker network connect frontend node-frontend
```
### Weryfikacja API:
```
curl http://localhost:8080/api
```
Wynik przed zmianą:
![alt text](https://github.com/JakubLatawiec/AdministracjaSieciamiKomputerowymi/blob/main/Lab1/Screenshots/curlbefore.png?raw=true)

Wynik po zmianie:
![alt text](https://github.com/JakubLatawiec/AdministracjaSieciamiKomputerowymi/blob/main/Lab1/Screenshots/curlafter.png?raw=true)
## 4. Aplikacja frontendowa [2 pkt]
### Pobranie obrazu z DockerHub:
```
docker pull node:lts
```
### Tworzenie własnego obrazu
Plik Dockerfile:
``` docker
#Użyj gotowego obrazu node w wersji LTS
FROM  node:lts

#Ustaw domyślny folder aplikacji w kontenerze
WORKDIR  /app

#Skopiuj pliki aplikacji
COPY  .  .

#Zainstaluj odpowiednie biblioteki
RUN  npm  install

#Uruchom aplikacje
CMD  ["npm",  "start"]
```
Plik .dockerignore:
``` docker
#Tutaj należy dodać pliki/katalogi pomijane przy wykonywaniu polecenia COPY
src
public
```
Budowanie obrazu (bedąc w folderze client):
```
docker build -t node-frontend:0.1 .
```
### Tworzenie kontenera:
- ---name (nazwa kontenera)
- ---network (przypisanie do istniejącej sieci
- --d (uruchomienie kontenera w tle)
- --p (przekierowanie portu kontenera i działającej w nim aplikacji)
- --e (zmienne środowiskowe)
- --v (przypisanie voluminu)
```
docker run --name node-frontend --network frontend -d -p 3333:3000 -e CI=true -v $(pwd)/src:/app/src -v $(pwd)/public:/app/public node-frontend:0.1
```
### Weryfikacja aplikacji:
Przed zmianą:
![alt text](https://github.com/JakubLatawiec/AdministracjaSieciamiKomputerowymi/blob/main/Lab1/Screenshots/webpagebefore.png?raw=true)
Po zmianie:
![alt text](https://github.com/JakubLatawiec/AdministracjaSieciamiKomputerowymi/blob/main/Lab1/Screenshots/webpageafter.png?raw=true)

## 5. Część problemowa: połączenie wszystkich komponentów aplikacji [3 pkt]
### Ustawienie odpowieniego IP kontenera bazy danych
Sprawdzenie ip kontenerów w sieci backend:
```
docker network inspect backend
```
Wynik:
``` yaml
"Containers": {
            "0a72b24174ed96f5931da7c67abcff888139bcc6d20bd46e58c4f7566cdd2a12": {
                "Name": "node-backend",
                "EndpointID": "e20e183732a30f419d689283ddda1d65d178d4c365ead44add765ffbd9c453af",
                "MacAddress": "02:42:ac:13:00:03",
                "IPv4Address": "172.19.0.3/16",
                "IPv6Address": ""
            },
            "5e1141d1c16ce67ad1bd52d38e405cc86b0d5040a14bc92a0011670619a3b43b": {
                "Name": "db",
                "EndpointID": "c947496679ad35e57d1f22ad368741d569f73b3dc8aa12b2edd0b7f7a2a0cf90",
                "MacAddress": "02:42:ac:13:00:02",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            }
        },
```
Zmiana w pliku .env:
```
# Server
NODE_ENV=development
PORT=8080
IP=127.0.0.1

# Database
DB_HOST=172.19.0.2
DB_PORT=3306
DB_NAME=dockerdb
DB_USER=dockerdb
DB_PSWD=Zaq12wsx
```
Test:
```
curl http://localhost:8080/api/products
```
Wynik:
![alt text](https://github.com/JakubLatawiec/AdministracjaSieciamiKomputerowymi/blob/main/Lab1/Screenshots/curlproducts.png?raw=true)

### Ustawienie odpowieniego IP kontenera API
Sprawdzenie ip kontenerów w sieci frontend:
```
docker network inspect frontend
```
Wynik:
``` yaml
"Containers": {
            "0a72b24174ed96f5931da7c67abcff888139bcc6d20bd46e58c4f7566cdd2a12": {
                "Name": "node-backend",
                "EndpointID": "069bddb63b2816d4be7d355bfb3cb7d2f23be4eb05f399edbbb44d139db4fd38",
                "MacAddress": "02:42:ac:14:00:02",
                "IPv4Address": "172.20.0.2/16",
                "IPv6Address": ""
            },
            "f32d78473d5955603c85ed824f9f4cc6ae36b4147d214fb3c520ef069b4c8f39": {
                "Name": "node-frontend",
                "EndpointID": "fcd8be83c13ff2a081a88358783837e50097aae02081b04f13481125c8407657",
                "MacAddress": "02:42:ac:14:00:03",
                "IPv4Address": "172.20.0.3/16",
                "IPv6Address": ""
            }
        },
```
Zmiana w pliku package.json:
``` yaml
"proxy": "http://172.20.0.2:8080",
```
Test:
![alt text](https://github.com/JakubLatawiec/AdministracjaSieciamiKomputerowymi/blob/main/Lab1/Screenshots/webpagefetch.png?raw=true)

