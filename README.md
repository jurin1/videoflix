# Videoflix: Unified Setup and Overview

This repository provides a centralized overview and combined installation instructions for the Videoflix project, which comprises a backend and a frontend.  Videoflix is a learning project replicating Netflix functionality using Angular and Django.

## Projects

*   **[Videoflix Backend](https://github.com/jurin1/videoflix_backend):**  The Django-based REST API that powers the application.
*   **[Videoflix Frontend](https://github.com/jurin1/videoflix_frontend):** The Angular-based user interface.

## Demo

*   **Live Demo:** [Videoflix Live Demo](https://videoflix.neizcon.de).  This demo showcases the core functionality of Videoflix.
*   *Note: The live demo may have limited functionality or be subject to periodic resets.*

## Prerequisites

Before you begin, ensure you have the following installed:

*   **Git:**  For cloning the repositories.  Get it from [https://git-scm.com/](https://git-scm.com/).
*   **Node.js and npm:** For the Frontend (Angular) project.  Download from [https://nodejs.org/](https://nodejs.org/).  An LTS (Long Term Support) version of Node.js is recommended.
*   **Python 3.11+:** For the Backend (Django) project. Download from [https://www.python.org/downloads/](https://www.python.org/downloads/).
*   **pip:**  Python package installer (usually comes with Python).
*   **Angular CLI:** Install globally: `npm install -g @angular/cli`
*   **PostgreSQL:**  Recommended database for production.  See installation instructions below.
*   **Redis:**  Required for Celery task queue (backend). See installation instructions below.
*   **FFmpeg:** Required for video conversion and thumbnail generation on the backend.  Download from [https://ffmpeg.org/](https://ffmpeg.org/) and ensure it's in your system's PATH.

## Installation

Follow these steps to set up both the backend and frontend components:

### 1. Clone the Repositories

```bash
git clone https://github.com/jurin1/videoflix_backend
git clone https://github.com/jurin1/videoflix_frontend
```

### 2. Install PostgreSQL

*   **Ubuntu/Debian:**

    ```bash
    sudo apt update
    sudo apt install postgresql postgresql-contrib
    ```

*   **macOS (using Homebrew):**

    ```bash
    brew install postgresql
    ```

    After installation, start the PostgreSQL service: `brew services start postgresql`

*   **Windows:**

    Download the installer from [https://www.postgresql.org/download/windows/](https://www.postgresql.org/download/windows/) and follow the instructions.

*   **Create a Database and User:**

    After installation, connect to the PostgreSQL server (e.g., using `psql` or pgAdmin) and create a database and user for the Videoflix backend.  Replace `<database_name>`, `<database_user>`, and `<password>` with your desired values.

    ```sql
    CREATE DATABASE <database_name>;
    CREATE USER <database_user> WITH PASSWORD '<password>';
    GRANT ALL PRIVILEGES ON DATABASE <database_name> TO <database_user>;
    ```

### 3. Install Redis

*   **Ubuntu/Debian:**

    ```bash
    sudo apt update
    sudo apt install redis-server
    ```

*   **macOS (using Homebrew):**

    ```bash
    brew install redis
    ```

    After installation, start the Redis service: `brew services start redis`

*   **Windows:**

    Download the installer from [https://github.com/microsoftarchive/redis/releases](https://github.com/microsoftarchive/redis/releases).  Alternatively, use WSL (Windows Subsystem for Linux) and install using the Ubuntu/Debian instructions.

    *Note: Ensure that the Redis server is running on its default port (6379).*

### 4. Backend Setup (Videoflix Backend)

1.  **Navigate to the backend directory:**

    ```bash
    cd videoflix_backend
    ```

2.  **Create a virtual environment (recommended):**

    ```bash
    python -m venv venv
    ```

3.  **Activate the virtual environment:**

    *   Windows:

        ```bash
        venv\Scripts\activate
        ```

    *   macOS/Linux:

        ```bash
        source venv/bin/activate
        ```

4.  **Install dependencies:**

    ```bash
    pip install -r requirements.txt
    ```

    *If a `requirements.txt` file does not yet exists you can create it by following the [Videoflix Backend](https://github.com/jurin1/videoflix_backend) instructions.*

5.  **Configure environment variables:**

    *   Copy the `.env.example` file to `.env`:

        ```bash
        cp .env.example .env
        ```

    *   Edit the `.env` file with your specific settings.  **Crucially, configure the database settings to use PostgreSQL:**

        ```
        DATABASE_ENGINE=django.db.backends.postgresql
        DATABASE_NAME=<database_name>
        DATABASE_USER=<database_user>
        DATABASE_PASSWORD=<password>
        DATABASE_HOST=localhost
        DATABASE_PORT=5432

        SECRET_KEY=your_django_secret_key  # Generate a strong key for production
        CELERY_BROKER_URL=redis://localhost:6379/0 # Redis connection URL
        FRONTEND_PASSWORD_RESET_URL=http://localhost:4200/password-reset  #Adjust to frontend URL

        # Email settings (if using email features)
        EMAIL_BACKEND=django.core.mail.backends.smtp.EmailBackend
        EMAIL_HOST=smtp.example.com
        EMAIL_PORT=587
        EMAIL_USE_TLS=True
        EMAIL_HOST_USER=your_email@example.com
        EMAIL_HOST_PASSWORD=your_email_password
        DEFAULT_FROM_EMAIL=your_email@example.com
        ```

    *   Replace `<database_name>`, `<database_user>`, `<password>`, `your_django_secret_key`, `your_email@example.com` and `your_email_password`  with your actual values.  **Important:** Generate a strong and unique `SECRET_KEY` for production deployments.

6.  **Run database migrations:**

    ```bash
    python manage.py migrate
    ```

7.  **Start the development server:**

    ```bash
    python manage.py runserver
    ```

    The backend API should be accessible at http://localhost:8000/.

8.  **Start Celery Worker and Beat (in separate terminals):**

    ```bash
    celery -A videoflix_backend worker -l info
    celery -A videoflix_backend beat -l info --scheduler django_celery_beat.schedulers:DatabaseScheduler
    ```

### 5. Frontend Setup (Videoflix Frontend)

1.  **Navigate to the frontend directory:**

    ```bash
    cd ../videoflix_frontend
    ```

2.  **Install dependencies:**

    ```bash
    npm install
    ```

3.  **Configure environment variables:**

    *   Edit `src/environments/environment.ts` (for development) or `src/environments/environment.prod.ts` (for production) to point to your backend API URL:

        ```typescript
        // src/environments/environment.ts
        export const environment = {
            production: false,
            apiUrl: 'http://localhost:8000/api' // Backend API URL
        };
        ```

        *Important: Ensure the apiUrl ends with `/api`.*

4.  **Start the development server:**

    ```bash
    ng serve
    ```

    The frontend should be accessible at http://localhost:4200/.

### 6. Accessing the Application

Once both the backend and frontend are running, you should be able to access the Videoflix application in your browser at `http://localhost:4200/`.

## Common Issues

*   **Database Connection Errors:**  Double-check your PostgreSQL credentials in the `.env` file.  Ensure the PostgreSQL server is running.  Verify that your PostgreSQL user has the necessary privileges on the database.
*   **Redis Connection Errors:** Verify that the Redis server is running. Check the `CELERY_BROKER_URL` setting in the `.env` file.
*   **CORS Errors:** If you encounter CORS errors in the browser, ensure that you have configured the Django CORS settings correctly in the backend (using the `django-cors-headers` package).  The `ALLOWED_HOSTS` settings and `CORS_ORIGIN_WHITELIST` settings might need to be configured.
*   **FFmpeg Errors:** If video processing fails, confirm that FFmpeg is installed correctly and is accessible in your system's PATH.
*   **Frontend API Connection Errors:** Double-check the `apiUrl` setting in the `environment.ts` or `environment.prod.ts` file.

## Contributing

Contributions are welcome!  Please follow the contribution guidelines in the respective backend and frontend repositories.

## License

This project is licensed under the MIT License.  See the `LICENSE` file in each repository for details.



