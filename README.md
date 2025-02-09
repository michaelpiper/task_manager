# Frappe Framework Setup Guide

**Author:** Michael Piper\
**Email:** [pipermichael@aol.com](mailto\:pipermichael@aol.com)

## Prerequisites

Before setting up Frappe, ensure you have the following installed:

- Python 3.10
- Node.js 20 (via NVM)
- Redis
- MariaDB
- Yarn
- wkhtmltopdf

## Installation Steps

### Linux (Ubuntu/Debian)

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install python3.10 python3.10-venv python3-pip
sudo apt install mariadb-server-10.6 mariadb-client-core-10.6
sudo mysql_secure_installation
sudo apt install redis-server
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
nvm install 20
nvm use 20
sudo apt install yarn
sudo apt install wkhtmltopdf
```

### Windows

1. Install [Python 3.10](https://www.python.org/downloads/).
2. Install [Redis](https://github.com/microsoftarchive/redis/releases).
3. Install [MariaDB](https://mariadb.org/download/).
4. Install [Node.js](https://nodejs.org/) using NVM:
   ```powershell
   nvm install 20
   nvm use 20
   ```
5. Install Yarn:
   ```powershell
   npm install -g yarn
   ```
6. Install wkhtmltopdf from [here](https://wkhtmltopdf.org/downloads.html).

### macOS

```bash
brew install python@3.10
brew install mariadb
brew install redis
brew install nvm
nvm install 20
nvm use 20
brew install yarn
brew install wkhtmltopdf
```

## Step 1: Create a Virtual Environment

```bash
mkdir frappe
cd frappe
python3 -m venv .venv
source .venv/bin/activate
```

## Step 2: Create the New Database User

Run the following SQL command to create the user `task_manage` with the password `password`:

```sql
CREATE USER 'task_manage'@'localhost' IDENTIFIED BY 'password';
```

### Grant Superuser Privileges

```sql
GRANT ALL PRIVILEGES ON *.* TO 'task_manage'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

### Verify the User

Check if the user was created successfully:

```sql
SELECT user, host FROM mysql.user;
```

Test the new user by logging in:

```bash
mysql -u task_manage -p
```

Enter the password `password` when prompted.

## Step 3: Install Frappe Bench

```bash
pip install frappe-bench
bench init frappe-bench --python python3.10
cd frappe-bench
```

## Step 4: Create a New Site

```bash
bench new-site task_manager.local
```

## Step 5: Create a New App

```bash
bench new-app task_manager
bench --site task_manager.local install-app task_manager
```

## Step 6: Create a "Task" DocType

Using CLI:

```bash
bench --site task_manager.local new-doctype Task
```

Using Frappe Interface:

1. Navigate to **Frappe Desk > Developer > DocType**.
2. Click **New**.
3. Enter "Task" as the name.
4. Add fields like `title`, `description`, `status`, `assigned_to`, and `due_date`.
5. Save the DocType.

## API Authentication

### Generate API Token

```bash
bench --site task_manager.local make-token user@example.com
```

### Include Token in API Requests

```json
{
  "Authorization": "token <api-token>"
}
```

## Database Schema for Project Management

### DocTypes

- **Project**: title, description, start\_date, end\_date
- **Task**: title, description, status, assigned\_to, due\_date, link to Project
- **User**: Default Frappe User DocType

### Relationships

- A Project has multiple Tasks.
- A Task is assigned to a User.

## Performance Optimization

### Optimize API Calls

- Use pagination:
  ```bash
  GET /api/resource/Task?limit_start=0&limit_page_length=20
  ```
- Fetch selective fields:
  ```bash
  GET /api/resource/Task?fields=["title", "status"]
  ```

### Implement Caching

```bash
sudo apt install redis-server
bench --site task_manager.local set-config redis_cache enabled
bench --site task_manager.local set-config redis_queue enabled
```

## Debugging

### Debugging Tools

```bash
bench --site task_manager.local --debug
```

### Example Solution for Slow API Responses

- Enable Redis caching.
- Optimize database queries.
- Use indexing for frequently queried fields.

