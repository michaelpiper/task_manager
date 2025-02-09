# Frappe Framework Setup Guide

**Author:** Michael Piper\
**Email:** [pipermichael@aol.com](mailto:pipermichael@aol.com)

## Prerequisites

Before setting up Frappe, ensure you have the following installed:

- Python 3.10
- Node.js 20 (via NVM)
- Redis
- MariaDB
- Yarn
- wkhtmltopdf
- Docker

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

## Setup task_manager.local on Localhost

### Linux/macOS

```bash
echo "127.0.0.1 task_manager.local" | sudo tee -a /etc/hosts
```

### Windows (Run as Administrator)

```powershell
Add-Content -Path "C:\Windows\System32\drivers\etc\hosts" -Value "127.0.0.1 task_manager.local"
```

## Docker Setup

### Install Docker

```bash
sudo apt install docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

### Run Frappe Using Docker

```bash
docker run -d -p 8000:8000 --name frappe-container frappe/bench:latest
```

## Step 1: Create a Virtual Environment

```bash
mkdir frappe
cd frappe
python3 -m venv .venv
source .venv/bin/activate
```

## Step 2: Create the New Database User

```sql
CREATE USER 'task_manage'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'task_manage'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

### Verify the User

```sql
SELECT user, host FROM mysql.user;
```

Test login:

```bash
mysql -u task_manage -p
```

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

### Using Frappe Console

```sh
cd ~/frappe-bench
bench --site task_manager.local console
```

Inside the Python shell:

```python
frappe.get_doc({
    "doctype": "DocType",
    "name": "Task",
    "module": "Desk",
    "custom": 1,
    "fields": [
        {"fieldname": "title", "fieldtype": "Data", "label": "Title"},
        {"fieldname": "description", "fieldtype": "Small Text", "label": "Description"}
    ]
}).insert()
```

### Using Web UI

1. Go to **Frappe Desk** (`http://task_manager.local:8000/desk`).
2. Navigate to **Developer > DocType**.
3. Click **New**, enter **Task**, and configure the fields.
4. Save and enable **Custom?** if needed.

## Develop RESTful APIs

- **Create:** `POST /api/resource/Task`
- **Read:** `GET /api/resource/Task/<task-name>`
- **Update:** `PUT /api/resource/Task/<task-name>`
- **Delete:** `DELETE /api/resource/Task/<task-name>`

### cURL Examples

#### Create a Task

```bash
curl -X POST http://task_manager.local:8000/api/resource/Task\
 -H "Authorization: token <api-key:api-secret>" \ 
 -H "Content-Type: application/json" \
 -d '{
    "title": "Complete API Development",
    "description": "Develop RESTful APIs for Task DocType",
    "status": "Open",
    "assigned_to": "user@example.com",
    "due_date": "2023-12-31"
  }'
```

### Authentication

Generate API token:

```bash
 make-token from dashboard
```

Include in headers:

```json
{
  "Authorization": "token <api-token>"
}
```

## Testing the Setup

To ensure everything is running correctly, follow these steps:

1. **Check Bench Status:**
   ```bash
   bench start
   ```
2. **Verify Site is Running:**
   - Open `http://task_manager.local:8000` in a browser.
3. **Test API Response:**
   ```bash
   curl -X GET http://task_manager.local:8000/api/method/ping
   ```
4. **Check MariaDB Connection:**
   ```bash
   bench --site task_manager.local mysql
   ```
5. **Run Unit Tests:**
   ```bash
   bench --site task_manager.local run-tests
   ```

## Data Management (20 Points)

- Design a **normalized** database schema for a project management system using DocTypes.
- Define relationships:
  - **Project:** title, description, start_date, end_date.
  - **Task:** title, description, status, assigned_to, due_date (linked to Project).
  - **User:** Use default Frappe User DocType.
- Implement in **DocType Builder**.

## Performance Optimization (20 Points)

- **Use Pagination:**
  ```bash
  GET /api/resource/Task?limit_start=0&limit_page_length=20
  ```
- **Selective Fields to Reduce Payload:**
  ```bash
  GET /api/resource/Task?fields=["title", "status"]
  ```
- **Enable Redis Caching:**
  ```bash
  sudo apt install redis-server
  bench --site task_manager.local set-config redis_cache enabled
  bench --site task_manager.local set-config redis_queue enabled
  ```

## Debugging and Problem Solving (10 Points)

- **Analyze Frappe logs:**
  ```bash
  bench --site task_manager.local --debug
  ```

