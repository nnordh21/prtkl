# 🌍 PRTKL
> **Automating the Subdivision Control Check:** An Open-Source GIS and LLM Pipeline for Cadastral Case Preparation

This repository contains the source code for the **FOSS4G academic track submission**. 

> [!NOTE]
> This version is an alternative, general-purpose application distinct from the one specified in the official paper. While the original paper centers heavily on Danish legislation and chartered surveyor workflows, this FOSS4G release allows you to plug in **any Web Feature Service (WFS) link**. It harmonizes and extracts data, attributes, and geometry on the fly, feeding a structured PostGIS analysis straight into a Large Language Model (LLM). 

Because this version is highly dynamic compared to the rigid original, it requires a bit of configuration. Time to put on your hacker hat! 🤠

---

## 🛠️ Prerequisites

PRTKL is the core web-service engine. To orchestrate the entire pipeline, you will need to manually hook up the following dependencies:

* **Node.js** (v18+ recommended) – To power the web service.
* **Supabase** – Our all-in-one backend (Database, Auth, and Vector extensions).
* **Ollama** – For running your LLM locally.
* *(Optional)* **Coolify** – For easy self-hosted deployment.
* *(Optional)* **Cloudflare Tunnel** – Crucial if your web service is hosted online but needs a secure bridge to your local Ollama instance (covered in the Coolify section).

---

## 🚀 Installation & Setup

### 1. Clone & Install Dependencies
First, grab the code and install the required Node packages:

```bash
git clone https://github.com/nnordh21/prtkl
cd prtkl
npm install
```

### 2. Environment Configuration
Copy the template environment file and fill in your specific service credentials:

```bash
cp .env.example .env
```

Open `.env` and configure your Supabase keys, database connection strings, and LLM API endpoints.

### 3. Supabase Setup
Supabase provides the heavy lifting for our spatial data layer. Follow these steps to prepare your instance:

1. Create a new project in the Supabase Dashboard.
2. Navigate to the **SQL Editor** in your Supabase dashboard.
3. Enable the **PostGIS** extension by running the following query:
   ```sql
   create extension postgis;
   ```
4. Create the necessary tables for tracking your WFS layers and LLM prompts (refer to the schema.sql file in this repository if provided, or let the application auto-migrate on first boot).

### 4. Local LLM Setup (Ollama)
To handle the automated legal and spatial interpretations:

1. Download and install Ollama.
2. Pull your LLM of choice (e.g., Llama 3 or Mistral):
   ```bash
   ollama run llama3
   ```

---

## 💻 Running the Application

### Local Development Mode
To spin up the local development server with hot-reloading:

```bash
npm run dev
```

The service will typically be accessible at `http://localhost:3000`.

### Production Build
To build and run the optimized application for deployment:

```bash
npm run build
npm run start
```

---

## ☁️ Advanced: Self-Hosting with Coolify (Recommended)

If you want to take PRTKL out of local development and host it reliably without breaking the bank, we recommend **Coolify**. It is an open-soruce, self-hostable Platform-as-a-Service (PaaS), allowing you to manage your web service and databases in one unified dashboard.

Here is the step-by-step workflow for setting up the entire architecture on a Virtual Private Server (VPS).

### 1. Spin Up a Coolify Instance
First, you need a fresh VPS running Ubuntu (e.g., Hetzner, DigitalOcean, or AWS EC2. We recommend Hostinger, as they have an easy dashboard and a one-click package installation of Coolify). If your hosting service does not provide a one-click installation, SSH into your server and run the official Coolify installation script:

```bash
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | bash
```
Once completed, access your new Coolify dashboard via the IP provided in your terminal and complete the initial onboarding.

### 2. Deploy (your own) Supabase instance
Instead of relying on Supabase's managed cloud, you can host it directly alongside your app, inside Coolify:

1. In the Coolify dashboard, navigate to **Services** -> **Add New Service**.
2. Search for and select the **Supabase** one-click template.
3. Configure your desired passwords and deploy.
4. *Crucial:* Once deployed, grab the API keys and the database connection URL. Because both Supabase and your web app will live on the same Coolify instance, they can communicate ultra-fast over the internal Docker network.

### 3. Deploy the PRTKL Web Service
Now, let's deploy the Node.js application:

1. In Coolify, go to **Projects** -> **Add New Resource** -> **Git Repository**.
2. Connect your GitHub account and select your `prtkl` repository.
3. Set the Build Pack to **Nixpacks** (it will automatically detect the Node.js environment based on your `package.json`).
4. **Environment Variables:** Before clicking deploy, navigate to the Environment Variables tab. You must define your `.env` variables here. Map the Supabase URL and Auth keys you generated in Step 2.
5. Hit **Deploy**.

### 4. Tying it Together: The Cloudflare Tunnel (Local LLM)
Here is where the magic happens. Your PRTKL web app and Supabase database are now living in the cloud on your VPS. However, to save on expensive cloud GPU costs, you are running your LLM (Ollama) on your local desktop hardware. 

To allow your cloud-hosted Coolify app to securely talk to your local hardware *without* opening dangerous ports on your home router, we use Cloudflare.

1. **On your local home machine** (where Ollama is running), start the Cloudflare tunnel:
   ```bash
   cloudflared tunnel --url http://localhost:11434
   ```
2. Copy the generated `.trycloudflare.com` URL from the terminal output.
3. **Back in Coolify**, go to your PRTKL web service's Environment Variables tab.
4. Add or update the variable for your LLM endpoint to point to the secure tunnel:
   `LLM_ENDPOINT=https://your-generated-url.trycloudflare.com`
5. Save and restart the PRTKL web service.
