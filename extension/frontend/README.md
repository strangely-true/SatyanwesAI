# SatyanwesAI: Fake News Detection Chrome Extension

A comprehensive Chrome extension that helps users identify fake news, misinformation, and manipulated media by performing real-time analysis of web content using machine learning, fact-checking APIs, and media forensics.

## Features

- 🔍 Real-time article content and media analysis
- 🤖 Machine learning-based fake news detection
- 📊 Sentiment and political bias analysis
- 📷 AI-generated image detection
- 📚 Integration with fact-checking services
- 🎭 Detailed credibility assessment
- 🔔 In-depth analysis view in side panel
- 🎨 Modern interface with customizable theme support
- 🎯 Automatic highlighting of misleading text segments
- 👁️ Visual indicators for content credibility

## Technical Architecture

SatyanwesAI follows a modular architecture with four main frontend components that interact with multiple backend microservices:

### 1. Content Script (content.js)

Extracts content from web pages and communicates with the background script.

**Key Responsibilities:**
- Extracts text from article-like web pages
- Identifies and monitors images and video content
- Sends content to the background script for analysis
- Highlights potentially misleading content on the page
- Adds analysis overlays to media elements
- Handles DOM manipulation for visual feedback

**Code Snippet:**
```javascript
// Function to extract article content
function extractArticleContent() {
  try {
    const selectors = [
      'article',
      '[role="article"]',
      '.article-content',
      '.post-content',
      'main',
      '.main-content'
    ];

    let articleElement = null;
    for (const selector of selectors) {
      const element = document.querySelector(selector);
      if (element) {
        articleElement = element;
        break;
      }
    }

    // If no article element found, use body content
    if (!articleElement) {
      articleElement = document.body;
    }

    // Extract text content
    const content = articleElement.innerText
      .replace(/\s+/g, ' ')
      .trim();

    return content;
  } catch (error) {
    console.error('Error extracting content:', error);
    return '';
  }
}
```

### 2. Background Script (background.js)

Acts as the central hub for the extension, coordinating communication between components and backend services.

**Key Responsibilities:**
- Receives content from the content script
- Sends data to separate backend endpoints for analysis:
  - Text content to the text analysis service
  - Media URLs to the media analysis service
  - Text content to the sentiment/bias analysis service
- Stores analysis results per tab
- Distributes results to UI components and content script
- Manages user authentication and token handling
- Coordinates theme preferences across components

**Code Snippet:**
```javascript
// Define backend endpoints
const TEXT_ANALYSIS_URL = "http://127.0.0.1:5000/analyze";
const IMAGE_ANALYSIS_URL = "http://127.0.0.1:3000/analyze_image";
const VIDEO_ANALYSIS_URL = "http://127.0.0.1:3000/analyze_video";
const AUDIO_ANALYSIS_URL = "http://127.0.0.1:3000/analyze_audio";
const SENTIMENT_BIAS_ANALYSIS_URL = "http://127.0.0.1:5002/analyze_sentiment_bias";
const GOOGLE_USERINFO_URL = 'https://www.googleapis.com/oauth2/v1/userinfo?alt=json';

// Keep track of active connections and processing state per tab
let activeConnections = new Set();
let processingState = {}; // { tabId: { textResult: ..., mediaResult: ..., mediaItems: { url: result, ... }, sentimentBiasResult: ... } }
```

### 3. Popup UI (popup.js)

Provides a quick summary of analysis results in a compact popup interface.

**Key Responsibilities:**
- Fetches analysis results for the current tab
- Displays a credibility assessment summary
- Shows sentiment and bias information
- Manages user authentication status
- Provides a button to open the detailed side panel
- Supports theme customization and synchronization

**Code Snippet:**
```javascript
// Function to update UI based on analysis result
function updateUI(data) {
    if (!data || (!data.textResult && !data.sentimentBiasResult)) {
        statusDiv.textContent = 'No analysis data available yet.';
        statusIndicator.className = 'status-indicator unknown';
        return;
    }

    // Display Text Analysis Result
    if (data.textResult) {
        if (data.textResult.error) {
            statusDiv.textContent = `Error: ${data.textResult.error}`;
            statusIndicator.className = 'status-indicator unknown';
        } else if (data.textResult.label !== undefined) {
            const isFake = data.textResult.label === "LABEL_1";
            const confidence = (data.textResult.score * 100).toFixed(1);
            
            statusDiv.textContent = `${isFake ? 
                'This content may be misleading' : 
                'This content appears to be authentic'} (${confidence}% confidence)`;
            
            // Update visual indicator based on credibility
            statusIndicator.className = `status-indicator ${isFake ? 'fake' : 'real'}`;
        }
    }

    // Display Sentiment/Bias Data
    if (data.sentimentBiasResult) {
        sentimentBiasSection.classList.remove('hidden');
        
        // Sentiment display
        const sentiment = data.sentimentBiasResult.sentiment;
        if (sentiment && sentiment.label) {
            let sentimentText = sentiment.label;
            let sentimentColorClass = 'bg-gray-200 text-gray-700';
            let sentimentIcon = '';
            
            // Configure display based on sentiment
            if (sentimentText.toLowerCase() === 'positive') {
                sentimentColorClass = 'bg-green-100 text-green-800';
                sentimentIcon = `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-3 h-3"><path stroke-linecap="round" stroke-linejoin="round" d="M4.5 10.5L12 3m0 0l7.5 7.5M12 3v18" /></svg>`;
            } else if (sentimentText.toLowerCase() === 'negative') {
                sentimentColorClass = 'bg-red-100 text-red-800';
                sentimentIcon = `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-3 h-3"><path stroke-linecap="round" stroke-linejoin="round" d="M19.5 13.5L12 21m0 0l-7.5-7.5M12 21V3" /></svg>`;
            } else if (sentimentText.toLowerCase() === 'neutral') {
                sentimentColorClass = 'bg-blue-100 text-blue-800';
                sentimentIcon = `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-3 h-3"><path stroke-linecap="round" stroke-linejoin="round" d="M19.5 12h-15" /></svg>`; 
            }
            sentimentDisplay.innerHTML = `${sentimentIcon} ${sentimentText}`;
            sentimentDisplay.classList.add(...sentimentColorClass.split(' '));
        }
        
        // Bias tags display
        const bias = data.sentimentBiasResult.bias;
        if (bias && bias.indicators && bias.indicators.length > 0) {
            biasTags.innerHTML = '';
            bias.indicators.forEach(indicator => {
                const tag = document.createElement('span');
                tag.className = 'inline-block bg-gray-100 text-gray-600 text-xs px-2 py-1 rounded mr-1 mb-1';
                tag.textContent = `${indicator.charAt(0).toUpperCase() + indicator.slice(1)}`; 
                biasTags.appendChild(tag);
            });
        }
    }
    
    actionButton.textContent = 'View Details';
    actionButton.disabled = false;
}
```

### 4. Side Panel UI (sidepanel.js)

Provides comprehensive analysis results including confidence scores, fact-checking sources, and media analysis.

**Key Responsibilities:**
- Displays detailed credibility assessment with confidence scores
- Shows AI-generated reasoning about the content
- Presents sentiment and bias analysis
- Displays fact-checking sources and related information
- Shows media manipulation detection results
- Offers theme customization options
- Provides an interface for user-initiated analysis

**Code Snippet:**
```javascript
// Function to generate AI reasoning about the content
function generateAIReasoning(data) {
    let reasoningContent = '';
    
    if (!data || (!data.textResult && !data.mediaResult)) {
        reasoningContent = '<p>Insufficient data available for AI reasoning.</p>';
        return reasoningContent;
    }
    
    // Generate reasoning based on text analysis
    if (data.textResult && !data.textResult.error) {
        const isFake = data.textResult.label === "LABEL_1";
        
        if (isFake) {
            reasoningContent += `<p class="mb-3">This content appears to contain misinformation or misleading elements. Here's why:</p>`;
            reasoningContent += '<ul class="list-disc ml-5 mb-3">';
            if (data.textResult.reasoning && Array.isArray(data.textResult.reasoning)) {
                data.textResult.reasoning.forEach(point => { reasoningContent += `<li class="mb-2">${point}</li>`; });
            } else {
                reasoningContent += `<li class="mb-2">The content contains claims that contradict verified information.</li>`;
                reasoningContent += `<li class="mb-2">The narrative appears to be misleading or false.</li>`;
                reasoningContent += `<li class="mb-2">Multiple fact-checking sources have flagged similar claims.</li>`;
            }
            reasoningContent += '</ul>';
            
            if (data.textResult.highlights && data.textResult.highlights.length > 0) {
                reasoningContent += '<p class="mb-2 font-medium">Highlighted misleading segments:</p>';
                reasoningContent += '<ul class="list-disc ml-5 mb-3 italic text-gray-600 dark:text-gray-400">';
                data.textResult.highlights.forEach(highlight => { reasoningContent += `<li class="mb-1">"${highlight}"</li>`; });
                reasoningContent += '</ul>';
            }
        } else {
            reasoningContent += `<p class="mb-3">This content appears to be credible based on my analysis. Here's why:</p>`;
            reasoningContent += '<ul class="list-disc ml-5 mb-3">';
            if (data.textResult.reasoning && Array.isArray(data.textResult.reasoning)) {
                data.textResult.reasoning.forEach(point => { reasoningContent += `<li class="mb-2">${point}</li>`; });
            } else {
                reasoningContent += `<li class="mb-2">Claims align with verified information.</li>`;
                reasoningContent += `<li class="mb-2">No contradictions found with reputable sources.</li>`;
                reasoningContent += `<li class="mb-2">Contains verifiable details.</li>`;
            }
            reasoningContent += '</ul>';
        }
    }
    
    // Add media analysis reasoning if available
    if (data.mediaResult && (data.mediaResult.manipulated_images_found > 0 || data.mediaResult.images_analyzed > 0)) {
        reasoningContent += '<p class="font-semibold mt-4 mb-2">Media Analysis:</p>';
        if (data.mediaResult.manipulated_images_found > 0) {
            reasoningContent += `<p class="mb-3">Detected potential manipulation in ${data.mediaResult.manipulated_images_found} of ${data.mediaResult.images_analyzed} images.</p>`;
            if (data.mediaResult.manipulated_media && data.mediaResult.manipulated_media.length > 0) {
                reasoningContent += '<ul class="list-disc ml-5 mb-3">';
                data.mediaResult.manipulated_media.forEach(item => {
                    const manipType = item.manipulation_type ? item.manipulation_type.replace(/_/g, ' ') : 'AI generation';
                    const confidencePercent = (item.confidence * 100).toFixed(1);
                    reasoningContent += `<li class="mb-2">Detected ${manipType} (${confidencePercent}% confidence).</li>`;
                });
                reasoningContent += '</ul>';
            }
        } else if (data.mediaResult.images_analyzed > 0) {
            reasoningContent += `<p class="mb-3">Analyzed ${data.mediaResult.images_analyzed} images; no manipulation detected.</p>`;
        }
    }
    
    // Add conclusion
    if (data.textResult || data.mediaResult) {
        reasoningContent += '<p class="font-semibold mt-4 mb-2">Conclusion:</p>';
        if (data.textResult && data.textResult.label === "LABEL_1" && data.textResult.score > 0.7) {
            reasoningContent += '<p class="text-red-600 dark:text-red-400">Content appears misleading. Consult additional sources.</p>';
        } else if (data.mediaResult && data.mediaResult.manipulated_images_found > 0) {
            reasoningContent += '<p class="text-yellow-600 dark:text-yellow-400">Media contains manipulated elements. Exercise caution.</p>';
        } else if (data.textResult && data.textResult.label !== "LABEL_1") {
            reasoningContent += '<p class="text-green-600 dark:text-green-400">Content appears factually accurate and reliable.</p>';
        } else {
            reasoningContent += '<p>Analysis inconclusive. Seek additional sources.</p>';
        }
    }
    
    if (!reasoningContent) {
        reasoningContent = '<p>Analysis complete, but more information needed for detailed reasoning.</p>';
    }
    return reasoningContent;
}
```

## File Interactions and Communication Architecture

The SatyanwesAI extension follows a well-structured communication architecture where each component has specific responsibilities and communicates with others through defined channels:

### Content Script (content.js) Interactions

- **Initializes** automatically when a webpage is loaded
- **Extracts** article content and media elements from the page DOM
- **Communicates with background.js** by sending:
  - `processText` messages with article text for credibility analysis
  - Content is analyzed up to 3000 characters to fit API limits
- **Receives from background.js**:
  - `applyHighlights` messages containing text segments to highlight
  - `analysisComplete` notifications when text analysis is finished
  - `displayMediaAnalysis` for image/video/audio analysis results
- **Modifies the webpage** by highlighting misleading content and adding analysis overlays to media

### Background Script (background.js) Interactions

- **Acts as the central communication hub** between all components
- **Maintains state** for each analyzed tab using the `processingState` object, storing:
  - Text analysis results (`textResult`)
  - Media analysis results (`mediaItems`)
  - Sentiment/bias analysis results (`sentimentBiasResult`)
- **Handles requests from content.js**:
  - Processes text through text analysis API (port 5000)
  - Processes text through sentiment/bias API (port 5002)
  - Processes media URLs through media analysis API (port 3000)
  - Sends highlight instructions back to content script
- **Communicates with popup.js and sidepanel.js**:
  - Responds to `getResultForTab` requests with analysis data
  - Sends analysis completion notifications
- **Handles user authentication**:
  - Manages Google OAuth tokens
  - Verifies user access for premium features

### Popup UI (popup.js) Interactions

- **Initializes** when the user clicks the extension icon
- **Queries background.js** for current tab analysis results
- **Updates UI** based on credibility assessment:
  - Shows a color-coded status indicator (red/green/gray)
  - Displays a summary with confidence level
  - Shows sentiment and bias tags
- **Opens sidepanel.js** when the user clicks the "View Details" button
- **Synchronizes theme preferences** with sidepanel using `chrome.storage.local`
- **Listens for changes** to analysis results via `chrome.runtime.onMessage`

### Side Panel UI (sidepanel.js) Interactions

- **Initializes** when opened from the popup or via Chrome's side panel feature
- **Queries background.js** for detailed analysis results
- **Renders comprehensive results**:
  - Credibility score and confidence level
  - AI-generated reasoning about the content credibility
  - Sentiment and bias analysis
  - Media analysis findings
  - Fact-check sources and related information
- **Allows user-initiated analysis** of text snippets and media
- **Provides theme customization** that affects all extension UI components

### Theme Management Across Components

Both popup.js and sidepanel.js implement the same theme management logic:

1. **Theme Storage**: Preferences stored in `chrome.storage.local` as 'light', 'dark', or 'system'
2. **Theme Detection**: System theme detected via `window.matchMedia('(prefers-color-scheme: dark')`
3. **Theme Synchronization**: Changes in one component reflect in the other through storage events
4. **Theme Application**: HTML classes control the appearance of UI elements in both components

## Communication Flow

The extension follows this data flow pattern:

1. **Content → Background**:
   - Content script extracts article text and media
   - Sends text for analysis using `processText` action
   - Sends media URLs for analysis when user initiates media analysis

2. **Background → Backend Services**:
   - Background script sends text to text analysis service (port 5000)
   - Background script sends text to sentiment/bias analysis service (port 5002)
   - Background script sends media URLs to media analysis service (port 3000)

3. **Background → Content**:
   - Background script sends highlight instructions to content script
   - Background script sends media analysis results to content script for display

4. **Background ↔ UI Components**:
   - Popup and sidepanel request analysis results from background script
   - Background notifies UI components when new analysis results are available

5. **Theme Synchronization**:
   - Popup and side panel use shared storage for theme preferences
   - Changes in either component are reflected in the other

## Prerequisites

- Chrome browser (version 88 or higher)
- Python 3.8 or higher (for backend services)
- PostgreSQL database (for user management)
- API keys for third-party services (see backend setup)

## Installation

### 1. Clone the Repository

```bash
git clone <repository-url>
cd SatyanwesAI
```

### 2. Set Up the Backend Services

1. Navigate to the backend directory:
```bash
cd extension/backend
```

2. Create and activate a virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install required packages:
```bash
pip install -r requirements.txt
```

4. Create `.env` files for each service with appropriate API keys (see Backend README)

5. Start the backend services (each in a separate terminal):
```bash
python check_media.py  # Media analysis service (port 3000)
python check_text.py   # Text analysis service (port 5000)
python check_sentiment.py  # Sentiment/bias analysis (port 5002)
```

### 3. Load the Extension in Chrome

1. Open Chrome and navigate to `chrome://extensions/`
2. Enable "Developer mode" in the top right corner
3. Click "Load unpacked" and select the `extension/frontend` directory
4. The SatyanwesAI icon should appear in your Chrome toolbar

## Usage

1. **Automatic Analysis**:
   - Navigate to any news article or content page
   - The extension will automatically analyze the page content
   - Click the extension icon to see a summary of the analysis

2. **Detailed Analysis**:
   - Click the "View Details" button in the popup
   - The side panel will open with comprehensive analysis results
   - Review credibility assessment, sentiment/bias analysis, fact-check sources, and media analysis

3. **Visual Indicators**:
   - Potentially misleading text segments are highlighted on the page
   - The popup shows a color-coded credibility indicator
   - The side panel displays confidence scores and AI reasoning

4. **Media Analysis**:
   - Images on the page can be analyzed for AI manipulation
   - Results show manipulation confidence and extracted text
   - Premium users can analyze video and audio content

5. **Theme Customization**:
   - Click the theme toggle button in either the popup or side panel
   - Choose between light, dark, or system theme

## Development

*   **Making Changes:** After modifying any frontend file (HTML, CSS, JS), you need to reload the extension in `chrome://extensions` by clicking the refresh icon.

*   **Debugging:**
    *   **Popup:** Right-click the extension icon and select "Inspect popup".
    *   **Background Script:** Go to `chrome://extensions`, find SatyanwesAI, and click the "service worker" link.
    *   **Content Script:** Open the developer tools (F12) on a webpage where the content script is active and check the Console tab (make sure the context is set to the page, not the extension).
    *   **Side Panel:** Right-click within the panel and select "Inspect".

*   **Backend URL:** If your backend services are running on different ports or hosts, update the endpoint URLs in `background.js`:
    ```javascript
    const TEXT_ANALYSIS_URL = "http://127.0.0.1:5000/analyze";
    const IMAGE_ANALYSIS_URL = "http://127.0.0.1:3000/analyze_image";
    // ...etc.
    ```

## Key Files

*   `manifest.json`: Extension configuration and permissions
*   `background.js`: Background service worker that coordinates communication
*   `content.js`: Content script that extracts and modifies web page content
*   `popup.html/js`: Extension popup UI and logic
*   `sidepanel.html/js`: Detailed analysis panel UI and logic

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- BERT and other machine learning models for text classification
- Sightengine API for media analysis
- Google Fact Check API for fact-checking
- Chrome Extension APIs for browser integration