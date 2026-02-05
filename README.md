# cicd-api (Spring Boot + Profiles + CI/CD)

Spring Boot -pohjainen REST API, joka käyttää tietokantaa ja erillisiä konfiguraatioprofiileja (**dev / test / prod**).
Projektissa on CI/CD-putki GitHub Actionsilla: testit ajetaan automaattisesti, Docker-image rakennetaan ja pusketaan Docker Hubiin, ja sovellus deployataan CSC cPouta -virtuaalikoneelle docker-composella.

## Arkkitehtuuri lyhyesti
- **Spring Boot REST API**
- **Tietokanta:** PostgreSQL (Docker Compose -palveluna)
- **Profiles:**
  - `dev` = paikallinen kehitys (Docker Compose)
  - `test` = testit (kevyt erilliskonfiguraatio)
  - `prod` = tuotanto CSC:llä (Docker Compose + PostgreSQL)

---

## Paikallinen kehitys (dev)

### Esivaatimukset
- Docker Desktop
- Git

### Käynnistys
1) Kloonaa repo:
```bash
git clone https://github.com/mimosa1234/cicd-api.git
cd cicd-api
Käynnistä dev-ympäristö:

docker-compose -f docker-compose.dev.yml up -d
Tarkista että palvelut ovat käynnissä:

docker-compose -f docker-compose.dev.yml ps
API:n testaus paikallisesti
Health endpoint:

curl -i http://localhost:8080/api/health
Testaus (test-profiili)
Paikallisesti
Aja testit test-profiililla:

SPRING_PROFILES_ACTIVE=test ./mvnw test
Miksi erillinen test-profiili?

Testit eivät käytä dev/prod-tietokantaa (turvallisempi)

Testit ovat nopeampia ja toistettavia (esim. in-memory DB)

CI-ajo pysyy deterministisenä

GitHub Actionsissa
CI-putkessa testit ajetaan aina test-profiililla, tyypillisesti näin:

SPRING_PROFILES_ACTIVE=test

./mvnw test

Konfiguraatioprofiilit (Spring Profiles)
Projektissa käytetään vähintään seuraavia profiileja:

dev
Tarkoitus: paikallinen kehitys

Aktivointi:

SPRING_PROFILES_ACTIVE=dev
Dev-ympäristössä tietokanta ajetaan Docker Composella.

test
Tarkoitus: automaattiset testit

Aktivointi:

SPRING_PROFILES_ACTIVE=test
Käytössä testiajossa (paikallisesti ja CI:ssä).

prod
Tarkoitus: tuotanto CSC:llä

Aktivointi: docker-compose.prod.yml asettaa prod-profiilin (ympäristömuuttuja tai vastaava).

Tuotannossa tietokanta tulee docker-composen db-palvelusta (ei localhost).

CI/CD (GitHub Actions)
Putki tekee seuraavat vaiheet:

Checkout koodi

Ajaa testit test-profiililla

Rakentaa Docker-imagen

Pushaa imagen Docker Hubiin

Deployaa CSC:lle SSH:n kautta:

hakee uudet imaget

käynnistää/päivittää palvelut docker-composella

Tuotanto (CSC cPouta)
Tuotannossa ajetaan:

docker-compose.prod.yml (PostgreSQL + Spring Boot app)

Sovellus on julkaistu porttiin 80 (host) ja ohjautuu containerin porttiin 8080.

Tuotanto-URL
Health endpoint:

http://195.148.20.184/api/health

Tuotannon pikatesti
curl -i http://195.148.20.184/api/health
CSC:llä hyödylliset debug-komennot
cd ~/deploy/cicd-api
docker-compose -f docker-compose.prod.yml pull
docker-compose -f docker-compose.prod.yml up -d
docker-compose -f docker-compose.prod.yml ps
docker logs --tail=200 cicd-api_app_1
Tärkeät tiedostot
docker-compose.dev.yml – paikallinen dev-ympäristö

docker-compose.prod.yml – tuotanto CSC:llä

application-dev.yml, application-test.yml, application-prod.yml – profiilikohtaiset asetukset

.github/workflows/* – CI/CD-workflowt
