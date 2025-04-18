# 🚀 Deploying a Web App on AWS EC2 or Azure VM with SSL & Custom Domain

---

## 🖥️ Step 1: Launch a Virtual Machine (EC2 or Azure)

### 🔸 For AWS EC2:

1. Go to **AWS EC2 Dashboard**.
2. Click on **Launch Instance**.
3. **Name & Tags:** Give your instance a name.
4. **Application & OS Image:** Leave the default settings (Amazon Linux).
5. **Instance Type:** Leave as default (e.g., `t2.micro`).
6. **Key Pair:**
   - Create a new key pair.
   - Name it and download the `.pem` file.
   - ⚠️ **Important:** Keep this `.pem` file safe. If you lose it, you can't access your VM!
7. Leave other options as default and click **Launch Instance**.

✅ Your EC2 instance is now created.

---

## ⚙️ Step 2: Connect to Your Virtual Machine

### 🪟 On Windows:

1. Create a `.bat` file (`my-app.bat`) and open it in a text editor.
2. Add the following command (replace paths and IPs):

```bash
ssh -i "C:\Users\YourName\path-to-your-key.pem" ec2-user@YOUR_VIRTUAL_MACHINE_PUBLIC_IP
```

✅ For **AWS**, the username is `ec2-user`  
✅ For **Azure**, the username is usually `azureuser`

3. Save and double-click the `.bat` file to connect.

---

## 📦 Step 3: Install Required Software (Git, Node.js, Nginx)

### 🔸 For AWS (Amazon Linux):

```bash
sudo yum update -y

# Install Git
sudo yum install git -y

# Install Node.js (via NodeSource)
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install -y nodejs

# Install Nginx
sudo amazon-linux-extras install nginx1 -y
```

### 🔸 For Azure (Ubuntu/Debian):

```bash
sudo apt update -y

# Install Git
sudo apt install git -y

# Install Node.js (via NodeSource)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Install Nginx
sudo apt install nginx -y
```

---

## 📁 Step 4: Deploy Your App

1. Clone your project from GitHub:

```bash
git clone https://github.com/your-username/your-repo.git
cd your-repo
```

2. Install project dependencies:

```bash
npm install
```

3. Start your app (use **pm2** to keep it alive):

```bash
sudo npm install -g pm2
pm2 start index.js --name "your-app"
pm2 startup
pm2 save
```

✅ Your app is now running on a port (e.g., `http://localhost:3000`)  
You can test it via browser:

```
http://YOUR_VIRTUAL_MACHINE_PUBLIC_IP:3000
```

---

## 🌐 Step 5: Connect Domain to Your App

### 🔧 Setup DNS

1. Buy a domain from GoDaddy, Hostinger, Namecheap, etc.
2. Go to DNS settings in your domain provider’s dashboard.
3. Create an **A Record**:
   - **Host:** `@`
   - **Value:** `YOUR_VIRTUAL_MACHINE_PUBLIC_IP`
   - **TTL:** `Default`

✅ It may take up to 30 minutes for DNS to propagate.

---

## 🌐 Step 6: Configure Nginx for Your Domain

1. Open the Nginx config file:

```bash
sudo nano /etc/nginx/nginx.conf
```

2. Replace contents with:

```nginx
events {}

http {
    server {
        listen 80;
        server_name yourdomain.com www.yourdomain.com;

        location / {
            proxy_pass http://localhost:3000;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }

        access_log /var/log/nginx/yourdomain.access.log;
        error_log /var/log/nginx/yourdomain.error.log;
    }
}
```

3. Test and restart Nginx:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

✅ Now you can access your website via `http://yourdomain.com`

---

## 🔒 Step 7: Enable HTTPS (SSL Certificate)

1. Install **Certbot**:

### For AWS (Amazon Linux):

```bash
sudo yum install certbot python3-certbot-nginx -y
```

### For Azure (Ubuntu):

```bash
sudo apt install certbot python3-certbot-nginx -y
```

2. Stop Nginx temporarily:

```bash
sudo systemctl stop nginx
```

3. Generate SSL Certificate:

```bash
sudo certbot certonly --standalone -d yourdomain.com -d www.yourdomain.com
```

4. After success, update Nginx config again:

```bash
sudo nano /etc/nginx/nginx.conf
```

Replace with:

```nginx
events {}

http {
    server {
        listen 443 ssl http2;
        server_name yourdomain.com www.yourdomain.com;

        ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
        ssl_protocols TLSv1.2 TLSv1.3;

        location / {
            proxy_pass http://localhost:3000;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }

        access_log /var/log/nginx/yourdomain.access.log;
        error_log /var/log/nginx/yourdomain.error.log;
    }

    server {
        listen 80;
        server_name yourdomain.com www.yourdomain.com;
        return 301 https://$host$request_uri;
    }
}
```

5. Start and check Nginx:

```bash
sudo systemctl start nginx
sudo systemctl status nginx
```

---

## ✅ Final Result

🎉 **Your app is now live on a custom domain with HTTPS!**

Access:

```
https://yourdomain.com
```

---

## 🔁 Bonus: Renew SSL Automatically (Optional)

Certbot takes care of this via cron job, but you can test renewal manually:

```bash
sudo certbot renew --dry-run
```
