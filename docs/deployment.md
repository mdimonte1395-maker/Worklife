# Production Deployment Checklist

## Pre-deployment

- [ ] All environment variables set in production
- [ ] Firebase security rules reviewed
- [ ] Airtable API key is read-only or limited
- [ ] Claude API spending limit set to $50/mo
- [ ] Database backups configured
- [ ] All 3rd party keys stored securely

## Deploy to Vercel

### Step 1: Connect GitHub

1. Go to [Vercel](https://vercel.com)
2. Click "New Project"
3. Connect GitHub account
4. Select `mdimonte1395-maker/Worklife` repository
5. Click "Import"

### Step 2: Environment Variables

1. In Vercel, go to **Settings** → **Environment Variables**
2. Add all variables from `.env.local`:
   - `NEXT_PUBLIC_*` (public)
   - `FIREBASE_*` (secret)
   - `AIRTABLE_*` (secret)
   - `CLAUDE_*` (secret)
   - `SUPABASE_*` (secret)

### Step 3: Build Settings

- Framework: **Next.js**
- Build Command: `npm run build`
- Output Directory: `.next`

### Step 4: Deploy

1. Click **Deploy**
2. Wait for build to complete (~2-3 min)
3. Get production URL: `https://worklife-vc-hub.vercel.app`

## Custom Domain (Optional)

1. In Vercel, go to **Settings** → **Domains**
2. Add your domain (e.g., `vc.yourdomain.com`)
3. Update DNS records at your registrar
4. Wait for DNS propagation (~24 hours)

## Post-deployment

### Security

- [ ] Enable HTTPS (Vercel default)
- [ ] Set secure cookies (Firebase handles)
- [ ] Configure CORS for API endpoints
- [ ] Enable rate limiting

### Monitoring

1. Set up error tracking:
   - Vercel Analytics (built-in)
   - Sentry (optional)

2. Monitor performance:
   - Vercel Insights
   - Google Analytics

3. Database monitoring:
   - Supabase dashboard
   - Airtable activity log

### Scaling

For 2 users, Vercel free tier is sufficient:
- ✅ Unlimited deployments
- ✅ Serverless functions
- ✅ Auto-scaling
- ✅ Edge caching

## Update Production

Each push to `main` branch auto-deploys:

```bash
git add .
git commit -m "Feature: Add new capability"
git push origin main
# Automatically deploys to Vercel
```

## Rollback

If deployment fails:

1. Go to Vercel Dashboard
2. Find previous deployment
3. Click "Promote to Production"

## Database Backups

### Airtable
- Automatic daily backups
- Accessible via Airtable settings

### Supabase
1. Go to Supabase Dashboard
2. Settings → Backups
3. Enable automatic backups (free: daily)

## Performance Tips

1. **Image Optimization**: Use Next.js `Image` component
2. **Code Splitting**: Next.js automatic
3. **Caching**: Enable ISR (Incremental Static Regeneration)
4. **CDN**: Vercel Edge Network included

## Monitoring Production

### Daily Checks
- [ ] No error notifications
- [ ] All API endpoints responding
- [ ] Real-time sync working
- [ ] Users can login

### Weekly Checks
- [ ] Performance metrics normal
- [ ] No suspicious API calls
- [ ] Database size reasonable

## Support URLs

- **Vercel Support**: https://vercel.com/support
- **Firebase Console**: https://console.firebase.google.com
- **Supabase Dashboard**: https://app.supabase.com
- **Airtable Help**: https://support.airtable.com

## Final Checklist

- [ ] Production URL working
- [ ] Login with both users successful
- [ ] Data displaying from Airtable
- [ ] Real-time updates working
- [ ] Claude analysis endpoint responding
- [ ] No console errors
- [ ] Performance acceptable (< 2s load)
- [ ] Mobile responsive

**You're live! 🎉**
