# Databricks Integration Setup Guide

## Overview

This platform now integrates with Databricks for real-time keystroke data ingestion. Data is stored both locally in CSV files (organized by user and session) and in Databricks Delta tables.

## Features

1. **Per-Session Data Folders**: Each test session creates a folder `data/{user_id}/{timestamp}/` containing CSV files
2. **User Authentication**: JWT-based authentication with unique user IDs
3. **Real-time Databricks Ingestion**: Data is sent to Databricks after each sentence completion
4. **Upsert Logic**: Running the test again replaces existing data in Databricks (not appends)

## Setup Instructions

### 1. Environment Variables

Create a `.env` file in the `backend/` directory:

```bash
# Databricks Configuration
DATABRICKS_SERVER_HOSTNAME=your-databricks-server-hostname.cloud.databricks.com
DATABRICKS_HTTP_PATH=/sql/1.0/warehouses/your-warehouse-id
DATABRICKS_ACCESS_TOKEN=your-access-token-here

# JWT Configuration
JWT_SECRET_KEY=your-secret-key-change-in-production-use-random-string

# Data Directory
DATA_DIR=data

# Users Database Path
USERS_DB_PATH=users.json
```

**Important**: Replace `your-access-token-here` with your actual Databricks access token.

### 2. Install Dependencies

```bash
cd backend
pip install -r requirements.txt
```

### 3. Databricks Tables

The tables will be created automatically on first use. The schema is:

**keystrokes table:**
- participant_id (STRING)
- test_section_id (STRING)
- sentence (STRING)
- user_input (STRING)
- keystroke_id (BIGINT)
- press_time (BIGINT)
- release_time (BIGINT)
- letter (STRING)
- keycode (INT)
- session_timestamp (STRING)
- created_at (TIMESTAMP)

**sessions table:**
- participant_id (STRING)
- test_section_id (STRING)
- created_at (TIMESTAMP)
- sentence_count (INT)
- total_keystrokes (INT)
- average_wpm (DOUBLE)
- session_timestamp (STRING)

### 4. Run the Application

**Backend:**
```bash
cd backend
uvicorn backend.main:app --reload --port 8000
```

**Frontend:**
```bash
cd frontend
npm install
npm start
```

## Usage Flow

1. **Register/Login**: Users must register or login first
2. **Consent**: Accept data collection consent
3. **Start Test**: Begin typing test
4. **Real-time Ingestion**: After each sentence, data is:
   - Saved to CSV in `data/{user_id}/{timestamp}/keystrokes.csv`
   - Sent to Databricks via real-time pipeline
5. **Test Completion**: Final session data is saved and ingested

## Data Organization

### CSV Files
```
data/
  {user_id_1}/
    20241115_143022/
      keystrokes.csv
      sessions.csv
    20241115_150145/
      keystrokes.csv
      sessions.csv
  {user_id_2}/
    20241115_160000/
      keystrokes.csv
      sessions.csv
```

### Databricks Tables

Data is stored in Delta tables with upsert logic:
- If a user runs the test again with the same `test_section_id`, the old data is deleted and replaced
- Each sentence gets its own `test_section_id`
- Session data is tracked per participant

## API Endpoints

### Authentication
- `POST /api/auth/register` - Register new user
- `POST /api/auth/login` - Login and get JWT token
- `GET /api/auth/me` - Get current user info (requires auth)

### Test Management
- `POST /api/session` - Create new test session (requires auth)
- `POST /api/test-section` - Create test section for sentence (requires auth)
- `POST /api/keystrokes` - Submit keystroke batch (requires auth)
- `POST /api/sentence-complete` - Complete sentence and trigger Databricks ingestion (requires auth)
- `POST /api/end-test` - End test and finalize data (requires auth)

## Troubleshooting

### Databricks Connection Issues

1. **Check Access Token**: Ensure your Databricks access token is correct
2. **Check Server Hostname**: Verify the server hostname matches your Databricks workspace
3. **Check HTTP Path**: Ensure the HTTP path matches your SQL warehouse endpoint
4. **Network**: Ensure your machine can reach Databricks (check firewall/proxy)

### Data Not Appearing in Databricks

1. Check backend logs for error messages
2. Verify Databricks credentials in `.env`
3. Check that tables were created (they're created automatically)
4. Verify network connectivity to Databricks

### Authentication Issues

1. Clear browser localStorage if tokens are stale
2. Check JWT_SECRET_KEY is set in `.env`
3. Verify user registration/login endpoints are working

## Notes

- CSV files are always created as backup, even if Databricks ingestion fails
- Databricks ingestion failures are logged but don't stop the test
- Each user's data is isolated by `user_id` (participant_id)
- Session timestamps ensure unique folders for each test run

