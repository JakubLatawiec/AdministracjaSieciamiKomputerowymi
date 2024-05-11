#Użyj gotowego obrazu node w wersji LTS
FROM node:lts

#Ustaw domyślny folder aplikacji w kontenerze
WORKDIR /app

#Skopiuj pliki aplikacji
COPY . .

#Zainstaluj odpowiednie biblioteki
RUN npm install

#Uruchom aplikacje
CMD ["npm", "run", "dev"]
