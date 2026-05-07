# Claude API Integration

## Overview

Use Claude to analyze venture data and generate market intelligence.

**Setup time: ~10 minutes**

## Step 1: Get Claude API Key

1. Go to [Anthropic Console](https://console.anthropic.com)
2. Sign up or login
3. Go to **API Keys** (left sidebar)
4. Click **"Create Key"**
5. Name: `worklife-vc-hub`
6. Copy the key (save safely)

## Step 2: Set Usage Limits

1. Go to **Billing** in Anthropic Console
2. Set monthly limit: `$50` (for safety)
3. Enable notifications at $30

## Step 3: Update Environment

In `.env.local`:

```env
CLAUDE_API_KEY=sk-ant-...
CLAUDE_MODEL=claude-3-5-sonnet-20241022
```

## Step 4: Create Claude Client

Create `src/lib/claude.ts`:

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({
  apiKey: process.env.CLAUDE_API_KEY,
});

// Analyze a venture
export async function analyzeVenture(ventureData: any) {
  const prompt = `
Analyze this venture investment opportunity and provide insights:

Company: ${ventureData.name}
Stage: ${ventureData.stage}
Industry: ${ventureData.industry}
Founder: ${ventureData.founder}
Investment Amount: $${ventureData.amount}
Description: ${ventureData.description}

Provide:
1. Market opportunity assessment
2. Competitive landscape
3. Risk factors
4. Growth potential
5. Recommendation (Invest/Pass/Follow)
  `;

  try {
    const message = await client.messages.create({
      model: process.env.CLAUDE_MODEL || 'claude-3-5-sonnet-20241022',
      max_tokens: 1024,
      messages: [
        {
          role: 'user',
          content: prompt,
        },
      ],
    });

    return message.content[0].type === 'text' ? message.content[0].text : '';
  } catch (error) {
    console.error('Claude API error:', error);
    throw error;
  }
}

// Market intelligence from news
export async function generateMarketIntel(newsItems: any[]) {
  const newsText = newsItems
    .map((item) => `- ${item.title}: ${item.description}`)
    .join('\n');

  const prompt = `
Based on these recent news items, provide market intelligence summary:

${newsText}

Provide:
1. Key trends
2. Opportunities for our portfolio
3. Risks to monitor
4. Investment implications
  `;

  try {
    const message = await client.messages.create({
      model: process.env.CLAUDE_MODEL || 'claude-3-5-sonnet-20241022',
      max_tokens: 2048,
      messages: [
        {
          role: 'user',
          content: prompt,
        },
      ],
    });

    return message.content[0].type === 'text' ? message.content[0].text : '';
  } catch (error) {
    console.error('Claude API error:', error);
    throw error;
  }
}

// Streaming response (for real-time UI updates)
export async function analyzeVentureStream(ventureData: any) {
  const prompt = `
Analyze this venture: ${ventureData.name} (${ventureData.industry})
Provide quick investment assessment.
  `;

  return client.messages.stream({
    model: process.env.CLAUDE_MODEL || 'claude-3-5-sonnet-20241022',
    max_tokens: 1024,
    messages: [
      {
        role: 'user',
        content: prompt,
      },
    ],
  });
}
```

## Step 5: Create API Endpoint

Create `src/pages/api/analyze.ts`:

```typescript
import type { NextApiRequest, NextApiResponse } from 'next';
import { analyzeVenture } from '@/lib/claude';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { ventureData } = req.body;

    if (!ventureData) {
      return res.status(400).json({ error: 'Venture data required' });
    }

    const analysis = await analyzeVenture(ventureData);
    res.status(200).json({ analysis });
  } catch (error) {
    console.error('Analysis error:', error);
    res.status(500).json({ error: 'Failed to analyze venture' });
  }
}
```

## Step 6: Test Claude Integration

Create `src/pages/api/test-claude.ts`:

```typescript
import type { NextApiRequest, NextApiResponse } from 'next';
import { analyzeVenture } from '@/lib/claude';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const testVenture = {
    name: 'TechStartup Inc',
    stage: 'Series A',
    industry: 'AI',
    founder: 'John Doe',
    amount: 5000000,
    description: 'Building AI tools for enterprise',
  };

  try {
    const analysis = await analyzeVenture(testVenture);
    res.status(200).json({ analysis });
  } catch (error) {
    res.status(500).json({ error: String(error) });
  }
}
```

Test at: `http://localhost:3000/api/test-claude`

## Step 7: Integrate with Frontend

Example React component:

```typescript
import { useState } from 'react';

export function VentureAnalyzer({ venture }) {
  const [analysis, setAnalysis] = useState('');
  const [loading, setLoading] = useState(false);

  const handleAnalyze = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/analyze', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ ventureData: venture }),
      });
      const data = await response.json();
      setAnalysis(data.analysis);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      <button onClick={handleAnalyze} disabled={loading}>
        {loading ? 'Analyzing...' : 'Analyze Venture'}
      </button>
      {analysis && <p>{analysis}</p>}
    </div>
  );
}
```

## Pricing Estimate

- **Analyze venture**: ~200 tokens = $0.01
- **Market intel**: ~800 tokens = $0.03
- **For 10 analyses/day**: ~$0.40/day = $12/month

## Troubleshooting

**"Invalid API key"**
- Check key starts with `sk-ant-`
- Regenerate at console.anthropic.com

**"Rate limited"**
- Wait 60 seconds
- Check monthly usage at Anthropic Console

**"Context length exceeded"**
- Reduce venture data size
- Use summaries instead of full text

## Next Steps

→ [Real-time Setup](./realtime-setup.md)
