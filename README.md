# 🏦 Modern Bank India — Full Stack Banking System

A full-stack banking application built as an 8th Semester Major Project, simulating core banking functionalities for both customers and bankers.

---

## 🚀 Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Vue.js 3 + Vite + Tailwind CSS |
| Backend | Node.js + Express.js |
| Database | PostgreSQL |
| Authentication | JWT + bcrypt |
| Email | Nodemailer + Gmail SMTP |
| AI Chatbot | OpenRouter API (Deepseek) |
| State Management | Pinia |
| Caching | Redis (optional) |

---

## ✨ Features

### Customer Portal
- 🔐 Register with OTP email verification
- 💰 View account balance and transaction history
- 💸 Transfer money to other customers
- 📊 View deposits (Fixed, Recurring, Savings)
- 💳 Virtual card generation
- 📈 CIBIL score tracking
- 🤖 AI-powered banking chatbot
- 🎙️ Voice assistant support
- 📧 Email notifications for transactions

### Banker Portal
- 👥 View and manage all customers
- 💵 Make deposits to customer accounts
- 🔍 Check account details and transaction history
- 🚩 Flag or suspend suspicious accounts
- 📋 Update customer KYC documents
- 📊 Generate transaction and customer reports (CSV/JSON)
- 📈 Transaction analysis dashboard
- 🔎 Advanced transaction filtering

---

## 🛠️ Prerequisites

- Node.js v18+
- PostgreSQL 16+
- Git

---

## ⚙️ Installation & Setup

### 1. Clone the repository
```bash
git clone https://github.com/YOUR_USERNAME/Banking-system.git
cd Banking-system
```

### 2. Setup PostgreSQL
- Install PostgreSQL and create a database:
```sql
CREATE DATABASE banking_system;
```

### 3. Backend Setup
```bash
cd backend
npm install
```

Create a `.env` file in the `backend` folder:
```env
PORT=5000

DB_HOST=localhost
DB_PORT=5432
DB_NAME=banking_system
DB_USER=postgres
DB_PASSWORD=your_postgres_password

JWT_SECRET=your_jwt_secret_key
JWT_EXPIRES_IN=7d

EMAIL_SERVICE=gmail
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_SECURE=false
EMAIL_USER=your_gmail@gmail.com
EMAIL_PASS=your_gmail_app_password
EMAIL_PASSWORD=your_gmail_app_password
EMAIL_FROM=your_gmail@gmail.com

OPENROUTER_API_KEY=your_openrouter_api_key
```

Run database migrations:
```bash
node run_migrations.js
```

Start the backend:
```bash
npm start
```

### 4. Frontend Setup
```bash
cd ../frontend
npm install
```

Create a `.env` file in the `frontend` folder:
```env
VITE_API_URL=http://localhost:5000
```

Start the frontend:
```bash
npm run dev
```

### 5. Access the Application
| Portal | URL |
|--------|-----|
| Frontend | http://localhost:5173 |
| Backend API | http://localhost:5000 |

---

## 🗄️ Database Setup

After running migrations, run these SQL queries in pgAdmin to add missing columns:

```sql
-- Add columns to customers table
ALTER TABLE customers ADD COLUMN IF NOT EXISTS name VARCHAR(255);
ALTER TABLE customers ADD COLUMN IF NOT EXISTS email VARCHAR(255) UNIQUE;
ALTER TABLE customers ADD COLUMN IF NOT EXISTS password VARCHAR(255);
ALTER TABLE customers ADD COLUMN IF NOT EXISTS address TEXT;
ALTER TABLE customers ADD COLUMN IF NOT EXISTS phone VARCHAR(20);
ALTER TABLE customers ADD COLUMN IF NOT EXISTS account_type VARCHAR(50) DEFAULT 'savings';
ALTER TABLE customers ADD COLUMN IF NOT EXISTS account_number VARCHAR(20) UNIQUE;
ALTER TABLE customers ADD COLUMN IF NOT EXISTS balance DECIMAL(15,2) DEFAULT 0.00;
ALTER TABLE customers ADD COLUMN IF NOT EXISTS status VARCHAR(20) DEFAULT 'pending';
ALTER TABLE customers ADD COLUMN IF NOT EXISTS cibil_score INTEGER DEFAULT 750;
ALTER TABLE customers ADD COLUMN IF NOT EXISTS created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP;
ALTER TABLE customers ADD COLUMN IF NOT EXISTS updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP;

-- Create transactions table
CREATE TABLE IF NOT EXISTS transactions (
  id SERIAL PRIMARY KEY,
  customer_id INTEGER REFERENCES customers(id) ON DELETE CASCADE,
  sender_id INTEGER REFERENCES customers(id),
  receiver_id INTEGER REFERENCES customers(id),
  type VARCHAR(50) NOT NULL,
  amount DECIMAL(15,2) NOT NULL,
  balance DECIMAL(15,2) DEFAULT 0.00,
  description TEXT,
  status VARCHAR(20) DEFAULT 'pending',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create deposits table
CREATE TABLE IF NOT EXISTS deposits (
  id SERIAL PRIMARY KEY,
  customer_id INTEGER REFERENCES customers(id) ON DELETE CASCADE,
  transaction_id INTEGER REFERENCES transactions(id),
  amount DECIMAL(15,2) NOT NULL,
  type VARCHAR(50) DEFAULT 'fixed',
  duration_months INTEGER DEFAULT 12,
  interest_rate DECIMAL(5,2) DEFAULT 7.5,
  status VARCHAR(20) DEFAULT 'active',
  start_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  maturity_date TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create bankers table
CREATE TABLE IF NOT EXISTS bankers (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  password VARCHAR(255) NOT NULL,
  employee_id VARCHAR(50) UNIQUE,
  phone VARCHAR(20),
  role VARCHAR(50) DEFAULT 'banker',
  status VARCHAR(20) DEFAULT 'active',
  token_version INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Create default banker account
```bash
# Run in backend folder to generate password hash
node -e "const bcrypt = require('bcrypt'); bcrypt.hash('Banker@123', 10).then(h => console.log(h))"
```

Then insert in pgAdmin (replace HASH with output above):
```sql
INSERT INTO bankers (name, email, password, employee_id, phone)
VALUES ('Admin Banker', 'admin@modernbank.com', 'HASH', 'EMP001', '+919999999999');
```

**Banker Login:** `admin@modernbank.com` / `Banker@123`

---

## 📁 Project Structure

```
Banking-system/
├── backend/
│   ├── src/
│   │   ├── config/        # Database configuration
│   │   ├── controllers/   # Route controllers
│   │   ├── models/        # Database models
│   │   ├── routes/        # API routes
│   │   ├── services/      # Business logic
│   │   └── utils/         # Helper utilities
│   ├── run_migrations.js
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── components/    # Reusable Vue components
│   │   ├── pages/         # Page components
│   │   ├── stores/        # Pinia state stores
│   │   └── main.js        # App entry point
│   └── package.json
└── README.md
```

---

## 🔑 Environment Variables

> **Never commit your `.env` files to GitHub!** They are already in `.gitignore`.

---

## 👨‍💻 Developer

**Gayatri Raut, Kalyani There, Kanak Rahangdale**
8th Semester Major Project
Modern Bank India — Full Stack Banking System

---

## 📄 License

This project is built for educational purposes as part of an academic major project.