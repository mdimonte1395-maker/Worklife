# Airtable Integration Setup

## Overview

Sync your Airtable base with your Next.js app for real-time venture data.

**Setup time: ~20 minutes**

## Step 1: Get Airtable API Key

1. Go to [Airtable API](https://airtable.com/api)
2. Click your account (top right)
3. Go to **Personal access tokens**
4. Click **"Create token"**
5. Give it a name: `worklife-vc-hub`
6. Grant scopes:
   - `data.records:read`
   - `data.records:write`
   - `schema.bases:read`
7. Copy the token (save safely)

## Step 2: Find Your Base ID

1. Open your Airtable base in browser
2. Click **Share** (top right)
3. Click **"Copy link"**
4. URL will look like: `https://airtable.com/appXXXXXXXXXXXXXX/tblYYYYYYYYYYYYYY/...`
5. The `appXXXXXXXXXXXXXX` part is your **Base ID**

## Step 3: Get Table Names

In your Airtable base:

1. Check the table names at the bottom
2. Common tables for VC:
   - `Ventures` - Main portfolio companies
   - `Companies` - Detailed company info
   - `Deals` - Transaction history
   - `News` - Market intelligence

## Step 4: Update Environment Variables

In `.env.local`, add:

```env
AIRTABLE_API_KEY=patXXXXXXXXXXXXXX
AIRTABLE_BASE_ID=appXXXXXXXXXXXXXX
AIRTABLE_VENTURES_TABLE=Ventures
AIRTABLE_COMPANIES_TABLE=Companies
```

## Step 5: Create Airtable Client

Create `src/lib/airtable.ts`:

```typescript
import Airtable from 'airtable';

const airtable = new Airtable({
  apiKey: process.env.AIRTABLE_API_KEY,
});

const base = airtable.base(process.env.AIRTABLE_BASE_ID!);

export const venturesTable = base(process.env.AIRTABLE_VENTURES_TABLE!);
export const companiesTable = base(process.env.AIRTABLE_COMPANIES_TABLE!);

// Fetch all ventures
export async function getAllVentures() {
  try {
    const records = await venturesTable
      .select({
        pageSize: 100,
        sort: [{ field: 'Created', direction: 'desc' }],
      })
      .all();

    return records.map((record) => ({
      id: record.id,
      ...record.fields,
    }));
  } catch (error) {
    console.error('Error fetching ventures:', error);
    throw error;
  }
}

// Create venture
export async function createVenture(fields: any) {
  try {
    const record = await venturesTable.create([
      {
        fields,
      },
    ]);
    return record;
  } catch (error) {
    console.error('Error creating venture:', error);
    throw error;
  }
}

// Update venture
export async function updateVenture(recordId: string, fields: any) {
  try {
    const record = await venturesTable.update(recordId, fields);
    return record;
  } catch (error) {
    console.error('Error updating venture:', error);
    throw error;
  }
}

// Delete venture
export async function deleteVenture(recordId: string) {
  try {
    await venturesTable.destroy(recordId);
    return true;
  } catch (error) {
    console.error('Error deleting venture:', error);
    throw error;
  }
}
```

## Step 6: Create API Endpoint

Create `src/pages/api/ventures.ts`:

```typescript
import type { NextApiRequest, NextApiResponse } from 'next';
import { getAllVentures } from '@/lib/airtable';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'GET') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const ventures = await getAllVentures();
    res.status(200).json(ventures);
  } catch (error) {
    console.error('API error:', error);
    res.status(500).json({ error: 'Failed to fetch ventures' });
  }
}
```

## Step 7: Test Connection

Run this command to test:

```bash
npm run dev
```

Then visit: `http://localhost:3000/api/ventures`

You should see your Airtable data in JSON format.

## Step 8: Real-time Sync (Optional)

For real-time updates, set up Airtable webhooks:

1. Go to [Airtable Automations](https://airtable.com/apps)
2. Create automation: "When record is created/modified"
3. Action: "Webhook"
4. URL: `https://your-domain.com/api/webhooks/airtable`
5. Test webhook

## Expected Airtable Structure

```
Ventures Table:
- Name (text)
- Stage (single select: Seed, Series A, Series B, etc.)
- Founder (text)
- Industry (single select: SaaS, FinTech, AI, etc.)
- Investment Amount (currency)
- Deal Date (date)
- Status (single select: Active, Exited, Failed)
- Notes (long text)
- Last Updated (date)

Companies Table:
- Company Name (text)
- Website (url)
- Founded (date)
- Team Size (number)
- Funding Total (currency)
- CEO (text)
- Description (long text)
```

## Troubleshooting

**"Invalid API key"**
- Check `.env.local` has correct key
- Regenerate token at airtable.com/api

**"Base not found"**
- Verify Base ID is correct
- Check it's in the URL: `airtable.com/appXXXXX`

**"Table not found"**
- Make sure table name matches exactly (case-sensitive)
- Check spelling in Airtable base

**"CORS errors"**
- These are normal for browser requests
- Always fetch through Next.js API (`/api/ventures`)

## Next Steps

→ [Claude API Integration](./claude-api.md)
