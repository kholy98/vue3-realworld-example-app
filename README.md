# Article Revisions Feature Implementation

This project extends the RealWorld example app with a comprehensive article revisions system, allowing authors to view, compare, and revert to previous versions of their articles.

## Quick Installation

```bash
# Install dependencies (using --legacy-peer-deps to handle dependency conflicts)
npm install --legacy-peer-deps

# Start the development server
npm run dev
```


# Article Revisions Feature

## Overview

The revisions system provides authors with complete version history for their articles, including:

- **View all previous versions** - Access the complete history of article changes
- **See revision details** - Examine title, description, and body content for each version
- **Compare changes between versions** - Visual diff functionality to see what changed
- **Revert to any previous version** - Restore article content to any historical state
- **Audit trail with timestamps and author information** - Track who made changes and when

## Backend Implementation (Laravel PHP)

### API Endpoints

```php
// Get all revisions for an article
GET /api/articles/{article}/revisions

// Get specific revision details
GET /api/articles/{article}/revisions/{revision}

// Revert article to a specific revision
POST /api/articles/{article}/revisions/{revision}/revert
