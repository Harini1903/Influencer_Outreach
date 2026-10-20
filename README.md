# Influencer Outreach Automation 🚀

An AI-powered influencer discovery and outreach automation project built with Python, Google Colab, the YouTube Data API, and Google Gemini.

The project helps streamline the process of discovering micro-influencers, analyzing their engagement, identifying relevant creators, generating personalized outreach messages, and tracking outreach activities.

## 🎯 Project Objective

Influencer marketing research can be time-consuming when performed manually. This project automates several steps in the influencer outreach workflow to help marketers identify potential creators and prepare personalized collaboration proposals more efficiently.

## ✨ Key Features

### 1. Influencer Discovery
- Searches YouTube using multiple beauty and fashion-related keywords.
- Collects channel information through the YouTube Data API.
- Removes duplicate channels using channel IDs.

### 2. Influencer Data Collection
Collects available channel information, including:
- Channel name and URL
- Subscriber count
- Video count
- Country, when available
- Channel description
- Recent video titles and publishing dates

### 3. Engagement Analysis
Analyzes recent YouTube videos using views, likes, and comments.

The engagement rate is calculated as:

`Engagement Rate (%) = (Total Likes + Total Comments) / Total Views × 100`

The notebook also calculates average views and tracks the number of videos used in the analysis.

### 4. Rule-Based Influencer Qualification
Filters and evaluates creators using configurable criteria, including:
- Subscriber count: 5,000–100,000
- Minimum engagement rate: 1%
- Minimum average views: 500
- Minimum videos used for engagement analysis: 5
- Recent channel activity
- Beauty and fashion niche relevance

Creators with unusually high engagement or other potential issues can be flagged for manual review.

### 5. Niche Classification and Review Flags
Uses keyword-based classification to identify Beauty, Fashion, or Other categories.

Additional checks flag possible seller accounts, content-language concerns, irrelevant recent videos, and missing contact routes.

### 6. AI-Powered Personalized Outreach
Uses Google Gemini to generate:
- Personalized email subject lines
- Email outreach drafts
- Instagram DM drafts

The prompt instructs the model to use available creator information, avoid unsupported claims, and follow specified message-length requirements.

### 7. Output Validation
- Checks email and DM word counts.
- Checks for prohibited or misleading phrases.
- Identifies messages that need additional review.
- Excludes rejected creators from outreach.

### 8. Human-in-the-Loop Approval
Creators can be marked as pending, approved, rejected, or requiring manual review.

Human review helps ensure that outreach is appropriate before any sending workflow proceeds.

### 9. Outreach Tracking
Uses SQLite to maintain an outreach log and Instagram DM queue.

Exports the following CSV files:

- `influencers_with_messages.csv` — influencer details and generated message drafts.
- `outreach_tracker.csv` — message generation, sending, and status information.
- `instagram_dm_queue.csv` — Instagram DM drafts and queue statuses.

## 🛠️ Tech Stack

- **Language:** Python
- **Environment:** Google Colab, Jupyter Notebook
- **Data Processing:** Pandas
- **Data Source:** YouTube Data API v3
- **Generative AI:** Google Gemini API
- **Database:** SQLite
- **Output Formats:** CSV

## 🔄 Workflow

1. Search YouTube for relevant channels.
2. Collect channel information.
3. Identify creators within the target subscriber range.
4. Analyze recent videos and engagement.
5. Apply qualification rules and niche classification.
6. Generate personalized outreach messages with Gemini.
7. Validate generated messages.
8. Review and approve suitable creators.
9. Prepare the email and Instagram DM queues.
10. Export the outreach tracker and CSV reports.

## 🚀 Getting Started

### Prerequisites

- A Google account
- Access to Google Colab
- A YouTube Data API key
- A Google Gemini API key

### Setup Instructions

1. Clone or download this repository.
2. Open `Influencer_Outreach.ipynb` in Google Colab.
3. Configure your API keys using **Colab Secrets**.
4. Use the following secret names, matching the notebook code:

   - `YOUTUBE_API_KEY`
   - `GEMINI_API_KEY`

5. Install the required dependencies.

   ```python
   !pip install -q pandas google-api-python-client google-genai
   ```

6. Run the notebook cells in order.
7. Review the generated results and download the CSV reports.

The notebook uses Google Colab's `userdata` to retrieve API keys without hardcoding them into Python cells.

## 🔐 Security and Responsible Use

- Never commit API keys, passwords, or other credentials to GitHub.
- Avoid publishing private contact information or sensitive outreach data.
- Respect platform API terms, rate limits, and applicable privacy requirements.
- Review AI-generated messages before contacting creators.
- Verify contact information instead of assuming missing details.
- Keep the email workflow in dry-run mode while testing.

## ⚠️ Limitations

- Influencer discovery is limited to the configured YouTube search queries.
- Niche classification relies on keywords rather than a trained machine learning model.
- Engagement metrics are based on available public video statistics and may not represent the creator's complete audience performance.
- Contact information may be unavailable or require independent verification.
- AI-generated messages require human review.
- The sample brand, **Glowly**, is fictional and is used to demonstrate outreach generation.
- Instagram messages are prepared in a queue; this notebook does not automatically send Instagram DMs through an official Instagram messaging integration.

## 🔮 Future Enhancements

- Creator scoring and ranking
- Semantic niche classification using embeddings or LLMs
- Improved contact extraction and verification
- Campaign-specific influencer selection
- A Streamlit review dashboard
- Integration with a supported email service
- Campaign response and conversion analytics
- Automated tests, structured logging, and modular application architecture

## 📌 Project Status

**Prototype / Proof of Concept**

This project demonstrates how data collection, rule-based filtering, generative AI, human review, and database tracking can be combined in an influencer outreach workflow.

## 👩‍💻 Author

Harini Parthiban

