# SatyanwesAI Project

SatyanwesAI is a comprehensive web application and Chrome extension designed to help users identify fake news, misinformation, and potentially AI-generated or manipulated media (images, videos) encountered online.

## Components

This repository contains two main parts:

1.  **`extension/`**: The Chrome Extension
    *   **`extension/frontend/`**: Contains the HTML, CSS, and JavaScript for the extension's user interface (popup, content scripts, background script, side panel). See `extension/frontend/README.md` for details.
    *   **`extension/backend/`**: A Python Flask server that handles content analysis by communicating with third-party APIs (Sightengine, OCR.space, etc.) and manages user authentication/authorization via a PostgreSQL database. See `extension/backend/README.md` for details.
2.  **`landing/`**: The Landing Page & Web Application
    *   A Next.js application serving as the project's landing page.
    *   Provides information about SatyanwesAI.
    *   Implements user authentication using Clerk.
    *   Features a dashboard for registered users, allowing them to manage their subscription (free/paid tiers) which affects access to advanced features in the *extension*.
    *   Built with React, TypeScript, Tailwind CSS, and Shadcn UI components.

## Core Functionality

*   **Text Analysis (Extension):** The Chrome extension automatically analyzes the text content of pages for credibility.
*   **Media Detection (Extension):** The extension allows users to right-click on media (or use the popup) to initiate an analysis.
*   **Backend Analysis (Extension Backend):** Requests are sent to multiple Flask backend services:
    *   **Text:** Analyzed for credibility using a machine learning model.
    *   **Sentiment/Bias:** Analyzed for sentiment and political bias.
    *   **Images:** Checked for AI generation (Sightengine) and text content (OCR.space).
    *   **Video (Paid Tier):** Checked for potential manipulation/scams (Sightengine Video) or AI generation.
*   **User Tiers (Backend & Landing Page):** The backend checks the user's tier (stored in PostgreSQL) before allowing access to premium features. Users manage their subscription through the Next.js landing page/dashboard.
*   **Results Display (Extension Frontend):** Analysis results (credibility scores, confidence levels, extracted text, warnings) are displayed through the extension's popup or side panel.

## Getting Started

To run the full project, you need to set up and run both the extension backend services and the landing page application.

1.  **Set up the Extension Backend:** 
    * Follow the instructions in `extension/backend/README.md`
    * Install Python dependencies with `pip install -r requirements.txt`
    * Set up PostgreSQL database
    * Configure `.env` files with your API keys and credentials

2.  **Set up the Landing Page:** 
    * Install Node.js and pnpm
    * Run `pnpm install` in the `landing/` directory
    * Configure environment variables for Clerk authentication

3.  **Run the Backend Services:** 
    * Start the main media analysis server: `python extension/backend/check_media.py`
    * Start the text analysis server: `python extension/backend/check_text.py` 
    * Start the sentiment/bias analysis server: `python extension/backend/check_sentiment.py`
    * (Optional) Start the video analysis server: `python extension/backend/check_video.py`

4.  **Run the Landing Page:** 
    * Start the Next.js development server: `pnpm dev` in the `landing/` directory

5.  **Load the Extension:** 
    * Open Chrome and navigate to `chrome://extensions/`
    * Enable "Developer mode" in the top right corner
    * Click "Load unpacked" and select the `extension/frontend/` directory

## Project Structure Overview

```
/
├── extension/
│   ├── backend/                    # Python Flask servers
│   │   ├── check_media.py          # Media analysis server (images)
│   │   ├── check_text.py           # Text credibility analysis server
│   │   ├── check_sentiment.py      # Sentiment and bias analysis server
│   │   ├── check_video.py          # Video analysis service
│   │   ├── requirements.txt        # Python dependencies
│   │   ├── db.sql                  # Database setup script
│   │   ├── test_backend.py         # Backend tests
│   │   └── README.md               # Backend documentation
│   │
│   └── frontend/                   # Chrome Extension UI
│       ├── manifest.json           # Extension configuration
│       ├── background.js           # Background service worker
│       ├── content.js              # Content script running on websites
│       ├── popup.html/js           # Extension popup UI
│       ├── sidepanel.html/js       # Detailed analysis view
│       └── README.md               # Frontend documentation
│
├── landing/                        # Next.js Landing Page & Dashboard
│   ├── app/                        # Next.js App Router pages
│   │   ├── globals.css             # Global styles
│   │   ├── page.tsx                # Landing page component
│   │   ├── layout.tsx              # Root layout component
│   │   ├── dashboard/              # User dashboard pages
│   │   └── sign-in/                # Authentication pages
│   │
│   ├── components/                 # React components
│   │   ├── ui/                     # Shadcn UI components
│   │   └── various feature components
│   │
│   ├── public/                     # Static assets
│   ├── package.json                # Dependencies and scripts
│   └── tailwind.config.ts          # Tailwind configuration
│
└── README.md                       # This file (Overall Project Overview)
```

Refer to the README files within each subdirectory (`extension/backend`, `extension/frontend`) for more specific details on setup, configuration, and usage.
