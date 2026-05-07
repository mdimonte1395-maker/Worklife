# Worklife VC Hub

**Enterprise Venture Capital Database Platform**

A production-ready platform for managing VC portfolio companies with real-time data sync, AI-powered analysis, and secure 2-user authentication.

## Features

✅ **Secure Authentication** - Firebase for 2-user access  
✅ **Airtable Integration** - Your existing data backend  
✅ **Real-time Sync** - Supabase WebSocket updates  
✅ **AI Analysis** - Claude API for venture insights  
✅ **RSS Feed Processing** - Automated market intelligence  
✅ **Production Ready** - Deploy to Vercel in minutes  

## Quick Start

### 1. Clone & Install

```bash
cd Worklife
npm install
cp .env.example .env.local
```

### 2. Get API Keys (15 min total)

- **Firebase**: https://console.firebase.google.com (Create project → Authentication → Enable Email/Password)
- **Airtable**: Already have it → Get API key from https://airtable.com/api
- **Claude**: https://console.anthropic.com (Get API key, set $50/mo limit)
- **Supabase**: https://supabase.com (Create project, get URL + keys)

### 3. Configure Environment

Fill in `.env.local` with all API keys from above.

### 4. Start Development

```bash
npm run dev
# Open http://localhost:3000
```

### 5. Deploy to Vercel

```bash
npm install -g vercel
vercel --prod
```

## Architecture

```
┌─────────────────┐
│   Next.js App   │
│  (Vercel Host)  │
└────────┬────────┘
         │
    ┌────┴────────────────────────────┐
    │                                  │
    v                                  v
┌──────────────┐              ┌──────────────┐
│   Firebase   │              │ Supabase     │
│ (Auth + DB)  │              │ (Real-time)  │
└──────┬───────┘              └──────┬───────┘
       │                             │
       └──────────────┬──────────────┘
                      v
          ┌──────────────────────┐
          │     Airtable Base    │
          │ (Your Data Backend)  │
          └──────────────────────┘
                      │
       ┌──────────────┼──────────────┐
       v              v              v
  ┌────────┐   ┌────────┐   ┌──────────┐
  │ Claude │   │  APIs  │   │ RSS/News │
  │  (AI)  │   │        │   │          │
  └────────┘   └────────┘   └──────────┘
```

## Setup Guides

1. **[Firebase Setup](./docs/firebase-setup.md)** - 2-user authentication
2. **[Airtable Integration](./docs/airtable-integration.md)** - Data sync
3. **[Claude API](./docs/claude-api.md)** - AI analysis
4. **[Real-time Setup](./docs/realtime-setup.md)** - WebSocket updates
5. **[Deployment](./docs/deployment.md)** - Production checklist

## Project Structure

```
Worklife/
├── src/
│   ├── pages/
│   │   ├── api/           # API endpoints
│   │   ├── auth/          # Auth pages
│   │   └── dashboard/     # Main app
│   ├── components/        # React components
│   ├── lib/
│   │   ├── firebase.ts    # Firebase config
│   │   ├── airtable.ts    # Airtable client
│   │   ├── claude.ts      # Claude API
│   │   └── supabase.ts    # Real-time sync
│   └── types/             # TypeScript types
├── docs/                  # Setup guides
├── .env.example          # Environment template
├── package.json          # Dependencies
└── README.md             # This file
```

## Cost Breakdown (Monthly)

| Service | Cost | Notes |
|---------|------|-------|
| Firebase | $0 | Free tier (2 users) |
| Airtable | $10-20 | Your current plan |
| Claude API | $20-50 | Usage-based |
| Supabase | $0 | Free tier real-time |
| Vercel | $0-20 | Free tier or Pro |
| **Total** | **~$30-90** | |

## Implementation Timeline

**Week 1**: Setup & Authentication (2 days)  
**Week 2**: Data Integration (2 days)  
**Week 3**: Deployment & Launch (1-2 days)  

**Total: 5-6 working days**

## API Endpoints

### Authentication
- `POST /api/auth/login` - Login with email/password
- `POST /api/auth/logout` - Logout user
- `GET /api/auth/me` - Get current user

### Data Management
- `GET /api/ventures` - List all ventures
- `POST /api/ventures` - Create venture
- `GET /api/ventures/:id` - Get venture details
- `PUT /api/ventures/:id` - Update venture
- `DELETE /api/ventures/:id` - Delete venture

### AI Analysis
- `POST /api/analyze` - Get Claude analysis on venture
- `POST /api/market-intel` - Market intelligence from RSS feeds

### Real-time
- WebSocket: `wss://your-domain.com/api/realtime`

## Security

✅ Environment variables for all secrets  
✅ Firebase security rules (2-user only)  
✅ CORS configured for Airtable  
✅ Rate limiting on API endpoints  
✅ Encrypted API keys in production  

## Support

For setup issues, check the specific guides in `/docs` folder. Each has detailed troubleshooting.

## License

Private - For internal use only
