# Keystroke Capture Platform

A Python + React platform for collecting typing speed and keystroke data, designed to replicate the data collection methodology from the [136 Million Keystrokes](https://github.com/aalto-ui/136m-keystrokes) project.

## Overview

This platform captures detailed keystroke metrics including:
- Key press and release timestamps
- Key codes and character values
- Typing patterns and inter-key intervals
- Session and participant tracking

Data is initially stored in CSV files and can later be migrated to Databricks for ML model training.

## Architecture

- **Backend**: FastAPI (Python) - REST API for receiving and storing keystroke data
- **Frontend**: React - Typing test interface with keystroke event capture
- **Storage**: CSV files (daily rotation) - Ready for Databricks ingestion

## Project Structure

```
databricks_hack/
├── backend/                 # Python FastAPI backend
│   ├── main.py             # API endpoints
│   ├── models.py           # Pydantic data models
│   ├── storage/
│   │   └── csv_writer.py   # CSV persistence layer
│   └── requirements.txt    # Python dependencies
├── frontend/               # React frontend
│   ├── src/
│   │   ├── components/
│   │   │   └── TypingTest.js  # Main typing test component
│   │   └── App.js
│   └── package.json
├── data/                   # CSV data files (created at runtime)
│   ├── keystrokes_YYYYMMDD.csv
│   └── sessions_YYYYMMDD.csv
├── scripts/
│   └── preview_csv.py      # Utility to preview collected data
├── notebooks/
│   └── databricks_ingest.ipynb  # Databricks ingestion notebook
├── shared/
│   └── schema.md          # Data schema documentation
└── README.md
```

## Setup Instructions

### Prerequisites

- Python 3.8+
- Node.js 16+ and npm
- (Optional) Databricks account for future data migration

### Backend Setup

1. Navigate to the backend directory:
```bash
cd backend
```

2. Create a virtual environment (recommended):
```bash
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install dependencies:
```bash
pip install -r requirements.txt
```

4. Set environment variables (optional):
```bash
export DATA_DIR=../data  # Default is ./data
```

5. Run the backend server:
```bash
uvicorn backend.main:app --reload --port 8000
```

The API will be available at `http://localhost:8000`

API documentation: `http://localhost:8000/docs`

### Frontend Setup

1. Navigate to the frontend directory:
```bash
cd frontend
```

2. Install dependencies:
```bash
npm install
```

3. Set environment variables (optional):
```bash
export REACT_APP_API_URL=http://localhost:8000
```

4. Start the development server:
```bash
npm start
```

The frontend will be available at `http://localhost:3000`

## Usage

### Running a Typing Test

1. Start the backend server (see Backend Setup above)
2. Start the frontend server (see Frontend Setup above)
3. Open `http://localhost:3000` in your browser
4. Read and accept the consent form
5. Click "Start Test" to begin
6. Type the displayed sentences
7. Complete all sentences to finish the test

### Viewing Collected Data

Use the preview script to summarize collected data:

```bash
python scripts/preview_csv.py [data_directory]
```

Example output:
```
KEYSTROKE DATA SUMMARY
================================================================================

File: keystrokes_20240115.csv
Date: 20240115
  Total keystrokes: 523
  Unique participants: 1
  Unique sessions: 1
  Sample session WPM: 45.2
  Average key hold duration: 125.3 ms
```

### Data Files

CSV files are created in the `data/` directory (or as specified by `DATA_DIR`):

- **keystrokes_YYYYMMDD.csv**: Individual keystroke events
- **sessions_YYYYMMDD.csv**: Session summaries

Files are rotated daily to manage size and enable batch processing.

## Data Schema

See `shared/schema.md` for detailed schema documentation.

### Keystroke Events

Each keystroke event includes:
- `PARTICIPANT_ID`: Unique participant identifier
- `TEST_SECTION_ID`: Unique session identifier
- `SENTENCE`: Target sentence being typed
- `USER_INPUT`: Actual user input at capture time
- `KEYSTROKE_ID`: Sequential keystroke identifier
- `PRESS_TIME`: Key press timestamp (milliseconds)
- `RELEASE_TIME`: Key release timestamp (milliseconds)
- `LETTER`: Character or key name (e.g., 'a', 'SHIFT', 'BKSP')
- `KEYCODE`: JavaScript keyCode value

## API Endpoints

### POST `/api/session`
Create a new typing test session.

**Request:**
```json
{
  "participant_id": null  // Optional, will be generated if not provided
}
```

**Response:**
```json
{
  "participant_id": "uuid-here",
  "test_section_id": "uuid-here",
  "message": "Session created successfully"
}
```

### POST `/api/keystrokes`
Submit a batch of keystroke events.

**Request:**
```json
{
  "participant_id": "uuid",
  "test_section_id": "uuid",
  "sentence": "The quick brown fox...",
  "user_input": "The quick brown fox...",
  "keystrokes": [
    {
      "press_time": 1473284537607,
      "release_time": 1473284537771,
      "keycode": 84,
      "letter": "T"
    }
  ]
}
```

### GET `/api/session/{test_section_id}/stats`
Get statistics for a session.

### GET `/api/health`
Health check endpoint.

## Databricks Migration

When ready to migrate to Databricks:

1. Upload CSV files to a location accessible by Databricks (DBFS, S3, Azure Blob, etc.)
2. Open `notebooks/databricks_ingest.ipynb` in Databricks
3. Update configuration paths
4. Run the ingestion cells to create Delta tables and load data

See the notebook for detailed instructions and example queries.

## Development

### Running Tests

Backend tests (to be added):
```bash
cd backend
pytest
```

Frontend tests:
```bash
cd frontend
npm test
```

### Code Style

Backend: Follow PEP 8, use Black formatter
Frontend: Follow ESLint rules from react-scripts

## Privacy & Ethics

- Users must provide explicit consent before data collection
- Data collection is transparent and clearly explained
- Participants can decline to participate
- Data should be anonymized before sharing or analysis
- Follow applicable data protection regulations (GDPR, etc.)

## References

- [136 Million Keystrokes Project](https://github.com/aalto-ui/136m-keystrokes)
- [Observations on Typing from 136 Million Keystrokes](https://userinterfaces.aalto.fi/136Mkeystrokes/)

## License

MIT License - See LICENSE file for details

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## Support

For issues or questions, please open an issue on GitHub.

