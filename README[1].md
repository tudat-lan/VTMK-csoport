# VTMK Csoport

Magyar felületű, önállóan üzemeltethető hang- és szöveges közösségi rendszer. Ez saját fejlesztésű alkalmazás, nem TeamSpeak termék vagy kompatibilis szerver.

## Funkciók

- Tagregisztráció, bejelentkezés, SQLite adatbázis és admin/moderátor szerepkörök.
- Hang- és szövegcsatornák, chat, jelenlét és LiveKit/WebRTC hang.
- Képernyőmegosztás közvetlen WebRTC peer-to-peer videóval. A szerver csak jelzést/kapcsolat-egyeztetést továbbít: a megosztónak külön TLS/WSS jelzőportja van (7900), a nézőnek külön TLS/WSS jelzőportja (7901); a videó nem halad át az alkalmazásszerveren.
- Windows kliens rögzített `https://vtmk.hu` központtal; saját, eredeti VTMK jelvény és ikon.

## Ubuntu + Portainer telepítés

1. A Portainer Stacknek a teljes projektkönyvtárat el kell érnie (Dockerfile, `server/`, `client/`, Compose fájl). Válaszd a Stacks → Add stack → Repository módot.
2. Állítsd be a `.env.example` változóit Portainerben, egyedi, legalább 32 karakteres JWT és LiveKit titkokkal.
3. A webes HTTPS reverse proxy irányítsa az appot az `app:8080` címre, a LiveKit websocketet az `livekit:7880` címre. TLS szükséges.
4. A host Nginx terminálja a TLS-t és a websocket forgalmat is. Példa a host Nginx `http` konfigurációs részére:

```nginx
server {
  listen 7900 ssl;
  server_name vtmk.hu;
  ssl_certificate /etc/letsencrypt/live/vtmk.hu/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/vtmk.hu/privkey.pem;
  location / { proxy_pass http://127.0.0.1:17900; proxy_http_version 1.1; proxy_set_header Upgrade $http_upgrade; proxy_set_header Connection "upgrade"; }
}
server {
  listen 7901 ssl;
  server_name vtmk.hu;
  ssl_certificate /etc/letsencrypt/live/vtmk.hu/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/vtmk.hu/privkey.pem;
  location / { proxy_pass http://127.0.0.1:17901; proxy_http_version 1.1; proxy_set_header Upgrade $http_upgrade; proxy_set_header Connection "upgrade"; }
}
```

5. A tűzfalon a webhez TCP 80/443, jelzéshez TCP 7900/7901, LiveKithez TCP 7881 és UDP 50000–60000 szükséges. A 17900/17901 csak a helyi gépen érhető el. A képernyő videója a böngészők közötti WebRTC ICE kapcsolat UDP útján halad; STUN segíti a közvetlen kapcsolat létrehozását.
6. SQLite adat a `voiceforge_data` Docker volume-ban tárolódik. Első regisztrált felhasználó admin lesz.

Ubuntu 18.04 régi, már nem támogatott Docker-host verzió. Ha a Docker Engine vagy a LiveKit nem indul, telepíts frissebb Ubuntu LTS-t.

## Windows kliens build

Node.js 20+ alatt:

```powershell
npm --prefix client install
npm --prefix client run dist:win
```

A kliens a `client/dist` könyvtárba készül. A backendben a WebSocket csak jelzőüzeneteket közvetít. A TLS reverse proxy és a két nyilvános jelzőport szükséges. Szigorú vagy szimmetrikus NAT esetén a közvetlen peer-to-peer kapcsolat meghiúsulhat; TURN relay nincs konfigurálva, ezért ilyen hálózatokon a megosztás nem fog működni.

Ez a változat még nem teljes TeamSpeak funkcióparitás: például részletes jogosultságöröklés, privát szoba, közvetlen chat, fájlátvitel, könyvjelző-szinkron és push-to-talk nincs benne. A hang LiveKit/WebRTC, nem TeamSpeak protokoll.

## Licenc

A saját forráskód MIT licencű; a függőségek külön licencek szerint használhatók. Nincs beépített licenclejárat vagy fizetős licenckulcs.
