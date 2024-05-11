# ASK LAB 3
## Traefik Proxy [2 pkt]
### Tworzenie sieci w pliku compose.yaml:
``` yaml
services:
  ...
  # Utworzone poprzednio serwisy
  ...
  
networks:
  ...
  # Utworzone poprzednio sieci
  ...
  proxy: #Sieć o nazwie proxy
    name: proxy
    ipam:
      driver: default
      config:
        - subnet: 172.28.117.0/24  #Adresacja sieci
```

### Konfiguracja Traefik'a w pliku proxy/compose.yaml:
``` yaml
services:
  traefik:
    image: "traefik:v3.0"
    container_name: "traefik"
  ports:
    - "80:80"  # Mapowanie portu 80 hosta na port 80 kontenera
    - "443:443"  # Mapowanie portu 443 hosta na port 443 kontenera
  command:
    - "--api.insecure=true"  # Udostępnianie dashboardu bez uwierzytelniania
    - "--providers.docker=true"
    - "--providers.docker.exposedbydefault=false"
    - "--entrypoints.web.address=:80"  # Nasłuchiwanie na porcie 80 hosta
    - "--entrypoints.websecure.address=:443"  # Nasłuchiwanie na porcie 443 hosta

  volumes:
    - "/var/run/docker.sock:/var/run/docker.sock:ro"
  networks:
    - proxy
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.api.rule=Host(`traefik.localhost`)"
    - "traefik.http.routers.api.service=api@internal"


networks:
  proxy:
    name: proxy
    external: true
```
### Wynik:
VSCode przekirował port 80 na 44979:
![Alt text](https://github.com/JakubLatawiec/AdministracjaSieciamiKomputerowymi/blob/main/Lab3/Screenshots/TreafikDashboard.png?raw=true)

## DuckDNS domains [1 pkt]
### Utworzone domeny:
![Alt text](https://github.com/JakubLatawiec/AdministracjaSieciamiKomputerowymi/blob/main/Lab3/Screenshots/DuckDNS.png?raw=true)

## Serwis dostępny publicznie - React client [2 pkt]
### Zmodyfikowany plik compose.production.yaml:
``` yaml
services:
  frontend:
    ...
    # Poprzednie ustawienia
    ...
    networks:
      frontend: #Przypisanie do sieci "frontend"
        ipv4_address: 172.58.117.10  #Przypisanie statycznego adresu ip
      proxy: #Przypisanie do sieci "frontend"
        ipv4_address: 172.28.117.10  #Przypisanie statycznego adresu ip
    labels:
      traefik.enable: true #Przekierowywanie przez Traefik
      traefik.docker.network: proxy #Domyślna sieć przekierowywania

      #HTTP
      traefik.http.routers.frontend.entrypoints: web #Entrypoint nasłuchujący na porcie 80
      traefik.http.routers.frontend.rule: Host(`frontend.jlatawiec.duckdns.org`) #Adres usługi
```
### Wynik:
![Alt text](https://github.com/JakubLatawiec/AdministracjaSieciamiKomputerowymi/blob/main/Lab3/Screenshots/WebsiteHTTP.png?raw=true)

## Serwis dostępny lokalnie - phpmyadmin [2 pkt]
### Zmodyfikowany plik compose.yaml:
``` yaml
services:
  ...
  # Poprzednie serwisy (do serwisu db została dodana sieć phpmyadmin)
  ...
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: "phpmyadmin"
    environment:
      PMA_HOST: db
    networks:
      - proxy
      - phpmyadmin
    labels:
      traefik.enable: true #Przekierowywanie przez Traefik
      traefik.docker.network: proxy #Domyślna sieć przekierowywania
      
      #HTTP
      traefik.http.routers.phpmyadmin.entrypoints: web #Entrypoint nasłuchujący na porcie 80
      traefik.http.routers.phpmyadmin.rule: Host(`phpmyadmin.jlatawiec-local.duckdns.org`) #Adres usługi

networks:
  ...
  # Poprzednie sieci
  ...
  phpmyadmin: #Sieć o nazwie phpmyadmin
    name: phpmyadmin
    ipam:
      driver: default
      config:
        - subnet: 172.18.117.0/24  #Adresacja sieci
```

### Wynik:
![Alt text](https://github.com/JakubLatawiec/AdministracjaSieciamiKomputerowymi/blob/main/Lab3/Screenshots/PhpMyAdminHTTP.png?raw=true)

## SSL Offloading with Traefik [3 pkt]
### Zmodyfikowany plik proxy/compose.yaml:
``` yaml
services:
  treafik:
    ...
    # Poprzednie ustawienia
    ...
    command:
      ...
      # Poprzenie komendy
      ...
      #DNS CHALLENGE
      - "--certificatesresolvers.letsencrypt.acme.dnschallenge=true" #Włączenie dns challenge
      - "--certificatesresolvers.letsencrypt.acme.dnschallenge.provider=duckdns" #Ustawienie providera na duckdns
      - "--certificatesresolvers.letsencrypt.acme.email=moj_adres@email.xyz" #Ustawienie adresu email zarejestrowanego w duckdns
      - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json" #Ustawienie ścieżki przechowywania certyfikatów w kontenerze
    
    volumes:
      - "./letsencrypt:/letsencrypt"  # Wolumen na przechowywanie certyfikatów
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
    environment:
      DUCKDNS_TOKEN: TUTAJ-BYL-MOJ-TOKEN-DUCKDNS
```

### Zmodyfikowany plik compose.production.yaml:
``` yaml
services:
  frontend:
    ...
    # Poprzednie ustawienia
    ...
    labels:
      ...
      # Poprzednie etykiety
      ...
      #HTTPS:
      traefik.http.routers.frontend-secure.entrypoints: websecure #Entrypoint nasłuchujący na porcie 443
      traefik.http.routers.frontend-secure.rule: Host(`frontend.jlatawiec.duckdns.org`) #Adres usługi
      traefik.http.routers.frontend-secure.tls: true #Włączenie tls
      traefik.http.routers.frontend-secure.tls.certresolver: letsencrypt #Ustawienie resolvera certyfikatów
      "traefik.http.routers.frontend-secure.tls.domains[0].main": jlatawiec.duckdns.org #Ustawnienie głównej domeny
      "traefik.http.routers.frontend-secure.tls.domains[0].sans": "*.jlatawiec.duckdns.org" #Ustawienie sub domen
```

### Zmodyfikowany plik compose.yaml:
``` yaml
services:
  ...
  # Poprzednie serwisy
  ...
  phpmyadmin:
    ...
    # Poprzednie ustawienia
    ...
    labels:
      ...
      # Poprzednie etykiety
      ...
      # HTTPS
      traefik.http.routers.phpmyadmin-secure.entrypoints: websecure
      traefik.http.routers.phpmyadmin-secure.rule: Host(`phpmyadmin.jlatawiec-local.duckdns.org`)
      traefik.http.routers.phpmyadmin-secure.tls: true
      traefik.http.routers.phpmyadmin-secure.tls.certresolver: letsencrypt
      "traefik.http.routers.phpmyadmin-secure.tls.domains[0].main": jlatawiec-local.duckdns.org
      "traefik.http.routers.phpmyadmin-secure.tls.domains[0].sans": "*.jlatawiec-local.duckdns.org"
```
### Wynik:
VSCode przekirował port 443 na 39697:
![Alt text](https://github.com/JakubLatawiec/AdministracjaSieciamiKomputerowymi/blob/main/Lab3/Screenshots/WebsiteHTTPS.png?raw=true)
![Alt text](https://github.com/JakubLatawiec/AdministracjaSieciamiKomputerowymi/blob/main/Lab3/Screenshots/PhpMyAdminHTTPS.png?raw=true)
