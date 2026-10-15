# Running the applications

Run each command in a separate terminal from the project root.

```powershell
cd backend
php artisan migrate
php artisan serve --host=0.0.0.0 --port=8000
```

```powershell
cd frontend
npm run dev
```

```powershell
cd mobile
npm start
```

Configure the database connection in `backend/.env` before running migrations.
Grant administrator access to an existing account with
`php artisan library:make-admin user@example.com`. The API enforces this role;
members cannot access administrator operations.

## Docker deployment

The root `Dockerfile` builds the Laravel API in `backend/`; the React frontend
and mobile app are separate applications and are not included in this image.
Build from the repository root. The container serves Apache on port `10000` by
default and uses the hosting platform's `PORT` value when one is provided.

For Render, create a Docker web service and configure its environment variables
in the Render dashboard, not in the Dockerfile or source control. Set
`APP_ENV=production`, `APP_DEBUG=false`, `APP_KEY`, `APP_URL`, and
`FRONTEND_URL`, plus `DB_CONNECTION=mysql`, `DB_HOST`, `DB_PORT`, `DB_DATABASE`,
`DB_USERNAME`, and `DB_PASSWORD` for the managed MySQL database. Generate a
unique application key with `php artisan key:generate --show` from `backend/`
and store the output as `APP_KEY` in the dashboard.

After the service can reach its database, run `php artisan migrate --force`
once (or use the platform's pre-deploy command). Migrations are deliberately
not run during image builds or every container start. The frontend must be
deployed separately with `VITE_API_URL` set to the deployed API's `/api` URL.
Because receipt photos use local storage by default, configure a persistent
disk or private S3-compatible storage if those uploads must survive container
replacements.

Run backend feature tests from `backend` with `php artisan test`. Tests use an
in-memory SQLite database and do not affect the development database.

To create two reusable demo members with five-day overdue penalties, run
`php artisan db:seed --class=Database\\Seeders\\DemoPenaltyAccountsSeeder` from
`backend`. The demo login password is `LibraryDemo!2026`; these accounts and
their dedicated test books are saved in the configured database.

For a physical phone, set `EXPO_PUBLIC_API_URL` in `mobile/.env` to the
computer's LAN address, for example `http://192.168.1.20:8000/api`. The default
`http://10.0.2.2:8000/api` address is for the Android emulator.

# Circulation behavior

Member requests are saved for administrator review. Approving a request creates
a 14-day loan and reserves one available copy. Its due date is shown to members
and administrators after approval. Confirming a return restores the copy;
overdue loans accrue a $2.50-per-day penalty. The member balance includes fees
on overdue books that have not yet been returned. Members can print a penalty
receipt, pay at the cashier, and submit a receipt photo from the mobile app.
Administrators can review the photo from the web or mobile admin interface,
approve the submitted payment to clear its amount, or decline it with a notice
so the member can submit a replacement photo. Receipt photos are stored
privately and can only be viewed by the receipt owner or an administrator.
Local storage is the default for development and single-server deployments.
For scalable or multi-server production, configure a private S3-compatible
bucket and set `FILESYSTEM_DISK=s3`, `AWS_ACCESS_KEY_ID`,
`AWS_SECRET_ACCESS_KEY`, `AWS_DEFAULT_REGION`, and `AWS_BUCKET` in
`backend/.env`. For S3-compatible providers, also set `AWS_ENDPOINT`; set
`AWS_USE_PATH_STYLE_ENDPOINT=true` when the provider requires path-style URLs.
Keep the bucket private and do not commit credentials.

Administrator member registration requires a first name, last name, email, and
School ID; the middle name is optional. New accounts receive a temporary
password that must be shared securely. Members and administrators can change
their own password from their profile section. Profile details are currently
read-only.

Circle is a library-specific assistant that searches the live catalog, answers
questions about a member's requests and active loans, and can submit a borrow
request from an available-book recommendation. It uses the application's
library data and supported help topics; it is not a general-purpose AI chatbot.
