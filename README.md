# Smart Clinic Management System

Smart Clinic Management System is a full-stack Java application for a small clinic. It brings doctor search, appointment booking, and prescription management into separate dashboards for patients, doctors, and administrators.

This project was built for IBM's *Java Development Capstone Project* on Coursera. It covers user stories, database design, Spring Boot REST APIs, a browser interface, SQL reporting, and containerized local development.

## What you can do

| Role | Features |
| --- | --- |
| Patient | Register and sign in, search for doctors by name, specialty, or availability, and book, reschedule, or cancel appointments. |
| Doctor | Sign in, view scheduled appointments and patient records, and add or view prescriptions. |
| Administrator | Sign in and manage the clinic's list of doctors. |

## How it works

- **Backend** — Java 17, Spring Boot, Spring MVC, Spring Data JPA, Spring Data MongoDB, and JWT-based login.
- **Frontend** — Thymeleaf templates, HTML, CSS, and vanilla JavaScript.
- **Storage** — MySQL stores accounts, availability, and appointments. MongoDB stores prescriptions linked to appointment IDs.
- **Tooling** — Maven builds the application. Docker Compose runs the application and both databases. GitHub Actions workflows cover backend compilation and linting for Java, frontend files, and the Dockerfile.

## Run locally

Install Docker with the Compose plugin and OpenSSL. Run the following from the root of the repository, where `compose.databases.yaml` is located.

Generate local credentials for a fresh checkout. The command preserves an existing `.env` file, which is ignored by Git.

```bash
if [ ! -e .env ]; then
  (
    umask 077
    printf 'MYSQL_ROOT_PASSWORD=%s\nMYSQL_PASSWORD=%s\nMONGO_ROOT_PASSWORD=%s\nJWT_SECRET=%s\n' \
      "$(openssl rand -hex 16)" "$(openssl rand -hex 16)" \
      "$(openssl rand -hex 16)" "$(openssl rand -hex 32)" > .env
  )
fi

docker compose -f compose.databases.yaml up --build -d
docker compose -f compose.databases.yaml ps
```

Open the role selection page once the application starts. [Open Smart Clinic locally](http://localhost:8080/)

You can also check the public doctor list from a terminal. An empty list is expected with a fresh database.

```bash
curl -fsS http://localhost:8080/doctor
```

### Try the three dashboards

On a fresh database, create a demo administrator after the application has initialized the MySQL tables. Open the MySQL prompt with the account configured by Compose.

```bash
docker compose -f compose.databases.yaml exec mysql sh -c \
  'MYSQL_PWD="$MYSQL_PASSWORD" exec mysql --user="$MYSQL_USER" --database="$MYSQL_DATABASE"'
```

Replace the password placeholder with a throwaway local password before running the SQL.

```sql
INSERT INTO admin (username, password)
VALUES ('demo-admin', 'REPLACE_WITH_A_LOCAL_DEMO_PASSWORD');
```

Sign in as the administrator to add a doctor. Use the credentials entered for that doctor to open the doctor dashboard. Register a patient from the patient portal, then search for the doctor and book an available slot.

For startup diagnostics, use `docker compose -f compose.databases.yaml logs app`. To stop the stack while keeping the database volumes, use `docker compose -f compose.databases.yaml down`.

## Project files

- [User stories](user_stories.md) for the three roles.
- [Database design](schema-design.md) for MySQL and MongoDB.
- [Application](app/) with the Maven project, Spring Boot sources, dashboards, and Dockerfile.
- [Reporting procedures](sql/stored_procedures.sql) and [example queries](sql/run_reports.sql) for daily, monthly, and yearly appointment reports.
- [Example reporting output](reports/stored_procedure_results.txt) from the capstone lab. The reports use sample appointments, which need to be loaded separately for a fresh database.

## Demo scope

Use synthetic data for this educational project. The lab authentication flow stores passwords in plain text and passes JWTs in some URLs. A deployment with real clinic data would require password hashing and an authentication flow that keeps tokens out of URLs.
