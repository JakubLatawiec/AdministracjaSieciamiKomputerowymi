#Użyj gotowego obrazu node w wersji LTS
FROM node:lts

#Ustaw domyślny folder aplikacji w kontenerze
WORKDIR /app

#Skopiuj pliki aplikacji
COPY package*.json ./
COPY *.config.js ./
COPY *.json ./

#Zainstaluj odpowiednie biblioteki
RUN npm install

EXPOSE 3333

#Uruchom aplikacje
CMD ["npm", "start"]
