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

### **Solution 1: Use the Frappe Console**

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

### **Solution 2: Create via Web UI**

1. Go to **Frappe Desk** (`http://task_manager.local/desk`).
2. Navigate to **Developer > DocType**.
3. Click **New**, enter **Task** as the name, and configure the fields.
4. Save and enable **Custom?** if needed.

### **Solution 3: Using Bench Reload**

```sh
bench --site task_manager.local reload-doctype Task
```

## Develop RESTful APIs

Use Frappe's built-in API capabilities:

- **Create:** `POST /api/resource/Task`
- **Read:** `GET /api/resource/Task/<task-name>`
- **Update:** `PUT /api/resource/Task/<task-name>`
- **Delete:** `DELETE /api/resource/Task/<task-name>`

### cURL Examples

#### Create a Task

```bash
curl -X POST http://task_manager.local:8000/api/resource/Task\
 -H "Authorization: token be5920cb845e764:b98c5601b5d8517" \
 -H "Content-Type: application/json" \
 -d '{
    "title": "Complete API Development",
    "description": "Develop RESTful APIs for Task DocType",
    "status": "Open",
    "assigned_to": "user@example.com",
    "due_date": "2023-12-31"
  }'
```

### Add Authentication

Generate an API token:

```bash
bench --site task_manager.local make-token user@example.com
```

Include the token in request headers:

```json
{
  "Authorization": "token <api-token>"
}
```

## Performance Optimization

### Use Pagination for Large Datasets

```bash
GET /api/resource/Task?limit_start=0&limit_page_length=20
```

### Selective Fields to Reduce Payload

```bash
GET /api/resource/Task?fields=["title", "status"]
```

### Enable Redis Caching

```bash
sudo apt install redis-server
bench --site task_manager.local set-config redis_cache enabled
bench --site task_manager.local set-config redis_queue enabled
```

## Debugging and Problem Solving

Analyze Frappe logs:

```bash
bench --site task_manager.local --debug
```

If API responses are slow:
- Enable Redis caching
- Optimize database queries

Document issues and resolutions in the Frappe Community Forum.


