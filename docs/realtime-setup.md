# Real-time Data Sync Setup

## Overview

Enable live data updates across both users using Supabase WebSocket.

**Setup time: ~15 minutes**

## Step 1: Create Supabase Project

1. Go to [Supabase](https://supabase.com)
2. Click **"New Project"**
3. Database password: Generate secure one
4. Region: Choose closest to you
5. Click **"Create new project"** (takes ~2 min)

## Step 2: Get API Keys

1. Go to **Settings** → **API**
2. Copy these values:
   - `Project URL` → `NEXT_PUBLIC_SUPABASE_URL`
   - `anon (public)` key → `NEXT_PUBLIC_SUPABASE_ANON_KEY`
   - `service_role` key → `SUPABASE_SERVICE_KEY`

## Step 3: Update Environment

In `.env.local`:

```env
NEXT_PUBLIC_SUPABASE_URL=https://xxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGc...
SUPABASE_SERVICE_KEY=eyJhbGc...
```

## Step 4: Create Tables

In Supabase, go to **SQL Editor** and run:

```sql
-- Ventures table for real-time sync
CREATE TABLE public.ventures (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  stage TEXT NOT NULL,
  industry TEXT,
  founder TEXT,
  amount DECIMAL(12, 2),
  status TEXT DEFAULT 'Active',
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Enable realtime
ALTER PUBLICATION supabase_realtime ADD TABLE ventures;

-- Activity log
CREATE TABLE public.activity (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_email TEXT NOT NULL,
  action TEXT NOT NULL,
  venture_id TEXT,
  timestamp TIMESTAMP DEFAULT NOW()
);

ALTER PUBLICATION supabase_realtime ADD TABLE activity;
```

## Step 5: Create Supabase Client

Create `src/lib/supabase.ts`:

```typescript
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!;
const supabaseKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient(supabaseUrl, supabaseKey);

// Subscribe to venture changes
export function subscribeToVentures(callback: (data: any) => void) {
  return supabase
    .channel('ventures-changes')
    .on(
      'postgres_changes',
      {
        event: '*',
        schema: 'public',
        table: 'ventures',
      },
      (payload) => {
        console.log('Change received!', payload);
        callback(payload);
      }
    )
    .subscribe();
}

// Log activity
export async function logActivity(
  userEmail: string,
  action: string,
  ventureId?: string
) {
  const { error } = await supabase.from('activity').insert([
    {
      user_email: userEmail,
      action,
      venture_id: ventureId,
    },
  ]);

  if (error) console.error('Activity log error:', error);
}
```

## Step 6: React Hook for Real-time

Create `src/hooks/useRealtimeVentures.ts`:

```typescript
import { useEffect, useState } from 'react';
import { subscribeToVentures } from '@/lib/supabase';

export function useRealtimeVentures() {
  const [ventures, setVentures] = useState<any[]>([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    // Subscribe to changes
    const subscription = subscribeToVentures((payload) => {
      if (payload.eventType === 'INSERT') {
        setVentures((prev) => [...prev, payload.new]);
      } else if (payload.eventType === 'UPDATE') {
        setVentures((prev) =>
          prev.map((v) => (v.id === payload.new.id ? payload.new : v))
        );
      } else if (payload.eventType === 'DELETE') {
        setVentures((prev) => prev.filter((v) => v.id !== payload.old.id));
      }
    });

    setIsLoading(false);

    return () => {
      subscription?.unsubscribe();
    };
  }, []);

  return { ventures, isLoading };
}
```

## Step 7: Use in Component

```typescript
import { useRealtimeVentures } from '@/hooks/useRealtimeVentures';

export function VenturesList() {
  const { ventures, isLoading } = useRealtimeVentures();

  if (isLoading) return <div>Loading...</div>;

  return (
    <div>
      <h2>Ventures (Real-time)</h2>
      {ventures.map((v) => (
        <div key={v.id}>
          <h3>{v.name}</h3>
          <p>Stage: {v.stage}</p>
          <p>Updated: {new Date(v.updated_at).toLocaleString()}</p>
        </div>
      ))}
    </div>
  );
}
```

## Step 8: Set RLS (Row-Level Security)

In Supabase SQL Editor:

```sql
-- Enable RLS
ALTER TABLE ventures ENABLE ROW LEVEL SECURITY;
ALTER TABLE activity ENABLE ROW LEVEL SECURITY;

-- Allow authenticated users to read/write
CREATE POLICY "Allow authenticated users" ON ventures
  FOR SELECT
  USING (auth.role() = 'authenticated');

CREATE POLICY "Allow authenticated users insert" ON ventures
  FOR INSERT
  WITH CHECK (auth.role() = 'authenticated');
```

## Webhook for Airtable Sync

When Airtable changes, sync to Supabase:

Create `src/pages/api/webhooks/airtable.ts`:

```typescript
import type { NextApiRequest, NextApiResponse } from 'next';
import { supabase } from '@/lib/supabase';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).end();
  }

  try {
    const { records } = req.body;

    // Sync each record to Supabase
    for (const record of records) {
      await supabase.from('ventures').upsert([
        {
          id: record.id,
          name: record.fields.Name,
          stage: record.fields.Stage,
          industry: record.fields.Industry,
          founder: record.fields.Founder,
          amount: record.fields['Investment Amount'],
          status: record.fields.Status,
        },
      ]);
    }

    res.status(200).json({ success: true });
  } catch (error) {
    console.error('Webhook error:', error);
    res.status(500).json({ error: 'Sync failed' });
  }
}
```

## Troubleshooting

**"Connection refused"**
- Check Supabase URL is correct
- Verify project is active

**"Realtime not working"**
- Check `ALTER PUBLICATION supabase_realtime` was run
- Restart Next.js: `npm run dev`

**"RLS policy error"**
- Ensure policies are created correctly
- Check user is authenticated

## Next Steps

→ [Deployment](./deployment.md)
