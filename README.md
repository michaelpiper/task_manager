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

### **Solution 1: Use the Frappe Console**

Run the following commands inside your **Bench directory** (`frappe-bench`):
```sh
cd ~/frappe-bench
bench --site task_manager.local console
```
Then, inside the Python shell, run:
```python
frappe.get_doc({
    "doctype": "DocType",
    "name": "Task",
    "module": "YourModuleName",
    "custom": 1,
    "fields": [
        {"fieldname": "title", "fieldtype": "Data", "label": "Title"},
        {"fieldname": "description", "fieldtype": "Small Text", "label": "Description"}
    ]
}).insert()
```
Replace **`YourModuleName`** with the actual module name.

### **Solution 2: Create via Web UI**
1. Go to **Frappe Desk** (`http://task_manager.local/desk`).
2. Navigate to **Developer > DocType**.
3. Click **New**, enter **Task** as the name, and configure the fields.
4. Save and enable **Custom?** if needed.

### **Solution 3: Using Bench Reload**
If you modified an existing DocType, reload it:
```sh
bench --site task_manager.local reload-doctype Task
```

## Default Administrator Username

The default Administrator username is:

```sh
Administrator
```

To access your Frappe site at [http://task_manager.local/](http://task_manager.local/), you need to map the domain `task_manager.local` to your local machine's IP address (`127.0.0.1` or `localhost`).

### Step 1: Edit the Hosts File

Open the terminal on your Linux machine.

Open the hosts file in a text editor with root privileges:

```bash
sudo nano /etc/hosts
```

Add the following line at the end of the file:

```
127.0.0.1   task_manager.local
```

Save the file and exit the editor:

- In nano, press **CTRL + X**, then **Y**, and **Enter**.

