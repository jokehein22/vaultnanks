# VaultBanks - Web Application

## Overview
VaultBanks is a standalone web application that allows users to earn points by completing CPA offers, watching ads, and visiting content lockers. Points can be converted to USD based on a geo-tiered payout system and withdrawn to bank accounts via Wise API integration.

## Tech Stack
- **Frontend**: React + TypeScript + Vite + TailwindCSS + Shadcn UI
- **Backend**: Node.js + Express + TypeScript
- **Database**: PostgreSQL (Neon) with Drizzle ORM
- **Routing**: Wouter (frontend), Express (backend)
- **State Management**: TanStack Query
- **Authentication**: Email/password with scrypt hashing, express-session
- **Payment Processing**: Wise API integration

## Project Structure
```
├── client/
│   ├── src/
│   │   ├── components/     # React components
│   │   │   ├── Dashboard.tsx      # Main balance & stats display
│   │   │   ├── OfferWall.tsx      # Offers grid + ad buttons
│   │   │   ├── WithdrawForm.tsx   # Withdrawal form
│   │   │   ├── History.tsx        # Earnings & withdrawals history
│   │   │   ├── Referral.tsx       # Referral system UI
│   │   │   └── Navigation.tsx     # Bottom tab bar
│   │   ├── pages/
│   │   │   ├── Home.tsx           # Main app page (protected)
│   │   │   ├── Login.tsx          # Login page
│   │   │   ├── Signup.tsx         # Registration page
│   │   │   ├── Admin.tsx          # Admin panel (6 tabs)
│   │   │   └── not-found.tsx
│   │   ├── lib/
│   │   │   └── queryClient.ts
│   │   └── hooks/
│   └── index.html
├── server/
│   ├── index.ts            # Express entry point with session middleware
│   ├── routes.ts           # API routes
│   ├── auth.ts             # Authentication helpers (hash, compare, middleware)
│   ├── storage.ts          # Database operations
│   ├── db.ts               # Drizzle/Neon connection
│   └── wise.ts             # Wise API integration
└── shared/
    └── schema.ts           # Drizzle models + geo payout config
```

## Authentication System
- **Email/Password Authentication**: Users register and login with email and password
- **Password Hashing**: scrypt with 64-byte key length and random salt
- **Session Management**: express-session with 7-day cookie expiry
- **Protected Routes**: Frontend redirects unauthenticated users to /login
- **Auth Middleware**: `requireAuth` checks session.userId for API protection

### Auth Endpoints
- `POST /api/auth/register` - Create new account with email, password, firstName
- `POST /api/auth/login` - Login with email and password
- `POST /api/auth/logout` - Destroy session
- `GET /api/auth/session` - Check authentication status

## Key Features
1. **User Profile & Balance** - Dashboard showing points, USD value, tier, and stats
2. **Offer Wall** - 10 CPA offers with click tracking and smart links
3. **Ad Rewards** - Watch Ad (+20 pts) and Visit & Earn popup (+30 pts)
4. **Steps Progression System** - Gamified 10-step daily flow with SmartLinks (see below)
5. **Geo-Based Payouts**:
   - Tier 1 (US, CA, GB, AU, etc.): 2000 pts = $20
   - Tier 2 (EU, LATAM, etc.): 2000 pts = $10
   - Tier 3 (CN, IN, and rest of world): 2000 pts = $2.50
6. **Withdrawal System** - Minimum 2000 points, bank info collection
7. **CPA Postback** - `/api/postback` endpoint for network callbacks
8. **Referral System** - 10% bonus on referee earnings, signup bonuses
9. **Multi-CPA Network Support** - Network-specific secrets, payout multipliers
10. **Wise API Integration** - Automated withdrawal processing via Wise

## Steps Progression System
A gamified daily earning system with sequential unlocking and SmartLink integration:

### Features
- **Daily Limit**: 10 steps per day, resets at midnight UTC
- **Sequential Unlocking**: Must complete step N before accessing step N+1
- **Timer System**: 20-40 second random timer per step
- **SmartLink Integration**: Each step has a unique SmartLink URL that opens in new tab
- **Postback Rewards**: CPA networks can send rewards via postback with step_ prefix click IDs

### SmartLink URLs (configured in server/routes.ts)
| Step | SmartLink URL |
|------|---------------|
| 1 | https://smrturl.co/8df5fc0 |
| 2 | https://smrturl.co/a2b57e1 |
| 3 | https://smrturl.co/e469d32 |
| 4 | https://smrturl.co/21d6fe2 |
| 5 | https://smrturl.co/64033c6 |
| 6 | https://smrturl.co/32ed72a |
| 7 | https://smrturl.co/b463ccd |
| 8 | https://smrturl.co/2c9b751 |
| 9 | https://smrturl.co/454fb74 |
| 10 | https://smrturl.co/775c2d0 |

### Step Statuses
- `locked` - Previous steps not completed
- `available` - Ready to start
- `pending` - SmartLink opened, timer running
- `timer_complete` - Timer finished, awaiting postback
- `rewarded` - Postback received, points awarded

### API Endpoints
- `GET /api/steps/progress` - Get daily progress with all step statuses
- `POST /api/open_step` - Start a step (opens SmartLink, starts timer)
- `POST /api/steps/complete-timer` - Complete timer and advance to next step

### Security
- Server-side timer validation (with 5-second grace period)
- Unique click_id generation with step_ prefix for postback tracking
- Duplicate postback prevention per click_id
- Sequential step enforcement on backend

## Security Features
- **Postback Duplicate Prevention**: Each click_id can only be converted once
- **Postback Cooldown**: 60-second cooldown between leads
- **Popup Reward Protection**: 
  - ymid format validation (must include user's ID prefix)
  - Server-side 30-second rate limiting between popup rewards
  - Daily limit of 40 popups per user
  - Unique ymid constraint to prevent replay attacks
- **Geo Detection**: Country detection with fallback tracking
  - Priority: Cloudflare headers → Browser locale → Proxy headers → default TIER_3
  - Dashboard shows country code or "Auto (Default rate)" when undetected
- **Session Auth**: Secure httpOnly cookies with scrypt password hashing
- **Wise Refund Protection**: Atomic transaction-based refund with SQL increment

## Withdrawal Statuses
- `pending` - Newly submitted, awaiting admin review
- `approved` - Admin approved, ready for Wise processing
- `rejected` - Admin rejected the withdrawal
- `processing` - Wise transfer in progress
- `wise_pending` - Transfer funded, waiting for Wise payout
- `wise_failed` - Wise transfer failed
- `completed` - Successfully paid out
- `refunded` - Cancelled and points restored

## API Endpoints
### Auth
- `POST /api/auth/register` - Create account
- `POST /api/auth/login` - Login
- `POST /api/auth/logout` - Logout
- `GET /api/auth/session` - Check session status

### User (requires auth)
- `GET /api/me` - Get user profile
- `GET /api/offers` - List active offers
- `POST /api/click` - Track offer click
- `GET /api/leads` - User's lead history
- `GET /api/withdrawals` - User's withdrawal history
- `POST /api/withdraw` - Submit withdrawal request
- `POST /api/rewarded-popup` - Claim popup reward (+30 pts)
- `POST /api/inapp-reward` - Claim ad reward (+20 pts)
- `GET /api/referral/stats` - Get referral statistics
- `POST /api/referral/apply` - Apply a referral code

### Public
- `POST /api/postback` - CPA network postback

### Admin
- `POST /api/admin/seed-offers` - Seed default offers
- `GET /api/admin/wise/status` - Wise API status
- `POST /api/admin/wise/process/:id` - Process withdrawal via Wise
- And more Wise-related endpoints...

## Environment Variables
- `DATABASE_URL` - PostgreSQL connection string
- `SESSION_SECRET` - Secret for session encryption
- `POSTBACK_SECRET` - Secret for CPA postback validation (optional)
- `WISE_API_TOKEN` - Wise API token for payment processing
- `WISE_PROFILE_ID` - Wise business profile ID
- `WISE_API_URL` - Wise API URL (defaults to sandbox)

## Database Tables
- `users` - Users with email, passwordHash, points balance, referral codes
- `offers` - CPA offers (10 seeded), with network assignment
- `clicks` - Click tracking with click_id
- `leads` - Completed offers (CPA postbacks)
- `withdrawals` - Withdrawal requests with Wise tracking
- `rewarded_popups` - Popup ad rewards
- `inapp_rewards` - In-app ad session rewards
- `cpa_networks` - Multi-network configuration
- `referrals` - Referral tracking
- `stepsRuns` - Daily steps progress (userId, runDateUtc, currentStep, completedSteps)
- `stepClicks` - Individual step attempts (runId, stepNumber, clickId, status, timerEndAt)

## Development
- Run `npm run dev` to start the app
- Run `npm run db:push` to push schema changes
- Offers are seeded via `POST /api/admin/seed-offers`

## Design
- Dark "vault" cozy theme
- Neon accent colors: Blue (#3B82F6), Purple (#A855F7), Soft Green (#10B981)
- Animated gradient background with slow color shifts
- VB circular badge logo with neon gradient
- Soft cards with rounded corners and subtle glow
- Mobile-first responsive design
- Bottom navigation with 6 tabs (Home, Steps, Offers, Invite, History, Wallet)

### UI Components
- **Dashboard**: VB badge logo, personalized greeting with wave emoji, points balance, payout rate info
- **Offer Wall**: Quick Rewards section (Watch Ad +20pts, Visit & Earn +30pts), CPA offers grid
- **Withdraw Form**: Full bank details, security notices, masked info storage
- **History**: Friendly time format, color-coded status badges
- **Referral**: Share code functionality, earnings stats, referral list
- **Login/Signup**: Clean forms with email/password fields, referral code support
