# SatyanwesAI Extension - Backend

This directory contains the Python Flask backend servers for the SatyanwesAI Chrome Extension. It provides several microservices that analyze content (text, images, videos, audio) for potential misinformation and AI manipulation.

## Features

*   **Text Analysis (`check_text.py`):**
    *   Analyzes article content for credibility and fake news detection
    *   Uses machine learning models to classify text credibility
    *   Integrates with Google Fact Check API for verification
    *   Highlights potentially misleading text segments

*   **Sentiment/Bias Analysis (`check_sentiment.py`):**
    *   Analyzes text for sentiment (positive, neutral, negative)
    *   Detects political bias and provides bias indicators
    *   Provides a summary of the sentiment and bias analysis

*   **Image Analysis (`check_media.py`):**
    *   Detects potential AI-generated images using the Sightengine API
    *   Extracts text from images using the OCR.space API
    *   Provides confidence scores for AI manipulation detection

*   **Video Analysis (`check_video.py`) (Paid Tier Required):**
    *   Analyzes video properties using the Sightengine Video API
    *   Detects potential manipulations or deep fakes in videos
    *   Feature limited to users with a 'paid' subscription tier

*   **Audio Analysis (Planned) (Paid Tier Required):**
    *   Framework in place for detecting AI-generated audio
    *   Will require a third-party AI audio analysis API

*   **User Authentication & Authorization:**
    *   Identifies users via Google OAuth
    *   Checks user subscription tier (free/paid) stored in a PostgreSQL database
    *   Restricts premium features to users with a 'paid' tier

*   **Database Integration:**
    *   Uses PostgreSQL to store user information (Google ID, tier)
    *   Manages database connections using `psycopg2-binary` and a connection pool

*   **Configuration:**
    *   Loads API keys and database credentials from `.env` files
    *   Different configuration options for development and production

*   **CORS Enabled:**
    *   Allows requests from the Chrome extension frontend
    *   Proper handling of cross-origin requests

## Setup

1.  **Prerequisites:**
    *   Python 3.8+ (recommended: Python 3.10)
    *   PostgreSQL Database Server
    *   API keys for third-party services

2.  **Clone the repository (if not already done):**
    ```bash
    git clone <repository_url>
    cd <repository_directory>/extension/backend
    ```

3.  **Create a virtual environment (recommended):**
    ```bash
    python -m venv venv
    # On Windows
    .\venv\Scripts\activate
    # On macOS/Linux
    source venv/bin/activate
    ```

4.  **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

5.  **Set up the Database:**
    *   Ensure your PostgreSQL server is running
    *   Create a database (e.g., `news_analysis_db`)
    *   Create a user and grant privileges (e.g., `user` with `password`)
    *   Run the SQL script `db.sql` to create the necessary database schema:
        ```bash
        psql -h <db_host> -p <db_port> -U <db_user> -d <db_name> -f db.sql
        ```
        (Replace placeholders with your actual database details)

6.  **Configure Environment Variables:**
    *   Create `.env` files for each service in the `extension/backend` directory
    *   For the media analysis service (`check_media.py`):

    ```dotenv
    # Sightengine API Credentials
    SIGHTENGINE_API_USER=YOUR_SIGHTENGINE_API_USER
    SIGHTENGINE_API_SECRET=YOUR_SIGHTENGINE_API_SECRET

    # OCR.space API Key
    OCR_SPACE_API_KEY=YOUR_OCR_SPACE_API_KEY

    # Google OAuth
    GOOGLE_CLIENT_ID=YOUR_GOOGLE_CLIENT_ID

    # Database Configuration
    DB_HOST=localhost
    DB_PORT=5432
    DB_NAME=news_analysis_db
    DB_USER=user
    DB_PASSWORD=password

    # Flask Settings
    PORT=3000
    FLASK_DEBUG=True  # Set to False in production
    ```

    *   For the text analysis service (`check_text.py`):

    ```dotenv
    # Google Fact Check API
    GOOGLE_FACT_CHECK_API_KEY=YOUR_GOOGLE_FACT_CHECK_API_KEY

    # Google Gemini (optional)
    GOOGLE_GEMINI_API_KEY=YOUR_GOOGLE_GEMINI_API_KEY

    # GDELT API (optional)
    GDELT_API_KEY=YOUR_GDELT_API_KEY

    # News API (optional)
    NEWS_API_KEY=YOUR_NEWS_API_KEY

    # Flask Settings
    PORT=5000
    FLASK_DEBUG=True  # Set to False in production
    ```

    *   For the sentiment analysis service (`check_sentiment.py`):

    ```dotenv
    # Flask Settings
    PORT=5002
    FLASK_DEBUG=True  # Set to False in production
    ```

## API Endpoints

### Media Analysis Service (Port 3000)

*   **`GET /`**: Health check endpoint. Returns the status of the server and database connection.

*   **`POST /analyze_image`**:
    *   **Description:** Analyzes an image URL for potential AI manipulation
    *   **Request Body:** `{ "media_url": "image_url_here" }`
    *   **Headers:** `Authorization: Bearer <google_access_token>`
    *   **Response:** JSON object containing analysis results (AI detection confidence, extracted text)
    *   **Authentication:** Requires valid Google access token

*   **`POST /analyze_video`**:
    *   **Description:** Analyzes a video URL for manipulation detection
    *   **Request Body:** `{ "media_url": "video_url_here" }`
    *   **Headers:** `Authorization: Bearer <google_access_token>`
    *   **Response:** JSON object containing analysis results
    *   **Authentication:** Requires valid Google access token
    *   **Authorization:** Requires 'paid' user tier

*   **`POST /analyze_audio`**:
    *   **Description:** Analyzes an audio URL for AI generation detection
    *   **Request Body:** `{ "media_url": "audio_url_here" }`
    *   **Headers:** `Authorization: Bearer <google_access_token>`
    *   **Response:** JSON object containing analysis results
    *   **Authentication:** Requires valid Google access token
    *   **Authorization:** Requires 'paid' user tier
    *   **Note:** Currently returns status 501 (Not Implemented)

### Text Analysis Service (Port 5000)

*   **`POST /analyze`**:
    *   **Description:** Analyzes article text for credibility
    *   **Request Body:** `{ "url": "article_url", "text": "article_content" }`
    *   **Headers:** `Authorization: Bearer <google_access_token>` (optional)
    *   **Response:** JSON object with credibility analysis, highlights, and fact-checking sources

### Sentiment/Bias Analysis Service (Port 5002)

*   **`POST /analyze_sentiment_bias`**:
    *   **Description:** Analyzes text for sentiment and political bias
    *   **Request Body:** `{ "text": "content_to_analyze" }`
    *   **Headers:** `Authorization: Bearer <google_access_token>` (optional)
    *   **Response:** JSON object with sentiment scores and bias indicators

## Running the Services

1. **Media Analysis Service:**
```bash
python check_media.py
```
The server will start on port 3000 (or the port specified in the `.env` file).

2. **Text Analysis Service:**
```bash
python check_text.py
```
The server will start on port 5000 (or the port specified in the `.env` file).

3. **Sentiment/Bias Analysis Service:**
```bash
python check_sentiment.py
```
The server will start on port 5002 (or the port specified in the `.env` file).

4. **Video Analysis Service:**
```bash
python check_video.py
```
The server will start on port 5001 (or the port specified in the `.env` file).

For production deployments, consider using a production-grade WSGI server like Gunicorn or Waitress behind a reverse proxy like Nginx.

## Testing

You can run basic tests for the backend services using:

```bash
python test_backend.py
```

This will verify that the services are running and responding correctly.

## Dependencies

See `requirements.txt` for a full list of Python packages. Key dependencies include:

*   Flask & Flask-Cors: Web framework and CORS support
*   psycopg2-binary: PostgreSQL database connector
*   requests: HTTP client for API calls
*   python-dotenv: Environment variable management
*   google-auth: Google OAuth authentication
*   torch & torchvision: For machine learning models
*   google-generativeai: Google Gemini API client for AI reasoning
*   nltk: Natural language processing
