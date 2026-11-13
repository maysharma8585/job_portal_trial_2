# Launchboard — job portal

A responsive job portal with a vanilla web frontend and a C++17/Crow REST API. The backend owns validation, password hashing, authorization, job management, applications, and database access.

## Technology

- Frontend: HTML5, CSS3, JavaScript (ES6+)
- Backend: C++17 / Crow Framework
- Primary database: MySQL 8 (ODBC)
- Alternative database: Microsoft Access (ODBC)
- API: REST / JSON

## Run locally

1. Install CMake 3.20+, a C++17 compiler, Git, OpenSSL, standalone Asio (`libasio-dev` on Debian/Ubuntu, or `vcpkg install asio` on Windows), and unixODBC (Linux/macOS) or the Windows ODBC development components. Install the MySQL 8 ODBC driver and create a database named `launchboard`.
2. Copy `.env.example` to `.env` and set `DATABASE_CONNECTION` to your ODBC connection string. The server reads environment variables; export them in your shell or load `.env` with your preferred environment loader.
3. Build and run:

   ```sh
   cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
   cmake --build build --config Release
   ./build/launchboard
   ```

   On Windows, run `build\Release\launchboard.exe` (or `build\launchboard.exe` for a single-configuration generator).
4. Visit [http://localhost:8080](http://localhost:8080). The API and static frontend are served by the same process. Tables are created on startup.

### Access alternative

Install the Microsoft Access Database Engine ODBC driver matching the server process architecture, set `DATABASE_DRIVER=access`, and point `DATABASE_CONNECTION` at an `.accdb` file. The app creates its tables on startup. Access is intended for local/single-user development; MySQL is recommended for multi-user use.

## API

All API routes are under `/api`. Authenticated routes take `Authorization: Bearer <token>`.

| Method | Route | Purpose |
| --- | --- | --- |
| GET | `/api/health` | API/database health |
| POST | `/api/auth/register` | Create candidate or employer account |
| POST | `/api/auth/login` | Sign in |
| GET | `/api/auth/me` | Current account |
| GET | `/api/jobs` | Search and filter published jobs |
| GET | `/api/jobs/:id` | Job details |
| POST | `/api/jobs` | Employer creates a job |
| PUT | `/api/jobs/:id` | Employer updates own job; admins may update any |
| DELETE | `/api/jobs/:id` | Employer deletes own job; admins may delete any |
| POST | `/api/jobs/:id/apply` | Candidate applies |
| GET | `/api/applications` | Candidate's applications or employer's received applications |
| PUT | `/api/applications/:id/status` | Employer updates application status |
| GET | `/api/admin/stats` | Admin dashboard counts |

The initial admin account is not seeded. Promote an account after registering by setting its `role` to `admin` directly in the `users` table. This avoids shipping a known privileged password.

## Notes

- Passwords are stored as salted PBKDF2-HMAC-SHA256 hashes; API bearer tokens are random and expire after 12 hours.
- The browser stores its bearer token in `sessionStorage`. Use HTTPS and a production-grade session strategy before public deployment.
- CMake fetches Crow and nlohmann/json during the first build, so network access is needed at configure time unless those dependencies are already cached. Crow's standalone Asio dependency must be installed separately.
