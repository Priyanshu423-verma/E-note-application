# E‑Notes — JSP/Servlet/JDBC (Advanced Java)

A simple, production‑ready **E‑Notes web application** built with **JSP**, **Servlets**, **JDBC**, and **Bootstrap**. Users can register, sign in, and securely create, edit, search, and delete personal notes.

> If you’re browsing this on GitHub, feel free to copy this entire README and adapt the placeholders (⚙️ Replace‑me) to your project.

---

## ✨ Features

* **User authentication** (register/login/logout, hashed passwords)
* **CRUD notes** (create, read, update, delete)
* **Search & filter** notes by title/content/date
* **Responsive UI** with Bootstrap 5
* **Secure sessions & CSRF-safe forms**
* **DAO pattern** with JDBC + Connection Pooling
* **Pagination** & friendly errors
* **Config via `db.properties` or `context.xml`**

---

## 🧰 Tech Stack

* **Language:** Java 8+ (works with 11/17)
* **Web:** JSP, Servlets (Jakarta/Javax)
* **DB:** MySQL/MariaDB (JDBC)
* **UI:** Bootstrap 5, Font Awesome
* **Build:** Maven (WAR packaging)
* **Server:** Apache Tomcat 9/10.1

> ⚠️ Choose **javax** (Tomcat 9) *or* **jakarta** (Tomcat 10.1+) namespaces consistently.

---

## 🏗️ Architecture (Overview)

```
Browser ↔ Servlet Controllers ↔ Service (Business Rules) ↔ DAO ↔ DB
                ↓
              JSP Views (JSTL/EL)
```

* **Servlets** handle requests & session auth
* **Services** validate input & enforce ownership
* **DAO** performs SQL with prepared statements
* **JSP** renders views with JSTL/EL

---

## 📁 Project Structure (suggested)

```
E-Notes/
├─ pom.xml
├─ src/main/java/
│  └─ com/example/enotes/
│     ├─ controller/              # Servlets (LoginServlet, NoteServlet, etc.)
│     ├─ dao/                     # UserDao, NoteDao
│     ├─ model/                   # User, Note POJOs
│     ├─ service/                 # AuthService, NoteService
│     └─ util/                    # DBUtil, PasswordUtil (BCrypt), Validation
├─ src/main/resources/
│  └─ db.properties               # DB creds (or use context.xml)
├─ src/main/webapp/
│  ├─ WEB-INF/
│  │  ├─ web.xml                  # Servlet mappings (if not using annotations)
│  │  └─ views/                   # JSPs under WEB-INF (prevent direct access)
│  ├─ assets/                     # css, js, images
│  └─ index.jsp
└─ README.md
```

---

## 🗄️ Database Schema (SQL)

```sql
-- ⚙️ Replace‑me: database name
CREATE DATABASE IF NOT EXISTS enotes DEFAULT CHARACTER SET utf8mb4;
USE enotes;

-- Users
CREATE TABLE IF NOT EXISTS users (
  id           BIGINT PRIMARY KEY AUTO_INCREMENT,
  name         VARCHAR(100) NOT NULL,
  email        VARCHAR(120) NOT NULL UNIQUE,
  password_hash VARCHAR(255) NOT NULL,
  created_at   TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Notes
CREATE TABLE IF NOT EXISTS notes (
  id           BIGINT PRIMARY KEY AUTO_INCREMENT,
  user_id      BIGINT NOT NULL,
  title        VARCHAR(200) NOT NULL,
  content      TEXT NOT NULL,
  pinned       TINYINT(1) DEFAULT 0,
  created_at   TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at   TIMESTAMP NULL DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Helpful indexes
CREATE INDEX idx_notes_user ON notes(user_id, created_at);
CREATE FULLTEXT INDEX ftx_notes_text ON notes(title, content);
```

---

## 🔑 Configuration

### Option A — `src/main/resources/db.properties`

```
jdbc.url=jdbc:mysql://localhost:3306/enotes?useSSL=false&serverTimezone=UTC
jdbc.username=⚙️your_user
jdbc.password=⚙️your_password
jdbc.pool.size=10
```

Load it via a small `DBUtil` using `HikariCP` (recommended) or `BasicDataSource`.

### Option B — `META-INF/context.xml` (Tomcat DataSource)

```xml
<Context>
  <Resource name="jdbc/enotes" auth="Container"
            type="javax.sql.DataSource" maxTotal="20" maxIdle="10"
            username="⚙️your_user" password="⚙️your_password"
            driverClassName="com.mysql.cj.jdbc.Driver"
            url="jdbc:mysql://localhost:3306/enotes?useSSL=false&serverTimezone=UTC"/>
</Context>
```

Then lookup via JNDI: `java:comp/env/jdbc/enotes`.

---

## 🚀 Quick Start

1. **Clone**

   ```bash
   git clone https://github.com/⚙️your-org/⚙️enotes.git
   cd enotes
   ```
2. **Create DB & tables**

   ```bash
   mysql -u root -p < docs/sql/schema.sql   # or paste the SQL above
   ```
3. **Configure DB** (`db.properties` *or* `context.xml`)
4. **Build WAR**

   ```bash
   mvn clean package
   ```
5. **Deploy to Tomcat**

   * Copy `target/enotes.war` → `TOMCAT_HOME/webapps/`
   * Start Tomcat: `bin/startup.sh` or `startup.bat`
6. **Open the app**

   * [http://localhost:8080/⚙️enotes](http://localhost:8080/⚙️enotes)

> **Jakarta vs Javax**: If using Tomcat 10.1+, migrate imports to `jakarta.servlet.*` and set `<packaging>war</packaging>` + appropriate dependencies.

---

## 🧩 Maven (example)

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>enotes</artifactId>
  <version>1.0.0</version>
  <packaging>war</packaging>

  <properties>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>
    <!-- Servlet API (choose one family) -->
    <!-- For Tomcat 9 (javax): -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>4.0.1</version>
      <scope>provided</scope>
    </dependency>

    <!-- For Tomcat 10.1+ (jakarta): -->
    <!--
    <dependency>
      <groupId>jakarta.servlet</groupId>
      <artifactId>jakarta.servlet-api</artifactId>
      <version>6.0.0</version>
      <scope>provided</scope>
    </dependency>
    -->

    <!-- JSTL -->
    <dependency>
      <groupId>jakarta.servlet.jsp.jstl</groupId>
      <artifactId>jakarta.servlet.jsp.jstl-api</artifactId>
      <version>2.0.0</version>
    </dependency>
    <dependency>
      <groupId>org.glassfish.web</groupId>
      <artifactId>jakarta.servlet.jsp.jstl</artifactId>
      <version>2.0.0</version>
    </dependency>

    <!-- JDBC Driver -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.3.0</version>
    </dependency>

    <!-- Connection Pool (optional but recommended) -->
    <dependency>
      <groupId>com.zaxxer</groupId>
      <artifactId>HikariCP</artifactId>
      <version>5.1.0</version>
    </dependency>

    <!-- BCrypt for password hashing -->
    <dependency>
      <groupId>org.mindrot</groupId>
      <artifactId>jbcrypt</artifactId>
      <version>0.4</version>
    </dependency>
  </dependencies>

  <build>
    <finalName>enotes</finalName>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.11.0</version>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-war-plugin</artifactId>
        <version>3.4.0</version>
      </plugin>
    </plugins>
  </build>
</project>
```

---

## 🧭 URL Routes (Servlet Mappings)

```xml
<!-- web.xml (if not using @WebServlet) -->
<web-app ...>
  <servlet>
    <servlet-name>LoginServlet</servlet-name>
    <servlet-class>com.example.enotes.controller.LoginServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>LoginServlet</servlet-name>
    <url-pattern>/login</url-pattern>
  </servlet-mapping>

  <servlet>
    <servlet-name>NoteServlet</servlet-name>
    <servlet-class>com.example.enotes.controller.NoteServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>NoteServlet</servlet-name>
    <url-pattern>/notes/*</url-pattern>
  </servlet-mapping>
</web-app>
```

**Example endpoints**

* `POST /login` — authenticate
* `POST /register` — create account
* `GET /notes` — list notes (with pagination & search)
* `POST /notes` — create note
* `GET /notes/{id}` — view note
* `POST /notes/{id}/update` — update note
* `POST /notes/{id}/delete` — delete note

---

## 🔒 Security

* Hash passwords with **BCrypt**
* Use **PreparedStatement** everywhere
* Store JSPs under **`/WEB-INF/views`**
* Add **CSRF tokens** for form posts (store token in session)
* Set `HttpOnly`, `Secure`, `SameSite` cookies
* Validate & sanitize all user input

---

## 🧪 Testing

* **DAO tests** with an in‑memory DB (H2) or a test schema
* **Controller tests** via `HttpServletRequest`/`HttpServletResponse` mocks
* UI tests with Selenium (optional)

---

## 📸 Screenshots

Add `docs/screenshots/` and reference them here:

| Page        | Screenshot                                   |
| ----------- | -------------------------------------------- |
| Login       | ![Login](docs/screenshots/login.png)         |
| Dashboard   | ![Dashboard](docs/screenshots/dashboard.png) |
| Create Note | ![Create](docs/screenshots/create-note.png)  |

---

## 🧯 Troubleshooting

* **`HTTP 404` after deploy** → Check context path: `http://localhost:8080/enotes/` and WAR name.
* **`No suitable driver`** → Ensure `mysql-connector-java` is on classpath (not `provided`).
* **`java.lang.ClassNotFoundException: com.mysql.cj.jdbc.Driver`** → Add driver dependency & correct driver class.
* **DB timezone/auth errors** → Append `serverTimezone=UTC` and ensure account has rights.
* **JSTL tag not found** → Use both JSTL API & impl dependencies (see Maven above).

---

## 🗺️ Roadmap

* Tags & categories for notes
* File attachments (images/PDF)
* Share notes (read‑only links)
* Dark mode toggle
* REST API + SPA front‑end

---

## 🤝 Contributing

1. Fork the repo & create a feature branch: `git checkout -b feat/your-feature`
2. Commit with conventional messages
3. Open a PR with a clear description & screenshots

---

## 📄 License

This project is licensed under the **MIT License** — see `LICENSE`.

---

## 🙌 Acknowledgements

* Bootstrap team for the CSS framework
* JSTL maintainers
* Tomcat & MySQL communities

---

## 🔗 Project Links

* **Repository:** [https://github.com/⚙️your-org/⚙️enotes](https://github.com/⚙️your-org/⚙️enotes)
* **Live Demo:** https\://⚙️your-domain.com (optional)
* **Issues:** [https://github.com/⚙️your-org/⚙️enotes/issues](https://github.com/⚙️your-org/⚙️enotes/issues)

---

## 📚 Appendix: Sample JSP Snippet (list notes)

```jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt" %>
<table class="table table-striped">
  <thead><tr><th>Title</th><th>Updated</th><th></th></tr></thead>
  <tbody>
    <c:forEach items="${notes}" var="n">
      <tr>
        <td>${n.title}</td>
        <td><fmt:formatDate value="${n.updatedAt}" pattern="yyyy-MM-dd HH:mm"/></td>
        <td class="text-end">
          <a href="${pageContext.request.contextPath}/notes/${n.id}" class="btn btn-sm btn-outline-primary">View</a>
          <form action="${pageContext.request.contextPath}/notes/${n.id}/delete" method="post" class="d-inline">
            <input type="hidden" name="_csrf" value="${csrfToken}"/>
            <button class="btn btn-sm btn-outline-danger">Delete</button>
          </form>
        </td>
      </tr>
    </c:forEach>
  </tbody>
</table>
```

## 📚 Appendix: DAO Snippet (prepared statement)

```java
public Optional<Note> findById(long id, long ownerId) throws SQLException {
  String sql = "SELECT id, user_id, title, content, pinned, created_at, updated_at FROM notes WHERE id=? AND user_id=?";
  try (Connection con = ds.getConnection();
       PreparedStatement ps = con.prepareStatement(sql)) {
    ps.setLong(1, id);
    ps.setLong(2, ownerId);
    try (ResultSet rs = ps.executeQuery()) {
      if (rs.next()) return Optional.of(mapper(rs));
      return Optional.empty();
    }
  }
}
```

---

> ✅ **Next step:** Replace all ⚙️ placeholders (repo, DB creds, domain) and push this README to your GitHub project.
