# VaultBanks Telegram Mini App - Design Guidelines

## Design Approach

**Selected Approach:** Design System - Material Design 3 (adapted for Telegram ecosystem)

**Justification:** VaultBanks is a utility-focused earning platform where clarity, trust, and efficiency are paramount. Users need to quickly understand their balance, complete offers, and track earnings. Material Design 3's emphasis on information hierarchy and mobile-first patterns aligns perfectly with Telegram Mini App constraints.

**Key Principles:**
- Trust through transparency (clear point values, payout rates visible)
- Efficiency in navigation (max 2 taps to any action)
- Information density without clutter
- Native Telegram feel with modern polish

---

## Typography System

**Font Family:** System font stack (`-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif`)

**Type Scale:**
- **Display (32px/bold):** Dashboard balance, major numeric values
- **Headline (24px/semibold):** Section headers ("Your Offers", "Withdrawal History")
- **Title (18px/semibold):** Offer cards, modal headers
- **Body (16px/regular):** Primary content, descriptions
- **Caption (14px/regular):** Timestamps, secondary info, geo-tier labels
- **Label (12px/medium):** Badges, status indicators, button text in cards

**Number Display:** Use tabular-nums font-variant for points/USD values to prevent layout shifts

---

## Layout & Spacing System

**Tailwind Spacing Units:** Standardize on 2, 4, 6, 8, 12, 16, 20 (e.g., p-4, m-8, gap-6)

**Container Structure:**
- Max-width: `max-w-2xl` (672px) - optimized for mobile/small tablets
- Horizontal padding: `px-4` (mobile), `px-6` (tablet+)
- Vertical section spacing: `py-8` between major sections

**Card Spacing:**
- Internal padding: `p-4` for compact cards, `p-6` for detailed views
- Gap between cards: `gap-4` in grids/lists

**Safe Areas:** Add `pb-safe` for Telegram's bottom UI elements, `pt-safe` for status bar

---

## Component Library

### Dashboard Header
- Full-width hero card with current balance
- Structure: Large points display (Display typography) + USD equivalent below (Caption)
- Quick stats row: Lifetime leads, Current tier, Last reward time
- Spacing: `p-6`, rounded-2xl, shadow-lg

### Offer Wall Cards (Grid)
- Layout: `grid grid-cols-2 gap-4` on mobile, `grid-cols-3` on wider screens
- Each card: `aspect-square` or `aspect-[4/3]`, rounded-xl, shadow-md
- Content hierarchy: Offer title (Title), Payout preview (Body), "Start" CTA (Label)
- Hover state: Subtle scale transform (scale-105), increased shadow

### Navigation
- Bottom tab bar (sticky): 4 icons - Home, Offers, History, Withdraw
- Active state: filled icon variant
- Spacing: `h-16`, icons at 24px, label below at 12px

### Withdrawal Form
- Single-column layout: max-w-md centered
- Input fields: `h-12`, rounded-lg, clear label above (Caption)
- Bank info: Expandable accordion pattern
- Geo-tier display: Prominent conversion rate badge (e.g., "Tier 1: 2000 pts = $20")

### Transaction History
- List layout: Each item in rounded-lg card with `p-4`
- Row structure: Icon (left) | Details (center, flex-1) | Amount (right, tabular)
- Status badges: Pill-shaped, 6px vertical padding, 12px horizontal
- Timestamps: Caption typography, muted

### Ad Reward Buttons
- "Watch Ad" primary: Full-width or prominent placement, `h-12`, rounded-full
- "Visit & Earn" popup: Secondary style, `h-10`, with icon prefix
- Cooldown state: Disabled appearance with countdown timer (Caption)

### Modals/Overlays
- Full-screen on mobile (slides up from bottom)
- Rounded top corners (rounded-t-2xl) with drag handle
- Close button: top-right, 40x40 tap target
- Content padding: `p-6`

### Status Indicators
- Point additions: Toast notification (bottom), slides up, auto-dismiss 3s
- Loading states: Skeleton screens matching card layouts
- Empty states: Centered icon (96px) + message + CTA

---

## Animations

**Minimal Usage - Strategic Only:**
- Page transitions: Fade (200ms ease-out)
- Card interactions: Scale on tap (100ms)
- Point increment: CountUp animation (1s) for balance updates
- Toast notifications: Slide up + fade in (300ms)

**No Animations:**
- Scrolling parallax
- Decorative micro-interactions
- Background animations

---

## Images

**Hero Section:** No traditional hero image. Replace with data visualization - dashboard card showing points balance with subtle gradient background pattern.

**Offer Cards:** Each offer card includes a placeholder thumbnail (16:9 ratio) showing offer category/type. Use simple icon-based illustrations, not photos.

**Trust Signals:** Small badge icons for geo-tiers (flag icons, 20px), payment method logos (bank icons, 24px) in withdrawal section.

**Empty States:** Minimalist line-art illustrations (single accent outline, no fills) at 120x120px for "No offers completed yet" states.

---

## Accessibility & Polish

- All interactive elements: min-height 44px tap targets
- Focus states: 2px outline offset for keyboard navigation
- Contrast: Ensure text meets WCAG AA (4.5:1) against backgrounds
- Loading skeletons: Shimmer effect matching component shapes
- Error states: Inline validation with icon + message below inputs
- Success confirmations: Full-screen checkmark animation for withdrawal submission