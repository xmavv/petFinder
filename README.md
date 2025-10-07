# 🦮 PetFinder - Lost+found pets
Aplikacja mobilna do zgłaszania zaginięć i odnajdywania zwierząt domowych z wykorzystaniem sztucznej inteligencji i analizy obrazu w celu walidacji i automatyzacji zdjęć oraz przyspieszeniu procesu odnalezienia zagubionego pupila.

# Audience Award winners on Team Projects Conference 2025 finale 🎉🎉🎉
[WIT](https://wit.pwr.edu.pl/aktualnosci/nasi-studenci-z-nagroda-publicznosci-kpz-447.html)
[PWR](https://pwr.edu.pl/uczelnia/aktualnosci/najlepsze-projekty-zespolowe-studentow-w4-i-w1211-13815.html)

### 🎯 Cel projektu
Celem systemu jest wsparcie właścicieli zaginionych psów poprzez:
* zgłaszanie zaginięć
* zgłaszanie znalezionych zwierząt
* automatyczne dopasowanie zdjęć przy pomocy modelu AI
* wymianę danych kontaktowych między właścicielem a znalazną

### 🚄 Komponenty serwera (FastAPI):
* auth - rejestracja, logowanie, JWT
* users - zarządzanie kontem
* reports - zgłoszenia zwierząt (lost/found)
* photos - przesyłanie i walidacja zdjęć
* ai_model - integracja z modelem ResNet50
* matching - wyszukiwanie podobieństwa > 70%
* rate_limiting - ograniczenie zapytać
* validation - filtrowanie niepoprawnych treści

### 🌆 Widoki
Projekt wizualnej identyfikacji znajduje się na [BEHANCE](https://www.behance.net/gallery/231665569/znalepsisko)

Przykładowe widoki aplikacji mobilnej:
* ekran logowania/rejestracji:
<img width="393" height="852" alt="homescreen" src="https://github.com/user-attachments/assets/1da60830-4e1b-4cc2-8d0d-7e828f68ba13" />

* formularz zgłoszeniowy - dodawanie raportu o zaginionym bądź odnalezionym zwierzęciu:
<img width="393" height="973" alt="newreport" src="https://github.com/user-attachments/assets/6758c58d-3dcd-4c3e-8ea3-6244c6dcca36" />

* lista dopasowanych zwierząt - zwierzęta odnalezione/zagubione z największym procentem podobieństwa:
<img width="393" height="852" alt="report" src="https://github.com/user-attachments/assets/e30990b7-c4fb-4f8b-9bb5-fa156f584221" />

* profil użytkownika i zarządzanie ostatnimi zgłoszeniami:
<img width="393" height="852" alt="lastreports" src="https://github.com/user-attachments/assets/7ff7c9e7-9d19-4bb3-829c-3f16387e1671" />


### 🔧 Technologie
- frontend: React Native (Expo)
- backend: FastAPI + Uvicorn
- baza danych: PostgreSQL + wtyczka `pgvector`
- model AI: CNN ResNet50 (embedding 2048D)

### 🔐 Zabezpieczenia
* Hasła szyfrowane `SHA256 + salt`
* Autoryzacja tokenowa `JWT`
* Rate limiting - maksymalnie 10 zapytań/minutę na endpointy AI
* Walidacja treści - akceptowane tylko zdjęcia psów (YOLO)

## Uruchomienie samego frontendu poprzez `apk`
Frontend aplikacji jest dostępny w zakładce `Releases`, gdzie dostępny jest plik `PetFinder-v1.0.0.apk`
Po pobraniu pliku, możliwe jest zapoznanie się z frontendem naszej aplikacji, jednak bez połączenia z backendem.

## Odpalanie projektu na Google Cloud
1. Pobierz i zainstaluj aplikację dostępową `gcloud CLI` (https://cloud.google.com/sdk/docs/install)
2. Zalogowanie się do aplikacji:
`gcloud auth login`
`gcloud auth application-default login`
3. Ustawienie projektu jako aktywny:
`gcloud config set project <id projektu google cloud>`
4. Aktywuj autentykację dla artifact registry:
`gcloud auth configure-docker eruope-central2-docker.pkg.dev`
5. Wypchnij obraz do repozytorium:
`docker push europe-central2-docker.pkg.dev/<id projektu google cloud>/pets-ai/pets-ai:latest`

### Pobieranie obrazu dockera na maszynie wirtualnej
1. Na lokalnej, zautentykowanej maszynie wygeneruj token uwierzytelniania:
`gcloud auth print-access-token`
2. Na maszynie wirtualnej uruchom:
`docker login -u oauth2accesstoken -p "<token uwierzytelnienia>" https://europe-central2-docker.pkg.dev`
3. Na maszynie wirtualnej uruchom:
`docker pull europe-central2-docker.pkg.dev/<id projektu google cloud>/pets-ai/pets-ai:latest`

### Tworzenie kontenera aplikacji serwerowej (backend na VM)
Na maszynie wirtualnej uruchom:
``docker run -d \
  --name backend \
  --link postgres:postgres \
  -p 80:8000 \
  -e PORT=8000 \
  -e POSTGRES_HOST=postgres \
  -e POSTGRES_PORT=5432 \
  -e POSTGRES_DB=lost+found \
  -e POSTGRES_USER=lost+found \
  -e POSTGRES_PASSWORD=<hasło do bazy danych> \
  -e AUTH_SECRET=<sekret uwierzytelniania> \
  europe-central2-docker.pkg.dev/<id projektu google cloud>/pets-ai/pets-ai:latest``

### Umożliwienie ruchu HTTP na porcie 80
1. Uruchom polecenie:
``gcloud compute firewall-rules create allow-http \
  --allow tcp:80 \
  --target-tags http-server \
  --description="Allow HTTP traffic on port 80"``
2. Dodaj tag do instancji maszyny wirtualnej poprzez uruchomienie polecenia:
``gcloud compute instances add-tags pets-ai \
  --tags=http-server --zone=europe-central2-c``

## Uruchomienie projektu lokalnie
1. Utwórz plik .end.local:
``PORT=8000
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=lost_found
POSTGRES_USER=lost_found
POSTGRES_PASSWORD=secret
AUTH_SECRET=change_me
``
2. Uruchomienie bazy danych na Dockerze:
``docker run -d \
  --name postgres_local \
  -e POSTGRES_DB=lost_found \
  -e POSTGRES_USER=lost_found \
  -e POSTGRES_PASSWORD=secret \
  -p 5432:5432 \
  postgres:16
``
3. Zbudowanie obrazu backendu:
`docker build -t pets-ai:local .`
4. Uruchomienie backendu:
``docker run -d \
  --name pets-ai-backend \
  --env-file .env.local \
  -p 8000:8000 \
  pets-ai:local
``
5. Migracja bazy, w przypadku, gdy nie jest uruchamiana przy odpaleniu obrazu:
``docker exec -it pets-ai-backend alembic upgrade head
``
6. Aplikacja będzie dostępna pod poniższym adresem:
`http://localhost:8000/docs`
