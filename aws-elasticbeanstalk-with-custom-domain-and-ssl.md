# 🚀 Deploying on AWS Elastic Beanstalk with SSL & Custom Domain

---

## 🌐 Step 1: Request an SSL Certificate

🔒 We'll use **AWS Certificate Manager (ACM)** for this.

1. Go to **AWS Certificate Manager**.
2. Click on **Request a certificate**.
3. Select **Public Certificate** and click **Next**.
4. Enter your domain (e.g., `yourdomain.com`) or subdomain (e.g., `api.yourdomain.com`).
5. Leave default settings and click **Request**.
6. You’ll now see your certificate with **Pending validation** status.
7. ACM will provide a **CNAME record** (Name & Value).
8. Go to your **DNS provider** (e.g., GoDaddy, Namecheap, Hostinger).
9. Create a new **CNAME record** using the provided Name & Value.

💡 **Note:**  
If you’re using a subdomain (`api.yourdomain.com`), make sure to remove the root domain from the "Name" when adding the CNAME in your DNS settings. Just keep the part ACM gives you (not `api.yourdomain.com._xyz...`, only `_xyz...`).

10. Return to ACM and wait for the status to change from **Pending** to **Issued**.

---

## ⚙️ Step 2: Create an Elastic Beanstalk App

1. Go to **AWS Elastic Beanstalk**.
2. Click **Create Application**.

### 🧱 Configuration Steps

#### ✅ Step 1: Application Info
- **Name** your app.
- **Platform**: Select your tech stack (Node.js, Python, Java, etc.).
- **Code Source**: Upload your ZIP file or select from an S3 bucket.
- Leave other settings as default.
- Click **Next**.

#### 🔐 Step 2: Permissions
- Select your **existing service role**.
- (If you’re the root user, this might not be necessary.)
- Click **Next**.

#### 🧰 Step 3: Network Settings
- Leave everything as default.
- Click **Next**.

#### 📦 Step 4: Environment Type & SSL
- In **Capacity**, select **Load Balanced** environment.
- In the **Listeners** section:
  - Add a new listener:
    - **Port**: `443`
    - **Protocol**: `HTTPS`
    - **SSL Certificate**: Choose the one you just created.
    - **SSL Policy**: Select `ELBSecurityPolicy-2016-08`
- Leave the rest as default.
- Click **Next**.

#### 📋 Step 5 & 6: Monitoring & Tags
- Leave default settings for both steps.
- Click **Submit**.

🕐 **Wait for your application to finish launching.**

---

## 🌍 Step 3: Connect Custom Domain

1. Go to **AWS EC2 > Load Balancers**.
2. Locate your load balancer and copy the **DNS name** (e.g., `abc-123.us-east-1.elb.amazonaws.com`).
3. Go to your **DNS provider**.
4. Add a new **CNAME** or **A Record**:
   - **Type**: Choose based on your domain setup.
   - **Value**: Paste the DNS name of your load balancer.
5. Save the record.

🕒 Wait 5–10 minutes for DNS propagation.

🎉 Boom! Your app is now live and secured with HTTPS on your custom domain.
