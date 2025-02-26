# Voice Agent Generator: Project Progress

## Project Overview

We're building a system that allows EZAISolutions to quickly create voice agents for clients by scraping their websites. This will enable rapid deployment of voice agents that are knowledgeable about client businesses, products, and services.

## Current Status

- Base repository cloned (Bland Boilerplate)
- Basic understanding of the architecture established
- Implementation plan created

## Next Steps

### 1. Install Required Dependencies

```bash
# Navigate to voice-chat directory
cd voice-chat

# Install web scraping and HTTP request packages
npm install cheerio axios

# Install crawler for website mapping
npm install crawlee

# Install OpenAI packages for content processing
npm install openai-edge 

# Install token counter for prompt management
npm install gpt-tokenizer

# Install UI components (if needed)
npm install @radix-ui/react-dialog @radix-ui/react-form
```

### 2. Create Directory Structure for New Features

```
voice-chat/
├── src/
│   ├── lib/
│   │   ├── crawler/           # Website crawling functionality
│   │   │   ├── index.ts       # Main crawler module
│   │   │   ├── mapper.ts      # Site mapping
│   │   │   └── extractor.ts   # Content extraction
│   │   │
│   │   ├── processor/         # Content processing
│   │   │   ├── index.ts       # Main processor module
│   │   │   ├── classifier.ts  # Content classification
│   │   │   └── generator.ts   # Q&A generator
│   │   │
│   │   └── prompt/            # Prompt engineering
│   │       ├── index.ts       # Main prompt module
│   │       ├── templates.ts   # Prompt templates
│   │       └── optimizer.ts   # Prompt optimization
│   │
│   ├── components/
│   │   ├── WebsiteInput.tsx   # URL input component
│   │   ├── ScrapeStatus.tsx   # Scraping progress display
│   │   └── AgentConfig.tsx    # Agent configuration options
│   │
│   ├── pages/
│   │   ├── index.tsx          # Main page (updated)
│   │   ├── api/
│   │   │   ├── scrape.ts      # Website scraping endpoint
│   │   │   ├── process.ts     # Content processing endpoint
│   │   │   └── agent.ts       # Agent creation endpoint
```

### 3. Implementation Tasks (In Order)

#### Phase 1: Website Scraping

1. **Create Website Crawler Module**
   - Implement URL validation
   - Set up crawling with proper rate limits
   - Create site mapper to identify important pages
   - Develop content extractor to clean HTML

2. **Build API Endpoints**
   - Create `/api/scrape` endpoint to handle website scraping requests
   - Implement proper error handling and status updates
   - Set up storage for scraped content

3. **Develop Frontend Components**
   - Create `WebsiteInput` component for URL submission
   - Build `ScrapeStatus` to show progress
   - Add proper loading states and error handling

#### Phase 2: Content Processing

4. **Create Content Processor**
   - Implement content classification (products, services, FAQs, etc.)
   - Build structured data formatter
   - Create Q&A generator from content

5. **Build Prompt Generator**
   - Design templates for different agent types
   - Implement prompt construction from processed content
   - Create optimization layer to fit within token limits

6. **API Integration**
   - Create `/api/process` endpoint for content processing
   - Create `/api/agent` endpoint for agent creation with Bland API

#### Phase 3: Agent Management

7. **Agent Configuration**
   - Build agent personality customization
   - Add voice selection options
   - Create testing interface

8. **Analytics and Improvement**
   - Implement feedback collection
   - Create performance tracking
   - Add agent refinement tools

## Immediate Next Steps

1. Set up the directory structure for new modules
2. Implement the basic website crawler functionality
3. Create the WebsiteInput component and scrape API endpoint
4. Test the basic scraping functionality

## Sample Code To Get Started

### 1. URL Validation (src/lib/crawler/utils.ts)

```typescript
/**
 * Validates and normalizes a URL
 */
export function validateUrl(url: string): string | null {
  try {
    // Try to construct a URL object (will throw if invalid)
    const urlObj = new URL(url);
    
    // Ensure it's HTTP or HTTPS
    if (!['http:', 'https:'].includes(urlObj.protocol)) {
      return null;
    }
    
    // Normalize to include trailing slash on root domain if needed
    if (urlObj.pathname === '') {
      urlObj.pathname = '/';
    }
    
    return urlObj.toString();
  } catch (error) {
    // If URL construction fails, try adding https:// and retry
    if (!url.startsWith('http')) {
      return validateUrl(`https://${url}`);
    }
    return null;
  }
}

/**
 * Extracts the domain from a URL
 */
export function getDomain(url: string): string {
  try {
    const urlObj = new URL(url);
    return urlObj.hostname;
  } catch (error) {
    return '';
  }
}
```

### 2. Basic Crawler (src/lib/crawler/index.ts)

```typescript
import axios from 'axios';
import * as cheerio from 'cheerio';
import { validateUrl, getDomain } from './utils';

interface CrawlResult {
  url: string;
  title: string;
  content: string;
  links: string[];
}

export async function crawlPage(url: string): Promise<CrawlResult | null> {
  const validUrl = validateUrl(url);
  if (!validUrl) {
    throw new Error(`Invalid URL: ${url}`);
  }
  
  try {
    const response = await axios.get(validUrl, {
      headers: {
        'User-Agent': 'EZAISolutions Voice Agent Generator/1.0',
      },
    });
    
    const html = response.data;
    const $ = cheerio.load(html);
    
    // Extract title
    const title = $('title').text().trim();
    
    // Extract main content (simplified - will need more sophisticated extraction)
    // Remove script tags, style tags, and comments
    $('script, style, comment').remove();
    
    // Get text from the body
    const content = $('body').text().replace(/\s+/g, ' ').trim();
    
    // Extract links
    const links: string[] = [];
    const domain = getDomain(validUrl);
    
    $('a').each((_, element) => {
      const href = $(element).attr('href');
      if (href) {
        try {
          // Resolve relative URLs
          const resolvedUrl = new URL(href, validUrl).toString();
          
          // Only include links from the same domain
          if (getDomain(resolvedUrl) === domain) {
            links.push(resolvedUrl);
          }
        } catch (e) {
          // Ignore invalid URLs
        }
      }
    });
    
    return {
      url: validUrl,
      title,
      content,
      links: [...new Set(links)], // Remove duplicates
    };
  } catch (error) {
    console.error(`Error crawling ${url}:`, error);
    return null;
  }
}
```

### 3. WebsiteInput Component (src/components/WebsiteInput.tsx)

```typescript
import React, { useState } from 'react';
import { Button } from './ui/button';
import { Card, CardContent, CardHeader, CardTitle } from './ui/card';
import { Input } from './ui/input';
import { validateUrl } from '../lib/crawler/utils';

interface WebsiteInputProps {
  onSubmit: (url: string) => void;
  isLoading: boolean;
}

export default function WebsiteInput({ onSubmit, isLoading }: WebsiteInputProps) {
  const [url, setUrl] = useState('');
  const [error, setError] = useState<string | null>(null);
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    // Validate URL
    const validUrl = validateUrl(url);
    if (!validUrl) {
      setError('Please enter a valid URL');
      return;
    }
    
    setError(null);
    onSubmit(validUrl);
  };
  
  return (
    <Card className="w-full max-w-md mx-auto border-none bg-white/10 backdrop-blur-lg shadow-2xl">
      <CardHeader className="pb-4">
        <CardTitle className="text-2xl font-light text-white">
          Create Voice Agent
        </CardTitle>
      </CardHeader>
      <CardContent>
        <form onSubmit={handleSubmit} className="space-y-4">
          <div className="space-y-2">
            <Input
              placeholder="Enter website URL (e.g., example.com)"
              value={url}
              onChange={(e) => setUrl(e.target.value)}
              className="bg-white/5 border-white/10 text-white"
              disabled={isLoading}
            />
            {error && (
              <div className="text-red-300 text-sm">{error}</div>
            )}
          </div>
          <Button 
            type="submit" 
            className="w-full bg-purple-500/80 hover:bg-purple-600/80"
            disabled={isLoading}
          >
            {isLoading ? 'Processing...' : 'Create Voice Agent'}
          </Button>
        </form>
      </CardContent>
    </Card>
  );
}
```

### 4. API Endpoint (src/pages/api/scrape.ts)

```typescript
import type { NextApiRequest, NextApiResponse } from 'next';
import { crawlPage } from '../../lib/crawler';
import { validateUrl } from '../../lib/crawler/utils';

type ScrapeResponse = {
  success: boolean;
  data?: any;
  error?: string;
};

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse<ScrapeResponse>
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ success: false, error: 'Method not allowed' });
  }

  try {
    const { url } = req.body;
    
    if (!url) {
      return res.status(400).json({ success: false, error: 'URL is required' });
    }
    
    const validUrl = validateUrl(url);
    if (!validUrl) {
      return res.status(400).json({ success: false, error: 'Invalid URL' });
    }
    
    // Crawl the website
    const result = await crawlPage(validUrl);
    
    if (!result) {
      return res.status(500).json({ success: false, error: 'Failed to crawl website' });
    }
    
    // Return the crawled data
    return res.status(200).json({ success: true, data: result });
  } catch (error) {
    console.error('Error in scrape API:', error);
    return res.status(500).json({ 
      success: false, 
      error: error instanceof Error ? error.message : 'Unknown error' 
    });
  }
}
```

## Notes

- The OpenAI Edge package requires an API key, which will need to be added to the environment variables
- Need to consider rate limits for both crawling and API usage
- Should implement caching to avoid redundant scraping and processing
- The content extraction in these examples is very basic - you'll need more sophisticated extraction for production use
- For a full implementation, we'll need to add site mapping to crawl multiple pages 