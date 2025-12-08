# Setting Up Focalboard with Supabase

This guide will help you configure Focalboard to use Supabase as your database backend instead of SQLite.

## Why Supabase?

Supabase provides a managed PostgreSQL database with:
- Automatic backups
- Scalability
- Real-time capabilities
- Free tier for small projects
- Easy deployment and management

## Prerequisites

1. A Supabase account (sign up at [supabase.com](https://supabase.com))
2. A Supabase project created
3. Focalboard source code

## Step 1: Get Your Supabase Database Connection String

1. Log in to your [Supabase Dashboard](https://app.supabase.com)
2. Select your project (or create a new one)
3. Click on the **Settings** icon (gear icon) in the left sidebar
4. Navigate to **Database** section
5. Scroll down to the **Connection string** section
6. Copy the **URI** connection string
   - It should look like: `postgres://postgres:[YOUR-PASSWORD]@db.xxxxxxxxxxxx.supabase.co:5432/postgres`
7. **Important**: Replace `[YOUR-PASSWORD]` with your actual database password
   - If you forgot your password, you can reset it in the Database Settings

## Step 2: Configure Focalboard

You have two options to configure Focalboard to use Supabase:

### Option A: Using Environment Variables (Recommended)

1. Copy the `.env.example` file to `.env`:
   ```bash
   cp .env.example .env
   ```

2. Edit `.env` and set your Supabase connection details:
   ```bash
   FOCALBOARD_DBTYPE=postgres
   FOCALBOARD_DBCONFIG=postgres://postgres:your-actual-password@db.xxxxxxxxxxxx.supabase.co:5432/postgres?sslmode=require
   ```

3. Make sure to add `?sslmode=require` at the end of the connection string for secure connections

### Option B: Using Configuration File

1. Edit `server-config.json` (or `config.json`):
   ```json
   {
     "dbtype": "postgres",
     "dbconfig": "postgres://postgres:your-actual-password@db.xxxxxxxxxxxx.supabase.co:5432/postgres?sslmode=require",
     ...
   }
   ```

> **Note**: Environment variables (Option A) will override configuration file settings, making it easier to manage different environments (development, production, etc.)

## Step 3: Run Focalboard

1. Start the Focalboard server:
   ```bash
   cd server
   go run main/main.go
   ```

2. Check the logs for successful database connection:
   - You should see: `connectDatabase dbType=postgres`
   - Database migrations should run automatically
   - No errors related to database connection

## Step 4: Verify Setup

1. Open Focalboard in your browser: `http://localhost:8000`
2. Create a test board
3. Add some cards
4. Go to your Supabase Dashboard -> Table Editor
5. You should see Focalboard tables created (like `focalboard_blocks`, `focalboard_boards`, etc.)

## Troubleshooting

### Connection Issues

If you see connection errors:

1. **Check SSL Mode**: Supabase requires SSL. Ensure your connection string ends with `?sslmode=require`
2. **Verify Password**: Make sure you're using the correct database password
3. **Check Firewall**: Ensure your network allows outbound connections to Supabase (port 5432)
4. **Connection Pooling**: For production, consider using Supabase's connection pooling:
   - Use the "Connection pooling" string instead of "Direct connection" in Supabase settings
   - Format: `postgres://postgres.xxxxxxxxxxxx:your-password@aws-0-region.pooler.supabase.com:6543/postgres`

### Migration Issues

If database migrations fail:

1. Check the Supabase logs in the Dashboard -> Logs
2. Ensure your database user has proper permissions
3. Check if tables already exist from a previous installation

### Performance Tips

1. **Use Connection Pooling**: For production deployments, use Supabase's connection pooling endpoint (port 6543)
2. **Indexes**: Focalboard creates necessary indexes automatically during migration
3. **Monitor**: Use Supabase's built-in monitoring to track database performance

## Migrating from SQLite

If you have an existing SQLite database you want to migrate:

1. **Export from SQLite**:
   ```bash
   sqlite3 focalboard.db .dump > focalboard_dump.sql
   ```

2. **Convert to PostgreSQL**:
   - You'll need to modify the SQL dump to be PostgreSQL-compatible
   - Tools like `pgloader` can help automate this process

3. **Import to Supabase**:
   - Use the Supabase SQL Editor to run the converted SQL
   - Or use `psql` command-line tool with your connection string

> **Warning**: Migration can be complex. Test thoroughly before migrating production data.

## Security Best Practices

1. **Never commit** `.env` file to version control
2. **Use strong passwords** for your database
3. **Enable Row Level Security (RLS)** in Supabase for additional protection
4. **Rotate credentials** regularly
5. **Use environment-specific** connection strings (dev, staging, production)

## Additional Resources

- [Supabase Documentation](https://supabase.com/docs)
- [PostgreSQL Connection Strings](https://www.postgresql.org/docs/current/libpq-connect.html#LIBPQ-CONNSTRING)
- [Focalboard Documentation](https://www.focalboard.com/docs/)

## Support

If you encounter issues:
1. Check Focalboard logs for specific error messages
2. Verify Supabase project status in the dashboard
3. Consult the Focalboard community or Supabase support
