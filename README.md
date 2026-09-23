# Meeting Room Booking System

Eit komplett bookingsystem for møterom med React, Express og PostgreSQL.

Denne README-fila finst på to språk:

- [Nynorsk](#nynorsk)
- [Русский](#русский)

---

<a id="nynorsk"></a>

# Nynorsk

## Innhald

1. [Om prosjektet](#om-prosjektet)
2. [Funksjonar](#funksjonar)
3. [Teknologi og arkitektur](#teknologi-og-arkitektur)
4. [Føresetnader](#føresetnader)
5. [Prosjektstruktur](#prosjektstruktur)
6. [Rask lokal oppstart](#rask-lokal-oppstart)
7. [Detaljert databaseoppsett](#detaljert-databaseoppsett)
8. [Miljøvariablar](#miljøvariablar)
9. [E-post og Gmail App Password](#e-post-og-gmail-app-password)
10. [Migrasjonar og testdata](#migrasjonar-og-testdata)
11. [Start, kontroll og omstart](#start-kontroll-og-omstart)
12. [Produksjonsbygg og Azure](#produksjonsbygg-og-azure)
13. [Tryggleik og backup](#tryggleik-og-backup)
14. [Feilsøking](#feilsøking)

## Om prosjektet

Systemet lèt tilsette reservere møterom, sjå dagens reservasjonar og administrere rom, brukarar, firma og tilgangar. Brukargrensesnittet er på nynorsk. README-fila inneheld òg ei fullstendig russisk rettleiing.

I utviklingsmodus køyrer frontend og backend som to separate prosessar:

```text
Nettlesar -> React/Vite på port 5173 -> Express API på port 4000 -> PostgreSQL
```

I produksjonsmodus blir frontend bygd inn i `backend/public`. Express leverer då både API-et og den ferdige nettsida frå éin prosess.

## Funksjonar

- registrering og innlogging med e-post og passord;
- JWT-basert autentisering;
- roller for administrator, brukar og visningsbrukar;
- kviteliste for godkjende e-postadresser;
- oppretting, endring og kansellering av reservasjonar;
- valfri gjesteinformasjon og kommentarar;
- fleksibel varigheit og kontroll av overlappande reservasjonar;
- administrasjon av møterom, fargar, bilete og deaktiveringsårsak;
- administrasjon av firma og firmalogoar;
- historikk som blir bevart når relaterte data blir sletta;
- offentleg dagsvising på `/display`;
- sekssifra kode for tilbakestilling av passord;
- PostgreSQL-avgrensingar som vernar dataintegriteten.

## Teknologi og arkitektur

| Del | Teknologi |
|---|---|
| Frontend | React 18, Vite, React Router |
| Backend | Node.js, Express |
| Database | PostgreSQL 15 eller nyare |
| Autentisering | JWT og bcrypt |
| E-post | Nodemailer med Gmail SMTP |
| Filopplasting | Multer, lokale filer i `backend/uploads` |
| Produksjon | Express leverer `backend/public` |
| CI/CD | GitHub Actions og Azure Web App |

Lokale adresser:

| Teneste | Adresse |
|---|---|
| Frontend | `http://localhost:5173` |
| Backend/API | `http://localhost:4000` |
| Helsesjekk | `http://localhost:4000/health` |
| Offentleg vising | `http://localhost:5173/display` |

## Føresetnader

Installer dette før du startar:

- Git;
- Node.js 20 LTS eller nyare, med npm;
- PostgreSQL 15 eller nyare;
- `psql`, eller pgAdmin dersom du føretrekkjer eit grafisk verktøy;
- ein Gmail-konto med tofaktorautentisering dersom systemet skal sende ekte e-post.

Kontroller installasjonane:

```bash
git --version
node --version
npm --version
psql --version
```

På Windows ligg `psql.exe` vanlegvis her:

```text
C:\Program Files\PostgreSQL\18\bin\psql.exe
```

Dersom `psql` ikkje er i `PATH`, bruk den fulle stien eller køyr SQL i Query Tool i pgAdmin.

## Prosjektstruktur

```text
.
├── backend/
│   ├── env.example             # Mal for backend/.env
│   ├── public/                 # Produksjonsbygg frå frontend
│   ├── scripts/                # Verktøy for enkeltmigrasjonar
│   ├── src/                    # Express API, ruter og tenester
│   └── uploads/                # Opplasta rom- og firmabilete
├── database/
│   └── migrations/             # SQL-migrasjonar 001–013
├── docs/
│   └── AZURE_DEPLOY.md         # Utfyllande Azure-rettleiing
├── frontend/
│   └── src/                    # React-applikasjonen
├── scripts/
│   └── run-migrations.js       # Produksjonsmigrasjonar i rett rekkefølgje
└── .github/workflows/
    └── main_bookingmoterom.yml # Bygg og distribusjon til Azure
```

## Rask lokal oppstart

Dette er kortversjonen for ei ny lokal database. Dei neste kapitla forklarer alle stega.

### 1. Klon prosjektet

```bash
git clone https://github.com/DaniilCS-20-2/Meeting-Room-Booking-System-Release.git
cd Meeting-Room-Booking-System-Release
```

### 2. Installer pakkar

```bash
cd backend
npm ci
cd ../frontend
npm ci
cd ..
```

`npm ci` brukar dei låste versjonane i `package-lock.json`. Bruk `npm install` berre dersom du medvite vil oppdatere låsefila.

### 3. Opprett database og brukar

```bash
psql -U postgres
```

```sql
CREATE ROLE ferma_app WITH LOGIN PASSWORD 'SETT_INN_EIT_STERKT_PASSORD';
CREATE DATABASE booking_app_db
  WITH OWNER = ferma_app
       ENCODING = 'UTF8'
       TEMPLATE = template0;
\q
```

### 4. Opprett `backend/.env`

PowerShell:

```powershell
Copy-Item .\backend\env.example .\backend\.env
```

macOS/Linux:

```bash
cp backend/env.example backend/.env
```

Rediger fila:

```dotenv
DATABASE_URL=postgresql://ferma_app:SETT_INN_DB_PASSORD@localhost:5432/booking_app_db
JWT_SECRET=SETT_INN_EIN_TILFELDIG_HEMMELEG_VERDI_PAA_MINST_32_TEIKN
NODE_ENV=development
PORT=4000
FRONTEND_URL=http://localhost:5173
ALLOWED_ORIGINS=
SMTP_USER=
SMTP_PASS=
ADMIN_EMAILS=admin@example.com
```

### 5. Køyr migrasjonane

PowerShell:

```powershell
$env:DATABASE_URL="postgresql://ferma_app:SETT_INN_DB_PASSORD@localhost:5432/booking_app_db"
node .\scripts\run-migrations.js
```

macOS/Linux:

```bash
export DATABASE_URL='postgresql://ferma_app:SETT_INN_DB_PASSORD@localhost:5432/booking_app_db'
node scripts/run-migrations.js
```

Skriptet køyrer `001` og `003`–`013`. `002_seed.sql` blir med vilje hoppa over fordi fila legg inn offentleg kjende demonstrasjonsdata.

### 6. Start begge prosessane

Terminal 1:

```bash
cd backend
npm run dev
```

Terminal 2:

```bash
cd frontend
npm run dev
```

Opne `http://localhost:5173`.

## Detaljert databaseoppsett

### Ny tom database

Tilrådd oppsett er ei eiga PostgreSQL-rolle utan superbrukarrettar:

```sql
CREATE ROLE ferma_app
  WITH LOGIN
       NOSUPERUSER
       NOCREATEDB
       NOCREATEROLE
       NOINHERIT
       PASSWORD 'SETT_INN_EIT_STERKT_PASSORD';

CREATE DATABASE booking_app_db
  WITH OWNER = ferma_app
       ENCODING = 'UTF8'
       TEMPLATE = template0;
```

Migrasjon `001_init.sql` brukar `pgcrypto` og `btree_gist`. Dersom databaseeigaren ikkje får opprette utvidingar, køyr dette éin gong som PostgreSQL-administrator i `booking_app_db`:

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS btree_gist;
```

### Database som allereie finst

Ta backup først. Opprett deretter ei avgrensa applikasjonsrolle som administrator:

```sql
CREATE ROLE ferma_app WITH LOGIN PASSWORD 'SETT_INN_EIT_STERKT_PASSORD';
GRANT CONNECT ON DATABASE booking_app_db TO ferma_app;
```

Kopla til `booking_app_db`, og køyr:

```sql
GRANT USAGE ON SCHEMA public TO ferma_app;
GRANT SELECT, INSERT, UPDATE, DELETE
  ON ALL TABLES IN SCHEMA public TO ferma_app;
GRANT USAGE, SELECT, UPDATE
  ON ALL SEQUENCES IN SCHEMA public TO ferma_app;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
  GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO ferma_app;
ALTER DEFAULT PRIVILEGES IN SCHEMA public
  GRANT USAGE, SELECT, UPDATE ON SEQUENCES TO ferma_app;
```

Køyr nye migrasjonar som databaseeigaren dersom `ferma_app` ikkje eig objekta som skal endrast. Sjølve applikasjonen bør framleis bruke den avgrensa rolla.

### Tilkoplingsstreng og pgAdmin

Formatet er `postgresql://BRUKAR:PASSORD@VERT:PORT/DATABASE`. Dersom passordet inneheld `@`, `:`, `/`, `?`, `#` eller `%`, må teikna URL-kodast. Migrasjonsskriptet kan alternativt bruke `PGHOST`, `PGPORT`, `PGUSER`, `PGPASSWORD` og `PGDATABASE`.

I pgAdmin:

1. Registrer PostgreSQL-tenaren.
2. Opprett eller vel `booking_app_db` under **Databases**.
3. Opne **Query Tool** for akkurat denne databasen.
4. Køyr SQL-kommandoane eller migrasjonsfilene i stigande nummerrekkefølgje.
5. Oppdater **Schemas > public > Tables**.

Pass på at Query Tool er kopla til `booking_app_db`, ikkje standarddatabasen `postgres`.

## Miljøvariablar

Backend les `backend/.env` ved oppstart. Fila er ignorert av Git og skal aldri committast.

| Variabel | Påkravd | Forklaring |
|---|---:|---|
| `DATABASE_URL` | Ja | PostgreSQL-tilkopling |
| `JWT_SECRET` | Ja | Signerer token; minst 32 tilfeldige teikn |
| `NODE_ENV` | Ja i produksjon | `development` eller `production` |
| `PORT` | Nei | Backend-port; standard og tilrådd lokalt er `4000` |
| `FRONTEND_URL` | Tilrådd | Primær tillaten frontend-origin |
| `ALLOWED_ORIGINS` | Ved fleire domene | Kommaseparert liste over ekstra origins |
| `SMTP_USER` | For e-post | Gmail-adressa som sender kodar |
| `SMTP_PASS` | For e-post | Gmail App Password, ikkje vanleg passord |
| `ADMIN_EMAILS` | Tilrådd | Kommaseparerte administratoradresser |

Generer `JWT_SECRET`:

```bash
node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
```

Når backend startar, blir adressene i `ADMIN_EMAILS` lagde i e-postkvitelista med administratorrolle. Ein ny brukar må likevel registrere seg med den godkjende adressa.

## E-post og Gmail App Password

Passordtilbakestilling sender ein sekssifra kode via Gmail SMTP. Utan SMTP-verdiar verkar resten av systemet, men sending av kode feilar.

1. Slå på tofaktorautentisering for Google-kontoen.
2. Gå til **Security > App passwords**.
3. Opprett eit app-passord, til dømes `Meeting Room Booking`.
4. Kopier det genererte 16-teikns passordet.
5. Set `SMTP_USER` til heile Gmail-adressa.
6. Set `SMTP_PASS` til app-passordet; mellomrom kan fjernast.
7. Start backend på nytt.

```dotenv
SMTP_USER=your-account@gmail.com
SMTP_PASS=abcdefghijklmnop
```

Feilen `Missing credentials for "PLAIN"` tyder at `SMTP_USER` eller `SMTP_PASS` er tom, manglar eller ikkje blei lesen frå `backend/.env`. Det vanlege Google-passordet verkar ikkje.

## Migrasjonar og testdata

| Fil | Innhald |
|---|---|
| `001_init.sql` | Grunnskjema, utvidingar, brukarar, rom, reservasjonar og kommentarar |
| `002_seed.sql` | Valfrie demonstrasjonsdata og testinnlogging |
| `003_flexible_duration.sql` | Fleksibel varigheit og tidsavgrensingar |
| `004_room_photos.sql` | Bilete for møterom |
| `005_email_whitelist.sql` | Kviteliste for registrering |
| `006_companies.sql` | Firma og firmatilknyting |
| `007_preserve_history.sql` | Bevaring av historiske data |
| `008_security_hardening.sql` | Tryggleiksforbetringar |
| `009_viewer_role.sql` | Rolle for visningstilgang |
| `010_room_color.sql` | Farge per møterom |
| `011_room_disabled_reason.sql` | Deaktiveringsårsak |
| `012_booking_guest_fields.sql` | Namn og e-post for gjester |
| `013_company_logo.sql` | Firmalogo |

Tilrådd migrasjonsskript stoppar ved første feil og hoppar over seed-fila. For lokale demonstrasjonsdata:

PowerShell:

```powershell
psql "$env:DATABASE_URL" -v ON_ERROR_STOP=1 -f .\database\migrations\002_seed.sql
```

macOS/Linux:

```bash
psql "$DATABASE_URL" -v ON_ERROR_STOP=1 -f database/migrations/002_seed.sql
```

```text
E-post: admin@ferma.no
Passord: admin123
```

Passordet er offentleg kjent. Ikkje bruk seed-fila i produksjon.

Manuell køyring av alle filer på ei heilt ny lokal database:

```powershell
Get-ChildItem .\database\migrations\*.sql |
  Sort-Object Name |
  ForEach-Object {
    psql "$env:DATABASE_URL" -v ON_ERROR_STOP=1 -f $_.FullName
    if ($LASTEXITCODE -ne 0) { throw "Migrering feila: $($_.Name)" }
  }
```

```bash
for file in database/migrations/*.sql; do
  psql "$DATABASE_URL" -v ON_ERROR_STOP=1 -f "$file" || exit 1
done
```

Ved oppgradering: ta backup, finn siste brukte migrasjon, køyr berre nyare filer i nummerrekkefølgje, stopp ved første feil og kontroller skjemaet. Prosjektet har ikkje ei eiga tabell som registrerer brukte migrasjonar; køyr ikkje alle filene blindt i produksjon.

## Start, kontroll og omstart

Start backend og frontend i kvar sin terminal:

```bash
cd backend
npm run dev
```

```bash
cd frontend
npm run dev
```

Frontend brukar automatisk `http://<same-vert>:4000` i utviklingsmodus.

Kontroller backend:

```powershell
Invoke-RestMethod http://localhost:4000/health
```

```bash
curl http://localhost:4000/health
```

Kontroller databasen:

```sql
SELECT current_database(), current_user;
SELECT COUNT(*) FROM public.rooms;
SELECT COUNT(*) FROM public.users;
SELECT COUNT(*) FROM public.bookings;
```

På ei ny database utan seed-data kan teljingane vere `0`. Kontroller òg innloggingssida, `/display`, `GET /api/display/today` og `cd frontend && npm run build`.

Trykk `Ctrl+C` i kvar terminal for å stoppe. Start backend på nytt etter endringar i `.env`. PostgreSQL treng vanlegvis ikkje omstart.

Etter `git pull`:

```bash
cd backend && npm ci
cd ../frontend && npm ci
cd ..
```

Kontroller om nye migrasjonar må brukast.

## Produksjonsbygg og Azure

```bash
cd frontend
npm ci
npm run build
cd ../backend
npm ci --omit=dev
npm start
```

Set produksjonsvariablane i plattforma, ikkje i Git:

```dotenv
NODE_ENV=production
PORT=4000
DATABASE_URL=postgresql://...
JWT_SECRET=...
FRONTEND_URL=https://ditt-domene.example
ALLOWED_ORIGINS=https://ekstra-domene.example
SMTP_USER=...
SMTP_PASS=...
ADMIN_EMAILS=...
```

Køyr `node scripts/run-migrations.js` frå prosjektrota før ny versjon startar. Ved `NODE_ENV=production` leverer Express React-bygget frå `backend/public`; Vite skal ikkje køyre separat.

GitHub Actions-fila `.github/workflows/main_bookingmoterom.yml` brukar Node.js 20, byggjer frontend og distribuerer til Azure Web App. GitHub må ha `AZURE_WEBAPP_PUBLISH_PROFILE`, og Azure må ha alle miljøvariablane. Sjå [docs/AZURE_DEPLOY.md](docs/AZURE_DEPLOY.md).

Etter distribusjon må du kontrollere `/health`, innlogging, database, CORS, e-post og varig lagring for `backend/uploads`.

## Tryggleik og backup

- Bruk ein databasebrukar utan superbrukarrettar.
- Bruk lange, tilfeldige passord og `JWT_SECRET` på minst 32 teikn.
- Ikkje legg `.env`, backupfiler, Gmail App Password eller publish profile i Git.
- Ikkje bruk `admin123` i produksjon.
- Set `NODE_ENV=production` og avgrens CORS til faktiske HTTPS-domene.
- Bruk TLS/SSL mot ekstern PostgreSQL.
- Ta regelmessig backup og bruk varig lagring for bilete.
- Roter ein hemmelegheit dersom ho blir vist i logg eller Git-historikk.

PowerShell-backup:

```powershell
$env:PGPASSWORD="SETT_INN_DB_PASSORD"
pg_dump -h localhost -p 5432 -U ferma_app -d booking_app_db -Fc -f .\booking_app_db.backup
Remove-Item Env:PGPASSWORD
```

macOS/Linux:

```bash
PGPASSWORD='SETT_INN_DB_PASSORD' pg_dump \
  -h localhost -p 5432 -U ferma_app -d booking_app_db \
  -Fc -f booking_app_db.backup
```

Gjenoppretting:

```bash
pg_restore -h localhost -p 5432 -U ferma_app -d booking_app_db --clean --if-exists booking_app_db.backup
```

`--clean` slettar eksisterande objekt. Ver sikker på måldatabasen.

## Feilsøking

### `Missing credentials for "PLAIN"`

`SMTP_USER` eller `SMTP_PASS` manglar. Bruk Gmail App Password og start backend på nytt. Dette er ein e-postfeil.

### `password authentication failed`

Kontroller rolle, passord, port og PostgreSQL-instans. URL-kod spesialteikn i passordet.

### `database "booking_app_db" does not exist`

Opprett databasen på tenaren som `DATABASE_URL` viser til, eller rett tilkoplingsstrengen.

### `relation ... does not exist`

Migrasjonane er ikkje køyrde mot same database som backend. Kontroller `DATABASE_URL`.

### `permission denied`

Bruk `GRANT`-kommandoane ovanfor eller køyr migrasjonane som databaseeigar.

### `psql` is not recognized

Legg PostgreSQL si `bin`-mappe i `PATH`, bruk pgAdmin eller full sti:

```powershell
& "C:\Program Files\PostgreSQL\18\bin\psql.exe" -U postgres -d booking_app_db
```

### Port `4000` eller `5173` er oppteken

Stopp den gamle prosessen med `Ctrl+C`. På Windows:

```powershell
Get-NetTCPConnection -LocalPort 4000,5173 -State Listen |
  Select-Object LocalPort, OwningProcess
Get-Process -Id PROCESS_ID
```

Kontroller programmet før du stoppar prosessen.

### Andre vanlege problem

- Frontend utan API: kontroller `/health`, port `4000`, CORS og HTTPS.
- Registrering avvist: adressa må finnast i `email_whitelist` eller `ADMIN_EMAILS`.
- Bilete forsvinn: bruk varig lagring for `backend/uploads`.
- Tom produksjonsside: køyr frontend-bygg og kontroller `backend/public/index.html`.
- `.env` verkar ikkje: start backend på nytt og kontroller at fila ikkje heiter `.env.txt`.

---

<a id="русский"></a>

# Русский

## Содержание

1. [О проекте](#о-проекте)
2. [Возможности](#возможности)
3. [Технологии и архитектура](#технологии-и-архитектура)
4. [Что нужно установить](#что-нужно-установить)
5. [Структура проекта](#структура-проекта)
6. [Быстрый локальный запуск](#быстрый-локальный-запуск)
7. [Подробная настройка базы данных](#подробная-настройка-базы-данных)
8. [Переменные окружения](#переменные-окружения)
9. [Почта и Gmail App Password](#почта-и-gmail-app-password)
10. [Миграции и тестовые данные](#миграции-и-тестовые-данные)
11. [Запуск, проверка и перезапуск](#запуск-проверка-и-перезапуск)
12. [Production и Azure](#production-и-azure)
13. [Безопасность и резервные копии](#безопасность-и-резервные-копии)
14. [Решение проблем](#решение-проблем)

## О проекте

Это система бронирования переговорных комнат. Сотрудники могут создавать и отменять бронирования, смотреть занятость комнат, а администраторы — управлять комнатами, пользователями, компаниями и доступом. Интерфейс приложения выполнен на нюнорске; README содержит полную инструкцию и на нюнорске, и на русском.

В режиме разработки frontend и backend запускаются отдельными процессами:

```text
Браузер -> React/Vite на порту 5173 -> Express API на порту 4000 -> PostgreSQL
```

В production frontend собирается в `backend/public`, после чего Express отдаёт и API, и готовый сайт из одного процесса.

## Возможности

- регистрация и вход по электронной почте и паролю;
- JWT-аутентификация;
- роли администратора, пользователя и пользователя только для просмотра;
- белый список разрешённых адресов электронной почты;
- создание, редактирование и отмена бронирований;
- необязательные данные гостя и комментарии;
- гибкая продолжительность и защита от пересекающихся бронирований;
- управление комнатами, цветами, фотографиями и причиной отключения;
- управление компаниями и логотипами;
- сохранение исторических данных после удаления связанных сущностей;
- публичный экран текущего дня по адресу `/display`;
- восстановление пароля с шестизначным кодом по электронной почте;
- ограничения PostgreSQL, защищающие целостность данных.

## Технологии и архитектура

| Часть | Технология |
|---|---|
| Frontend | React 18, Vite, React Router |
| Backend | Node.js, Express |
| База данных | PostgreSQL 15 или новее |
| Аутентификация | JWT и bcrypt |
| Электронная почта | Nodemailer и Gmail SMTP |
| Загрузка файлов | Multer, локальная папка `backend/uploads` |
| Production | Express раздаёт содержимое `backend/public` |
| CI/CD | GitHub Actions и Azure Web App |

Локальные адреса:

| Сервис | Адрес |
|---|---|
| Frontend | `http://localhost:5173` |
| Backend/API | `http://localhost:4000` |
| Проверка состояния | `http://localhost:4000/health` |
| Публичный экран | `http://localhost:5173/display` |

## Что нужно установить

- Git;
- Node.js 20 LTS или новее вместе с npm;
- PostgreSQL 15 или новее;
- `psql` либо pgAdmin;
- Gmail-аккаунт с двухфакторной аутентификацией, если нужна настоящая отправка кодов восстановления.

Проверьте версии:

```bash
git --version
node --version
npm --version
psql --version
```

В Windows `psql.exe` обычно находится здесь:

```text
C:\Program Files\PostgreSQL\18\bin\psql.exe
```

Если команда не найдена, добавьте папку `bin` PostgreSQL в `PATH`, укажите полный путь к `psql.exe` или используйте Query Tool в pgAdmin.

## Структура проекта

```text
.
├── backend/
│   ├── env.example             # Шаблон для backend/.env
│   ├── public/                 # Production-сборка frontend
│   ├── scripts/                # Запуск отдельной миграции
│   ├── src/                    # Express API, маршруты и сервисы
│   └── uploads/                # Фото комнат и логотипы
├── database/
│   └── migrations/             # SQL-миграции 001–013
├── docs/
│   └── AZURE_DEPLOY.md         # Дополнительная инструкция по Azure
├── frontend/
│   └── src/                    # React-приложение
├── scripts/
│   └── run-migrations.js       # Production-миграции по порядку
└── .github/workflows/
    └── main_bookingmoterom.yml # Сборка и публикация в Azure
```

## Быстрый локальный запуск

Ниже короткий сценарий для новой локальной базы. Следующие разделы подробно объясняют каждый шаг.

### 1. Клонируйте репозиторий

```bash
git clone https://github.com/DaniilCS-20-2/Meeting-Room-Booking-System-Release.git
cd Meeting-Room-Booking-System-Release
```

### 2. Установите зависимости

```bash
cd backend
npm ci
cd ../frontend
npm ci
cd ..
```

`npm ci` устанавливает версии из `package-lock.json`. Используйте `npm install`, только если намеренно хотите обновить lock-файлы.

### 3. Создайте базу и пользователя

Подключитесь администратором PostgreSQL:

```bash
psql -U postgres
```

```sql
CREATE ROLE ferma_app WITH LOGIN PASSWORD 'УКАЖИТЕ_СИЛЬНЫЙ_ПАРОЛЬ';
CREATE DATABASE booking_app_db
  WITH OWNER = ferma_app
       ENCODING = 'UTF8'
       TEMPLATE = template0;
\q
```

### 4. Создайте `backend/.env`

PowerShell:

```powershell
Copy-Item .\backend\env.example .\backend\.env
```

macOS/Linux:

```bash
cp backend/env.example backend/.env
```

Минимальный локальный пример:

```dotenv
DATABASE_URL=postgresql://ferma_app:УКАЖИТЕ_ПАРОЛЬ_БД@localhost:5432/booking_app_db
JWT_SECRET=УКАЖИТЕ_СЛУЧАЙНУЮ_СЕКРЕТНУЮ_СТРОКУ_ДЛИНОЙ_НЕ_МЕНЕЕ_32_СИМВОЛОВ
NODE_ENV=development
PORT=4000
FRONTEND_URL=http://localhost:5173
ALLOWED_ORIGINS=
SMTP_USER=
SMTP_PASS=
ADMIN_EMAILS=admin@example.com
```

### 5. Примените миграции

PowerShell:

```powershell
$env:DATABASE_URL="postgresql://ferma_app:УКАЖИТЕ_ПАРОЛЬ_БД@localhost:5432/booking_app_db"
node .\scripts\run-migrations.js
```

macOS/Linux:

```bash
export DATABASE_URL='postgresql://ferma_app:УКАЖИТЕ_ПАРОЛЬ_БД@localhost:5432/booking_app_db'
node scripts/run-migrations.js
```

Скрипт применяет `001` и `003`–`013`. `002_seed.sql` специально пропускается, потому что содержит общедоступные демонстрационные данные.

### 6. Запустите оба процесса

Терминал 1:

```bash
cd backend
npm run dev
```

Терминал 2:

```bash
cd frontend
npm run dev
```

Откройте `http://localhost:5173`.

## Подробная настройка базы данных

### Новая пустая база

Рекомендуется отдельная роль без прав суперпользователя:

```sql
CREATE ROLE ferma_app
  WITH LOGIN
       NOSUPERUSER
       NOCREATEDB
       NOCREATEROLE
       NOINHERIT
       PASSWORD 'УКАЖИТЕ_СИЛЬНЫЙ_ПАРОЛЬ';

CREATE DATABASE booking_app_db
  WITH OWNER = ferma_app
       ENCODING = 'UTF8'
       TEMPLATE = template0;
```

Миграция `001_init.sql` использует расширения `pgcrypto` и `btree_gist`. Если владелец базы не может устанавливать расширения, один раз выполните в `booking_app_db` от имени администратора:

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS btree_gist;
```

### Уже существующая база

Сначала сделайте резервную копию. Затем от имени администратора создайте ограниченную роль приложения:

```sql
CREATE ROLE ferma_app WITH LOGIN PASSWORD 'УКАЖИТЕ_СИЛЬНЫЙ_ПАРОЛЬ';
GRANT CONNECT ON DATABASE booking_app_db TO ferma_app;
```

Переключитесь именно на `booking_app_db` и выполните:

```sql
GRANT USAGE ON SCHEMA public TO ferma_app;
GRANT SELECT, INSERT, UPDATE, DELETE
  ON ALL TABLES IN SCHEMA public TO ferma_app;
GRANT USAGE, SELECT, UPDATE
  ON ALL SEQUENCES IN SCHEMA public TO ferma_app;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
  GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO ferma_app;
ALTER DEFAULT PRIVILEGES IN SCHEMA public
  GRANT USAGE, SELECT, UPDATE ON SEQUENCES TO ferma_app;
```

Если `ferma_app` не владеет объектами, новые миграции запускайте владельцем базы. Само приложение продолжайте запускать от ограниченной роли.

### Строка подключения и pgAdmin

Формат: `postgresql://ПОЛЬЗОВАТЕЛЬ:ПАРОЛЬ@ХОСТ:ПОРТ/БАЗА`. Символы `@`, `:`, `/`, `?`, `#` и `%` в пароле необходимо URL-кодировать. Скрипт миграций также понимает `PGHOST`, `PGPORT`, `PGUSER`, `PGPASSWORD` и `PGDATABASE`.

В pgAdmin:

1. Зарегистрируйте сервер PostgreSQL.
2. Создайте или выберите `booking_app_db` в **Databases**.
3. Откройте **Query Tool** именно для этой базы.
4. Выполните SQL-команды либо миграции по возрастанию номера.
5. Обновите **Schemas > public > Tables**.

Убедитесь, что Query Tool подключён к `booking_app_db`, а не к служебной базе `postgres`.

## Переменные окружения

Backend читает `backend/.env` при запуске. Файл исключён из Git и не должен попадать в commit.

| Переменная | Обязательна | Назначение |
|---|---:|---|
| `DATABASE_URL` | Да | Подключение к PostgreSQL |
| `JWT_SECRET` | Да | Подпись токенов; минимум 32 случайных символа |
| `NODE_ENV` | Да в production | `development` или `production` |
| `PORT` | Нет | Порт backend; локально рекомендуется `4000` |
| `FRONTEND_URL` | Рекомендуется | Основной разрешённый origin frontend |
| `ALLOWED_ORIGINS` | Для нескольких доменов | Дополнительные origins через запятую |
| `SMTP_USER` | Для почты | Gmail-адрес отправителя |
| `SMTP_PASS` | Для почты | Gmail App Password, не обычный пароль |
| `ADMIN_EMAILS` | Рекомендуется | Адреса администраторов через запятую |

Сгенерируйте `JWT_SECRET`:

```bash
node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
```

При старте backend адреса из `ADMIN_EMAILS` добавляются в белый список с ролью администратора. Если пользователя ещё нет, ему всё равно нужно зарегистрироваться.

## Почта и Gmail App Password

Восстановление пароля отправляет шестизначный код через Gmail SMTP. Без SMTP-настроек остальные функции работают, но отправка кода завершается ошибкой.

1. Включите двухфакторную аутентификацию Google.
2. Откройте **Security > App passwords**.
3. Создайте пароль приложения `Meeting Room Booking`.
4. Скопируйте 16-символьный пароль.
5. В `SMTP_USER` укажите полный Gmail-адрес.
6. В `SMTP_PASS` укажите App Password; пробелы можно убрать.
7. Перезапустите backend.

```dotenv
SMTP_USER=your-account@gmail.com
SMTP_PASS=abcdefghijklmnop
```

Ошибка `Missing credentials for "PLAIN"` означает, что `SMTP_USER` или `SMTP_PASS` отсутствуют, пусты либо не прочитаны из `backend/.env`. Обычный пароль Gmail не подходит.

## Миграции и тестовые данные

| Файл | Что добавляет |
|---|---|
| `001_init.sql` | Начальная схема, расширения, пользователи, комнаты, бронирования и комментарии |
| `002_seed.sql` | Необязательные демоданные и тестовая учётная запись |
| `003_flexible_duration.sql` | Гибкая продолжительность и ограничения времени |
| `004_room_photos.sql` | Фотографии комнат |
| `005_email_whitelist.sql` | Белый список для регистрации |
| `006_companies.sql` | Компании и привязка пользователей |
| `007_preserve_history.sql` | Сохранение исторических данных |
| `008_security_hardening.sql` | Усиление безопасности схемы |
| `009_viewer_role.sql` | Роль только для просмотра |
| `010_room_color.sql` | Цвет комнаты |
| `011_room_disabled_reason.sql` | Причина отключения комнаты |
| `012_booking_guest_fields.sql` | Имя и электронная почта гостя |
| `013_company_logo.sql` | Логотип компании |

Рекомендуемый скрипт останавливается при первой ошибке и не применяет seed-файл. Чтобы добавить демоданные только локально:

PowerShell:

```powershell
psql "$env:DATABASE_URL" -v ON_ERROR_STOP=1 -f .\database\migrations\002_seed.sql
```

macOS/Linux:

```bash
psql "$DATABASE_URL" -v ON_ERROR_STOP=1 -f database/migrations/002_seed.sql
```

```text
Email: admin@ferma.no
Пароль: admin123
```

Пароль публично известен. Не используйте seed-файл в production.

Ручное применение всех файлов к совершенно новой локальной базе:

```powershell
Get-ChildItem .\database\migrations\*.sql |
  Sort-Object Name |
  ForEach-Object {
    psql "$env:DATABASE_URL" -v ON_ERROR_STOP=1 -f $_.FullName
    if ($LASTEXITCODE -ne 0) { throw "Ошибка миграции: $($_.Name)" }
  }
```

```bash
for file in database/migrations/*.sql; do
  psql "$DATABASE_URL" -v ON_ERROR_STOP=1 -f "$file" || exit 1
done
```

При обновлении существующей базы: сделайте backup, определите последнюю применённую миграцию, запускайте только новые файлы по порядку, остановитесь при первой ошибке и проверьте схему. В проекте нет таблицы учёта миграций, поэтому не запускайте все файлы вслепую в production.

## Запуск, проверка и перезапуск

Запустите backend и frontend в разных терминалах:

```bash
cd backend
npm run dev
```

```bash
cd frontend
npm run dev
```

В development frontend автоматически использует `http://<тот-же-хост>:4000`.

Проверка backend:

```powershell
Invoke-RestMethod http://localhost:4000/health
```

```bash
curl http://localhost:4000/health
```

Проверка базы:

```sql
SELECT current_database(), current_user;
SELECT COUNT(*) FROM public.rooms;
SELECT COUNT(*) FROM public.users;
SELECT COUNT(*) FROM public.bookings;
```

В новой базе без seed-файла счётчики могут быть равны `0`. Проверьте также страницу входа, `/display`, `GET /api/display/today` и production-сборку командой `cd frontend && npm run build`.

Для остановки нажмите `Ctrl+C` в каждом терминале. После изменения `.env` перезапускайте backend. PostgreSQL обычно перезапускать не нужно.

После `git pull`:

```bash
cd backend && npm ci
cd ../frontend && npm ci
cd ..
```

После этого проверьте, появились ли новые миграции.

## Production и Azure

Сборка и запуск:

```bash
cd frontend
npm ci
npm run build
cd ../backend
npm ci --omit=dev
npm start
```

Задайте переменные в настройках сервера или облачной платформы, не в Git:

```dotenv
NODE_ENV=production
PORT=4000
DATABASE_URL=postgresql://...
JWT_SECRET=...
FRONTEND_URL=https://ваш-домен.example
ALLOWED_ORIGINS=https://дополнительный-домен.example
SMTP_USER=...
SMTP_PASS=...
ADMIN_EMAILS=...
```

Перед запуском новой версии выполните из корня `node scripts/run-migrations.js`. При `NODE_ENV=production` Express раздаёт React-сборку из `backend/public`; отдельный Vite-процесс не нужен.

Workflow `.github/workflows/main_bookingmoterom.yml` использует Node.js 20, собирает frontend и публикует приложение в Azure Web App. В GitHub должен быть secret `AZURE_WEBAPP_PUBLISH_PROFILE`, а в Azure — все переменные окружения. Подробнее: [docs/AZURE_DEPLOY.md](docs/AZURE_DEPLOY.md).

После публикации проверьте `/health`, вход, соединение с базой, CORS, почту и постоянное хранилище для `backend/uploads`.

## Безопасность и резервные копии

- Используйте отдельного пользователя БД без прав суперпользователя.
- Используйте длинные случайные пароли и `JWT_SECRET` не короче 32 символов.
- Не добавляйте `.env`, backup, Gmail App Password и Azure publish profile в Git.
- Не используйте `admin123` в production.
- Установите `NODE_ENV=production` и ограничьте CORS реальными HTTPS-доменами.
- Для удалённой PostgreSQL используйте TLS/SSL.
- Регулярно создавайте backup и используйте постоянное хранилище для изображений.
- Если секрет попал в журнал или Git-историю, немедленно замените его.

Backup в PowerShell:

```powershell
$env:PGPASSWORD="УКАЖИТЕ_ПАРОЛЬ_БД"
pg_dump -h localhost -p 5432 -U ferma_app -d booking_app_db -Fc -f .\booking_app_db.backup
Remove-Item Env:PGPASSWORD
```

macOS/Linux:

```bash
PGPASSWORD='УКАЖИТЕ_ПАРОЛЬ_БД' pg_dump \
  -h localhost -p 5432 -U ferma_app -d booking_app_db \
  -Fc -f booking_app_db.backup
```

Восстановление:

```bash
pg_restore -h localhost -p 5432 -U ferma_app -d booking_app_db --clean --if-exists booking_app_db.backup
```

`--clean` удаляет существующие объекты. Тщательно проверьте целевую базу.

## Решение проблем

### `Missing credentials for "PLAIN"`

Не заданы `SMTP_USER` или `SMTP_PASS`. Добавьте Gmail App Password в `backend/.env` и перезапустите backend. Это ошибка почты, не базы.

### `password authentication failed`

Проверьте роль, пароль, порт и экземпляр PostgreSQL. URL-кодируйте специальные символы в пароле.

### `database "booking_app_db" does not exist`

Создайте базу на сервере из `DATABASE_URL` либо исправьте имя в строке подключения.

### `relation ... does not exist`

Миграции применены не к той базе или не применены вовсе. Проверьте `DATABASE_URL`.

### `permission denied`

Выполните команды `GRANT` из раздела о существующей базе либо запускайте миграции владельцем базы.

### `psql` is not recognized

Добавьте `bin` PostgreSQL в `PATH`, используйте pgAdmin либо полный путь:

```powershell
& "C:\Program Files\PostgreSQL\18\bin\psql.exe" -U postgres -d booking_app_db
```

### Порт `4000` или `5173` занят

Остановите старый процесс через `Ctrl+C`. В Windows найдите PID:

```powershell
Get-NetTCPConnection -LocalPort 4000,5173 -State Listen |
  Select-Object LocalPort, OwningProcess
Get-Process -Id PROCESS_ID
```

Проверьте программу перед её остановкой.

### Другие частые проблемы

- Frontend открывается, API не отвечает: проверьте `/health`, порт `4000`, CORS и HTTPS.
- Регистрация отклоняется: адрес должен быть в `email_whitelist` или `ADMIN_EMAILS`.
- Изображения пропадают: подключите постоянное хранилище для `backend/uploads`.
- Пустая production-страница: соберите frontend и проверьте `backend/public/index.html`.
- Изменения `.env` не видны: перезапустите backend и проверьте, что файл не называется `.env.txt`.

---

## Лицензия / Lisens

Prosjektet er laga for intern bruk og læring. Avklar lisens og bruksvilkår med eigaren før offentleg vidarebruk.

Проект создан для внутреннего использования и обучения. Перед публичным переиспользованием уточните лицензию и условия у владельца.
