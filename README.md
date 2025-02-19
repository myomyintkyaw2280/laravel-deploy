# Laravel Project Deployment

Since you already have a Laravel project, here’s a **step-by-step Laravel deployment guide** covering both **shared hosting (cPanel) and VPS/cloud servers (AWS, DigitalOcean, etc.)**.  

---

## **1️⃣ Prepare Your Laravel Application**
Before deployment, make sure your Laravel project is production-ready:

### ✅ **Update `.env` File for Production**
- Set the correct environment variables in your `.env` file:
  
  ```env
  APP_ENV=production
  APP_DEBUG=false
  APP_URL=https://yourdomain.com
  ```

- Set your **database connection**:
  
  ```env
  DB_CONNECTION=mysql
  DB_HOST=your_database_host
  DB_PORT=3306
  DB_DATABASE=your_db_name
  DB_USERNAME=your_db_user
  DB_PASSWORD=your_db_password
  ```

### ✅ **Install Dependencies**
Run the following command to install all dependencies:
```sh
composer install --no-dev --optimize-autoloader
```
**Explanation:**
- `--no-dev`: Excludes development dependencies.
- `--optimize-autoloader`: Optimizes class loading.

---

## **2️⃣ Deploy Laravel to the Server**
### **Option 1: Deploying on Shared Hosting (cPanel)**
If you are using **cPanel (e.g., GreenGeeks, Namecheap, GoDaddy, etc.)**, follow these steps:

#### **Step 1: Upload Laravel Files**
1. **Zip your Laravel project** (except `vendor` and `node_modules`).
2. Upload the `.zip` file to the `public_html` directory using the **cPanel File Manager**.
3. Extract the zip file.

#### **Step 2: Move Laravel Files**
1. **Move all files (except the `public` folder) to a subdirectory** (e.g., `yourproject/`).
2. Inside `public_html`, **keep only the `public` folder**.
3. Rename `public` to something like `yourdomain`.

#### **Step 3: Update `index.php`**
Edit `public_html/yourdomain/index.php` and update:
```php
require __DIR__.'/../yourproject/bootstrap/autoload.php';
$app = require_once __DIR__.'/../yourproject/bootstrap/app.php';
```

#### **Step 4: Set Up `.env` File**
1. Copy `.env.example` to `.env` in the `yourproject/` directory.
2. Update **database credentials** (as shown in Step 1).

#### **Step 5: Set Permissions**
Run the following command using the **cPanel Terminal** or **SSH**:
```sh
chmod -R 775 storage bootstrap/cache
```

#### **Step 6: Set Up Cron Jobs (Optional)**
If you are using **queues or scheduled tasks**, set up a cron job in cPanel:
```sh
* * * * * php /home/yourcpaneluser/yourproject/artisan schedule:run >> /dev/null 2>&1
```

---

### **Option 2: Deploying on VPS (AWS, DigitalOcean, Linode, etc.)**
If you have an **Ubuntu server**, follow these steps:

#### **Step 1: Connect to Server via SSH**
Use the terminal to connect:
```sh
ssh youruser@yourserver_ip
```

#### **Step 2: Install Dependencies**
Install PHP, MySQL, and other dependencies:
```sh
sudo apt update && sudo apt install -y php-cli php-mbstring php-xml php-bcmath php-curl unzip curl git
```

#### **Step 3: Clone Your Laravel Project**
Upload your project via **Git**:
```sh
cd /var/www
git clone https://github.com/yourusername/yourproject.git
cd yourproject
```

#### **Step 4: Set Up `.env`**
```sh
cp .env.example .env
nano .env  # Edit database and APP_URL settings
```

#### **Step 5: Install Composer Dependencies**
```sh
composer install --no-dev --optimize-autoloader
```

#### **Step 6: Set Permissions**
```sh
sudo chown -R www-data:www-data /var/www/yourproject
sudo chmod -R 775 storage bootstrap/cache
```

#### **Step 7: Set Up Nginx/Apache**
For **Nginx**, configure a new site:
```sh
sudo nano /etc/nginx/sites-available/yourproject
```
Add this configuration:
```nginx
server {
    listen 80;
    server_name yourdomain.com;
    root /var/www/yourproject/public;
    index index.php index.html;
    
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```
Activate and restart Nginx:
```sh
sudo ln -s /etc/nginx/sites-available/yourproject /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

#### **Step 8: Migrate Database**
```sh
php artisan migrate --force
```

#### **Step 9: Set Up Supervisor for Queues (Optional)**
```sh
sudo nano /etc/supervisor/conf.d/laravel-worker.conf
```
Add:
```
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/yourproject/artisan queue:work --tries=3
autostart=true
autorestart=true
numprocs=1
user=www-data
redirect_stderr=true
stdout_logfile=/var/www/yourproject/storage/logs/worker.log
```
Reload Supervisor:
```sh
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start laravel-worker:*
```

---

## **3️⃣ Post-Deployment Steps**
Once deployed, run these final commands:

### ✅ **Clear and Cache Configuration**
```sh
php artisan config:clear
php artisan cache:clear
php artisan route:clear
php artisan view:clear
php artisan optimize
```

### ✅ **Set Up SSL (HTTPS)**
If using **cPanel**, enable **AutoSSL**.  
If using **VPS**, install **Certbot**:
```sh
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com
```

### ✅ **Restart Server**
```sh
sudo systemctl restart nginx
sudo systemctl restart php8.1-fpm
```

---

## **4️⃣ Testing the Deployment**
Now, open **https://yourdomain.com** in a browser and check if everything is working!

---

### **🎯 Summary**
| Step | Shared Hosting (cPanel) | VPS (AWS, DigitalOcean) |
|------|----------------|-------------------|
| Upload Files | File Manager (Zip) | Git Clone / SCP |
| Set Up `.env` | Edit in File Manager | Use SSH & Nano |
| Install Dependencies | cPanel Terminal | `composer install --no-dev` |
| Set Permissions | `chmod -R 775 storage` | `chown -R www-data:www-data` |
| Web Server | Apache (cPanel) | Nginx / Apache |
| Database | MySQL (cPanel) | `php artisan migrate --force` |
| Queue Processing | Cron Jobs | Supervisor |
| SSL | AutoSSL | Certbot |

---
