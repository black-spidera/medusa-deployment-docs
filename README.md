# Medusa Project - Installation and Setup Guide

## Clone the Repository

To get started, clone the repository:

```bash
git clone https://github.com/YOUR_USERNAME/medusa.git
```

## Project Setup

### 1. Navigate to the Project Directory

```bash
cd medusa
```

### 2. Configure Yarn

Create a `.yarnrc.yml` file with the following content to configure Yarn:

```bash
echo 'nodeLinker: node-modules' > .yarnrc.yml
```

### 3. Install Dependencies

Install the necessary dependencies and run the setup:

```bash
yarn install && yarn setup
```

### 4. Set Up Environment Variables

Update the environment variables in the `apps/backend/.env` file:

```env
DATABASE_URL=postgres://USERNAME:PASSWORD@localhost:5432/DB_NAME
DB_NAME=medusa-backend
```

Replace `USERNAME`, `PASSWORD`, and `DB_NAME` with your PostgreSQL credentials.

### 5. Run Setup Fix (if needed)

In the `package.json` file, add a `setup-fix` script:

```json
"scripts": {
    "setup-fix": "yarn build-packages && yarn setup-db && yarn seed && yarn setup-user"
}
```

Then, to resolve any setup issues, run:

```bash
yarn setup-fix
``` 

This script will build the packages, set up the database, seed initial data, and configure a default user.

### 6. Start the Development Server

```bash
yarn dev
```

---

## Configure the Publishable Key

1. Open the [Medusa Admin Publishable Key](http://localhost:9000/app/settings/publishable-api-keys) page.
   
   - **Credentials**:
     - **Email**: `admin@example.com`
     - **Password**: `supersecret`

2. Copy the token key for the "Webshop" channel.

3. Open the `apps/storefront/.env` file and add the token:

   ```env
   NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=<your_copied_token>
   ```

---

## Access Medusa Applications

- [Medusa Admin](http://localhost:9000/app)
- [Medusa Storefront](http://localhost:8000)

---

## PostgreSQL Configuration on a Live Server

If configuring PostgreSQL on a live server, update the following settings:

1. Open `postgresql.conf`:

   ```bash
   sudo nano /etc/postgresql/{version}/main/postgresql.conf
   ```

   Set `listen_addresses` to enable listening on all addresses (ensure to use single quotes around the asterisk):

   ```plaintext
   listen_addresses = '*'
   ```

2. Edit `pg_hba.conf` to allow connections:

   ```bash
   sudo nano /etc/postgresql/{version}/main/pg_hba.conf
   ```

   Add these lines:

   ```plaintext
   host    all             all             {server_ip}/32           md5
   host    all             all             0.0.0.0/0                md5
   ```

   Replace `{server_ip}` with your server’s IP.

3. Restart PostgreSQL:

   ```bash
   sudo systemctl restart postgresql
   ```

---

## Node, Nginx, and PM2 Setup for Live Deployment

### 1. Install Node.js and npm

```bash
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt install nodejs
node --version
```

### 2. Clone the Project from GitHub

```bash
git clone https://github.com/YOUR_USERNAME/medusa.git
cd medusa
```

### 3. Install Project Dependencies and Test

```bash
npm install
npm start
```

Stop the app using `ctrl + C`.

### 4. Set Up PM2 for Process Management

```bash
sudo npm install pm2 -g
pm2 start app (or your entry file)
```

**Additional PM2 Commands:**
- `pm2 show app`
- `pm2 status`
- `pm2 restart app`
- `pm2 stop app`
- `pm2 logs`
- `pm2 flush`

To ensure the app starts on reboot:

```bash
pm2 startup ubuntu
```

### 5. Set Up the Firewall with UFW

```bash
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
```

### 6. Install and Configure Nginx

1. Install Nginx:

   ```bash
   sudo apt install nginx
   ```

2. Configure the Nginx `default` file:

   ```bash
   sudo nano /etc/nginx/sites-available/default
   ```

   Add the following server block configuration:

   ```nginx
   server {
       listen 80;
       server_name yourdomain.com www.yourdomain.com;

       # Admin Panel
       location /app {
           proxy_pass http://localhost:9000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }

       # Storefront
       location / {
           proxy_pass http://localhost:8000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```

3. Verify the configuration:

   ```bash
   sudo nginx -t
   ```

4. Restart Nginx:

   ```bash
   sudo service nginx restart
   ```

Access your app via the domain without a port (port 80).

### 7. Set Up Environment Variables

Update `medusa-config.ts`:

```javascript
admin: {
    disable: process.env.DISABLE_MEDUSA_ADMIN === "true",
    backendUrl: process.env.MEDUSA_BACKEND_URL || "http://{server_ip}:9000",
    path: process.env.MEDUSA_ADMIN_PATH || "/app",
}
```

**Backend `.env`**

```env
STORE_CORS=http://{server_ip}:8000,https://docs.medusajs.com
ADMIN_CORS=http://{server_ip}:9000,https://docs.medusajs.com,http://{your_domain},https://{your_domain}
AUTH_CORS=http://{server_ip}:9000,https://docs.medusajs.com,http://{your_domain},https://{your_domain}
REDIS_URL=redis://{server_ip}:6379
JWT_SECRET=supersecret
COOKIE_SECRET=supersecret
DATABASE_URL=postgres://postgres:password@{server_ip}:5432/medusa-backend
DB_NAME=medusa-backend
```

**Frontend `.env`**

```env
NEXT_PUBLIC_MEDUSA_BACKEND_URL=http://{server_ip}:9000
NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=<your_copied_token>
NEXT_PUBLIC_BASE_URL=http://{server_ip}:8000
NEXT_PUBLIC_DEFAULT_REGION=us
REVALIDATE_SECRET=supersecret
```

--- 

Your project is now set up and accessible at the following URLs:

- **Storefront:** `http://{your_domain}` or `http://{server_ip}:8000`
- **Admin Panel:** `http://{your_domain}/app` or `http://{server_ip}:9000/app`
- **Admin Login:** `http://{server_ip}:9000/app/login` 
