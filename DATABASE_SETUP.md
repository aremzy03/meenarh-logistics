# Database Setup Guide

## Current Issue

The backend server is running but cannot connect to MySQL because the root user requires authentication.

**Error:** `Access denied for user 'root'@'localhost' (using password: NO)`

## Solution

You need to set up the MySQL database and configure authentication. Follow these steps:

### Option 1: Set MySQL Root Password (Recommended)

1. **Set a password for MySQL root user:**
   ```bash
   sudo mysql -u root
   ```

2. **Inside MySQL prompt, run:**
   ```sql
   ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'your_password_here';
   FLUSH PRIVILEGES;
   EXIT;
   ```

3. **Update the `.env` file in the `server/` directory:**
   ```env
   DB_PASSWORD=your_password_here
   ```

4. **Restart the backend server** (it will auto-restart with nodemon)

### Option 2: Remove MySQL Root Password

1. **Access MySQL as root:**
   ```bash
   sudo mysql -u root
   ```

2. **Inside MySQL prompt, run:**
   ```sql
   ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '';
   FLUSH PRIVILEGES;
   EXIT;
   ```

3. **The `.env` file should already have an empty password:**
   ```env
   DB_PASSWORD=
   ```

### Create the Database

After fixing authentication, create the database:

```bash
mysql -u root -p  # Enter your password if you set one (or leave empty if no password)
```

Inside MySQL:
```sql
CREATE DATABASE IF NOT EXISTS meenarh_logistics;
EXIT;
```

### Run the Schema

Load the database schema from the `server/schema.sql` file:

```bash
cd server
mysql -u root -p meenarh_logistics < schema.sql
```

## Verify Connection

Test the backend endpoint:

```bash
curl http://localhost:5000/health
```

You should see: `{"status":"ok"}`

Test signup (should work after database setup):

```bash
curl -X POST http://localhost:5000/api/user/signup \
  -H "Content-Type: application/json" \
  -d '{"name":"Test User","email":"test@example.com","password":"test123"}'
```

## Current Server Status

- ✅ **Frontend:** Running on http://localhost:3000
- ✅ **Backend:** Running on http://localhost:5000
- ✅ **Connection:** Fixed (`.env.local` created with API URL)
- ❌ **Database:** Needs authentication setup (follow steps above)

## Quick Summary

The "Network Error" you were seeing is **FIXED**. The frontend can now connect to the backend.

The remaining issue is **database authentication**, which requires you to:
1. Set up MySQL password (or remove it)
2. Create the `meenarh_logistics` database
3. Load the schema from `schema.sql`

After that, the signup/login functionality will work perfectly!
