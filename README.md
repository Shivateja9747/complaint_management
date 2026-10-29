# Complaint Management System

Students or employees raise complaints, receive a unique complaint ID and can check whether it is open, in progress or resolved.

> Stack: **React 18 (Vite) + Axios** · **Java 17 + Spring Boot 3.3.5 (Spring Web, Spring Data JPA/Hibernate, Bean Validation)** · **MySQL 8.x** · Maven

## 1. Problem statement (summary)

Submit a complaint (title, description, category) with a backend-generated unique ID and default status Open; list complaints; update status to Open / In Progress / Resolved; filter by status.

## 2. Features implemented

- Complaint form with category
- Backend-generated unique ID (`CMP-00001`, `CMP-00002` …) shown in the success message
- Status always starts as **Open**
- Status dropdown per complaint, filter by status, summary counters
- *Track a complaint* box: enter an ID to see its current status
- Resolved time is recorded automatically

## 3. Technologies and versions

| Layer | Technology |
|---|---|
| Frontend | React 18.3, Vite 5.4, Axios 1.7, plain CSS (responsive) |
| Backend | Java 17, Spring Boot 3.3.5, Spring Web, Spring Data JPA (Hibernate 6), Jakarta Bean Validation, Maven |
| Database | MySQL 8.x with MySQL Connector/J (managed by Spring Boot) |

## 4. Prerequisites

- JDK 17 or newer (`java -version`) and Maven 3.8+ (`mvn -version`) - or use IntelliJ IDEA's bundled Maven
- Node.js 18+ and npm (`node -v`, `npm -v`)
- MySQL Server 8.x **installed and running** on `localhost:3306`
- A MySQL user (for example `root`) that has privileges on database `complaint_management`
- VS Code or IntelliJ IDEA

## 5. Folder structure

```
16-Complaint-Management/
├── backend/
│   ├── src/
│   │   └── main/
│   │       ├── java/
│   │       │   └── com/
│   │       │       └── tt/
│   │       │           └── complaintmanagement/
│   │       │               ├── config/
│   │       │               │   └── CorsConfig.java
│   │       │               ├── controller/
│   │       │               │   └── ComplaintController.java
│   │       │               ├── dto/
│   │       │               │   ├── ComplaintRequest.java
│   │       │               │   ├── ComplaintStatusRequest.java
│   │       │               │   └── ComplaintSummary.java
│   │       │               ├── entity/
│   │       │               │   ├── Complaint.java
│   │       │               │   ├── ComplaintCategory.java
│   │       │               │   └── ComplaintStatus.java
│   │       │               ├── exception/
│   │       │               │   ├── BadRequestException.java
│   │       │               │   ├── ConflictException.java
│   │       │               │   ├── GlobalExceptionHandler.java
│   │       │               │   └── NotFoundException.java
│   │       │               ├── repository/
│   │       │               │   └── ComplaintRepository.java
│   │       │               ├── service/
│   │       │               │   └── ComplaintService.java
│   │       │               └── ComplaintManagementApplication.java
│   │       └── resources/
│   │           └── application.properties
│   ├── .gitignore
│   └── pom.xml
├── database/
│   ├── sample-data.sql
│   └── schema.sql
├── frontend/
│   ├── src/
│   │   ├── api/
│   │   │   ├── client.js
│   │   │   └── complaintApi.js
│   │   ├── components/
│   │   │   ├── ComplaintForm.jsx
│   │   │   └── Notice.jsx
│   │   ├── pages/
│   │   │   └── ComplaintsPage.jsx
│   │   ├── styles/
│   │   │   └── app.css
│   │   ├── App.jsx
│   │   └── main.jsx
│   ├── .env
│   ├── .gitignore
│   ├── index.html
│   ├── package.json
│   └── vite.config.js
└── README.md
```

## 6. MySQL setup

1. Make sure MySQL Server is running (Windows: *Services → MySQL80 → Start*; Linux: `sudo systemctl start mysql`; macOS: `brew services start mysql`).
2. This project uses its **own database**: `complaint_management`.
3. Create it **before starting the backend** (Hibernate creates tables, but not the database):

```sql
CREATE DATABASE IF NOT EXISTS complaint_management CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

Or run the ready-made script: `mysql -u root -p < database/schema.sql` (or open `database/schema.sql` in MySQL Workbench and execute it).

4. Tables are then created/updated automatically on the first backend start because `spring.jpa.hibernate.ddl-auto=update`. No SQL table scripts are executed by Spring (there is no `schema.sql` inside the backend), so the two strategies never conflict.

## 7. Configure `application.properties`

Open `backend/src/main/resources/application.properties` and change only these lines:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/complaint_management?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
spring.datasource.username=root
spring.datasource.password=YOUR_MYSQL_PASSWORD
```

- **Database name** → the word after `3306/` (keep `complaint_management` unless you created a different name).
- **Host / port** → `localhost:3306` (change if MySQL runs elsewhere or on another port).
- **Username / password** → your own MySQL credentials. Replace `YOUR_MYSQL_PASSWORD` with your real password (leave it empty if your root user has no password).
- No credentials are stored in Java code, and no environment variables are needed.

## 8. Run the backend (port 8080)

```bash
cd backend
mvn spring-boot:run
```

(or run `ComplaintManagementApplication` from IntelliJ IDEA / VS Code). When you see `Started ComplaintManagementApplication`, the API is live at **http://localhost:8080**.

## 9. Run the frontend (port 5173)

```bash
cd frontend
npm install
npm run dev
```

Open **http://localhost:5173**. Production build check: `npm run build`.

The API base URL is set in `frontend/.env`:

```
VITE_API_BASE_URL=http://localhost:8080/api
```

Change it only if the backend uses another host/port. Restart `npm run dev` after editing `.env`. CORS for `http://localhost:5173` is configured in `config/CorsConfig.java`.

## 10. REST API endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/complaints` | List complaints (newest first); optional filter ?status=OPEN|IN_PROGRESS|RESOLVED |
| GET | `/api/complaints/summary` | Counts of total, open, in-progress and resolved complaints |
| GET | `/api/complaints/track/{code}` | Look up a complaint by its generated ID, e.g. /api/complaints/track/CMP-00001 |
| POST | `/api/complaints` | Submit a complaint (title, description, category); the backend generates the ID and sets status OPEN |
| PATCH | `/api/complaints/{id}/status` | Update status to OPEN, IN_PROGRESS or RESOLVED, body {"status":"RESOLVED"} |

Errors are returned as JSON: `{ "timestamp", "status", "message", "errors": { "field": "message" } }` with HTTP 400 (validation), 404 (not found) or 409 (conflict/duplicate).

## 11. How to test the main features manually

1. Submit a complaint → message shows the generated ID (e.g. `CMP-00001`) and status *Open*.
2. Submit with an empty title → validation message.
3. Change the status dropdown to *In progress*, then *Resolved* → badge and counters change.
4. Choose *Resolved* in the filter → only resolved complaints.
5. Type `CMP-00001` in *Track a complaint* → status card appears; type an unknown ID → *No complaint found*.

## 12. Validation rules and business logic

- Title, description required (max 150 / 1000 characters); category must be a valid value
- Clients cannot set the ID or status when creating - the request DTO has no such fields
- ID = `CMP-` + 5-digit database id, saved in a unique column
- Changing to the same status → HTTP 409; unknown complaint → HTTP 404
- `resolvedAt` is set when the status becomes RESOLVED and cleared when reopened

## 13. Sample data

Optional sample rows are provided in `database/sample-data.sql`. Start the backend once first (so Hibernate creates the tables), then run: `mysql -u root -p < database/sample-data.sql`.

## 14. Common errors and solutions

| Problem | Solution |
|---|---|
| `Access denied for user 'root'@'localhost'` | Wrong username/password in `application.properties`. Test with `mysql -u root -p`. |
| `Unknown database 'complaint_management'` | Create it first: `CREATE DATABASE complaint_management;` (see section 6). |
| `Communications link failure` / `Connection refused` | MySQL is not running or uses a different port. Start MySQL and check the port in the JDBC URL. |
| `Port 8080 was already in use` | Stop the other program or set `server.port=8081` in `application.properties` **and** `VITE_API_BASE_URL=http://localhost:8081/api` in `frontend/.env`. |
| Port 5173 busy | Vite automatically picks 5174; then add that origin to `CorsConfig.java`, or free port 5173. |
| `Public Key Retrieval is not allowed` | Keep `allowPublicKeyRetrieval=true` in the JDBC URL (already included). |
| Browser shows "Cannot reach the backend" | Backend is not running, or `VITE_API_BASE_URL` is wrong. Open `http://localhost:8080/api/...` in the browser to check. |
| CORS error in browser console | Frontend is running on an origin other than 5173/4173; add it in `CorsConfig.java`. |
| `npm install` fails | Check the Node version (18+) and your internet connection. |


## 15. MySQL connection troubleshooting checklist

1. `mysql -u root -p` works from a terminal? If not, fix the password/service first.
2. `SHOW DATABASES;` lists `complaint_management`? If not, create it.
3. `SHOW GRANTS FOR 'root'@'localhost';` shows privileges on `complaint_management`? If you use another user, `GRANT ALL PRIVILEGES ON complaint_management.* TO 'user'@'localhost';`.
4. `SHOW VARIABLES LIKE 'port';` equals the port in the JDBC URL?
5. Special characters in the password are fine in `.properties`; do not wrap the password in quotes.
6. Read the **first** "Caused by:" line of the Spring Boot stack trace - it names the real MySQL problem.

## 16. How React, Spring Boot, REST, JPA and MySQL work together

1. The **React** component calls an **Axios** function (`src/api/client.js`) such as `api.get("/...")`.
2. Axios sends an HTTP request (JSON) to the **Spring Boot** REST controller in `controller/`.
3. The controller validates input (`@Valid`) and delegates to a **service** that holds the business rules.
4. The service uses a **Spring Data JPA repository**; **Hibernate** converts entity objects to SQL.
5. **MySQL** stores the rows permanently; the result travels back as JSON and React re-renders the screen.

## 17. Viva preparation

**Q1. Where and how is the complaint ID generated?**  
In `ComplaintService.submit`: after the first save gives the auto-increment id, the code `CMP-%05d` is built and saved; the column is unique.

**Q2. How do you ensure the default status is Open?**  
The service sets `OPEN` itself; `ComplaintRequest` does not accept a status.

**Q3. Why a unique constraint on `complaintCode`?**  
Guarantees uniqueness at database level even if two requests happen at once.

**Q4. How does tracking work?**  
`GET /api/complaints/track/{code}` → `findByComplaintCodeIgnoreCase`.

**Q5. What is `@PathVariable String code` used for?**  
Reads the code from the URL path.

**Q6. How are the counters calculated?**  
`countByStatus` for each status, returned by `/summary`.

**Q7. Why enums for category and status?**  
Restrict values to a fixed valid set and store them as readable strings.

**Q8. Why does the UI refresh after each status change?**  
It calls `load()` to get the updated list and counters from the database.
#   c o m p l a i n t _ m a n a g e m e n t  
 