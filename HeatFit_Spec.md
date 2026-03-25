# HeatFit — Product Specification
**Version:** 1.1 (MVP)
**Last Updated:** March 2026
**Build Tool:** Claude Code
**Platform:** Mobile-first responsive web app

---

## Table of Contents
1. [Product Overview](#1-product-overview)
2. [Technical Architecture](#2-technical-architecture)
3. [Authentication & Onboarding](#3-authentication--onboarding)
4. [Dashboard](#4-dashboard)
5. [Weight Tracker](#5-weight-tracker)
6. [Macro & Meal Logging](#6-macro--meal-logging)
7. [Workout & Activity Tracking](#7-workout--activity-tracking)
8. [AI Calorie Burn Estimation](#8-ai-calorie-burn-estimation)
9. [AI Coach (Chat)](#9-ai-coach-chat)
10. [Email Notifications](#10-email-notifications)
11. [UI & Design Guidelines](#11-ui--design-guidelines)
12. [MVP vs V2 Feature Scope](#12-mvp-vs-v2-feature-scope)

---

## 1. Product Overview

**HeatFit** is a mobile-first, responsive web application designed to help users simultaneously manage weight loss goals alongside strength and endurance training. Unlike apps that optimize for one goal at a time, HeatFit balances the tension between caloric deficit and athletic performance through an AI-powered coaching and logging experience.

### Core Value Props
- **Centralized performance view** — one place to see all calorie expenditure, consumption, and weight trend together, eliminating the need to cross-reference multiple apps
- **Honest calorie math** — AI re-estimates calorie burn rather than relying on inflated third-party data
- **Low-friction logging** — describe or photograph a meal and get instant macro estimates
- **Personalized coaching** — an AI coach with selectable personas that knows your full data context
- **Forward-looking intelligence** — the AI coach helps plan ahead for meals, events, and real-life hurdles, not just react to what already happened
- **Unified view** — weight, food, and workouts in one place with one connected goal

### Primary User (MVP)
Solo user / personal tool. Designed for someone actively strength training or doing endurance sport while pursuing a weight loss or body composition goal.

---

## 2. Technical Architecture

### Stack (Recommended for Claude Code)
| Layer | Technology |
|---|---|
| Frontend | React + Tailwind CSS (mobile-first) |
| Backend | Node.js / Next.js API routes |
| Database | PostgreSQL (via Supabase or PlanetScale) |
| Auth | Supabase Auth or Clerk (Google + Apple SSO) |
| AI Engine | Anthropic Claude API (claude-sonnet) |
| Photo Analysis | Claude Vision (claude-sonnet multimodal) |
| Workout Sync | Strava API (OAuth 2.0) |
| Email | Resend or SendGrid |
| Hosting | Vercel |

### AI Integration Points
All AI features run through a single Anthropic Claude API integration:

1. **Macro estimation** — Claude Vision analyzes meal photos + text descriptions to return estimated calories and macros
2. **Calorie burn estimation** — Claude models calorie burn from Strava workout data using research-backed inputs
3. **AI Coach** — Claude powers the chat interface with full user data context passed in each session
4. **Daily insights** — Claude generates a brief daily summary pushed to the dashboard

### Data Flow Summary
```
User Input (food photo/text, weight, workouts)
        ↓
HeatFit Database (PostgreSQL)
        ↓
Claude API (context bundle: goals + recent weight + today's meals + today's workout)
        ↓
AI responses (macro estimates, calorie burn, coaching, insights)
        ↓
Dashboard / Chat UI
```

---

## 3. Authentication & Onboarding

### Sign-Up Flow
- Email/password registration OR single-click SSO via **Google** or **Apple**
- Minimal friction — no required personal info at sign-up beyond email
- After account creation, user is routed to onboarding

### Onboarding Sequence
Onboarding is a step-by-step flow completed once. Progress is saved so users can resume if they exit. Steps:

#### Step 1: Units & Basic Biometrics
Ask the user:
- **Measurement preference:** Imperial (lbs, ft/in) / Metric (kg, cm) — applies across the entire app
- Height
- Current weight
- **Goal weight** (optional) — displayed as a target line on weight charts
- Biological sex (Male / Female) — used for metabolic calculations only
- General activity level outside of structured exercise:
  - Sedentary (desk job, minimal movement)
  - Lightly active (on feet part of the day)
  - Moderately active (physically active job or lifestyle)

#### Step 2: Goals
- **Primary goal:**
  - Lose weight
  - Maintain weight
  - Gain weight / muscle
- **If Lose or Gain:** ask target rate in lbs/week
  - Lose: 0.5 / 1.0 / 1.5 / 2.0 lbs/week
  - Gain: 0.25 / 0.5 / 1.0 lbs/week
- **Secondary goal (multi-select):**
  - Build strength
  - Improve endurance
  - General health
  - Body recomposition
- These goals are stored and passed as context to the AI coach

#### Step 3: Strava Connection
- Prompt: "Connect your Strava account to automatically import workouts and activity data."
- OAuth flow with Strava
- If user skips: workouts can be entered manually (see Section 7)
- Connection can be completed later in Settings

#### Step 4: AI Coach Persona
User selects their preferred coaching personality:

| Persona | Description |
|---|---|
| **Jillian** | Direct, no-nonsense. Pushes you hard. Celebrates wins, calls out excuses. |
| **Michael** | Balanced and real. Honest without being harsh. Encouraging but grounded. |
| **Tiara** | Warm and non-judgmental. Supportive first, challenging gently. |

Each persona is implemented via a distinct system prompt passed to Claude.

#### Step 5: Communication Preferences
- **Response length:** Short / Medium / Long
- **Emoji use:** None / A few / Lots

These preferences are stored and injected into every AI coach system prompt.

#### Step 6: Timezone & Eating Patterns
- **Timezone:** Auto-detected from browser with manual override option
- **Typical meal times** (used to schedule meal reminders intelligently):
  - Breakfast window (e.g., 7am–9am)
  - Lunch window (e.g., 12pm–1pm)
  - Dinner window (e.g., 6pm–8pm)
- These are defaults — the app refines them over time based on actual logging behavior

#### Step 7: Email Notification Preferences
User selects which notifications they want. Each can be toggled On/Off independently:

| Notification | Description | Default Send Time |
|---|---|---|
| **Morning check-in** | Weight log prompt + plan for the day ahead | 7:00am (user's timezone) |
| **Breakfast reminder** | Log your breakfast prompt | 15 min before breakfast window |
| **Lunch reminder** | Log your lunch prompt | 15 min before lunch window |
| **Dinner reminder** | Log your dinner prompt | 15 min before dinner window |
| **Day recap** | End-of-day summary: net calories, macros, coach note | 9:00pm (user's timezone) |
| **Weekly summary** | Week in review email every Sunday evening | Sunday 7:00pm |

All times are displayed and stored in the user's local timezone. User can adjust send times individually in Settings after onboarding.

---

## 4. Dashboard

The Dashboard is the app's home screen — visible immediately after login.

### Layout (Mobile-first)
- Sticky top bar: HeatFit logo + date + avatar/settings icon
- Bottom navigation bar with 5 tabs: Home, Log Food, Weight, Workouts, Chat

### Dashboard Sections

#### Today's Summary Card
- Calories consumed vs. goal
- Macros progress bar (protein / carbs / fat)
- Calorie burn (from AI estimate)
- Net calories
- Daily streak indicator

#### Weight Trend Mini-Chart
- Sparkline showing weight over last 14 days
- Current weight + change from start of week
- **Goal weight line** displayed as a dashed reference line if goal weight was entered during onboarding
- Distance to goal shown below chart (e.g., "12.4 lbs to goal")

#### AI Coach Insight Card
- 2–3 sentence daily insight generated by Claude based on yesterday's data
- "Ask Coach" button that opens Chat
- Refreshes each morning

#### Today's Workout Card
- Pulled from Strava sync or manual entry
- Shows workout type, duration, and AI-estimated calorie burn
- If no workout logged: shows motivational prompt or rest day confirmation

### Graph Views (expandable from dashboard)
- Weight over time: 1W / 1M / 3M / All
- Calories in vs. calories out over time: 1W / 1M
- Macro breakdown over time: 1W / 1M
- Net calorie trend vs. goal: 1W / 1M

---

## 5. Weight Tracker

### Log Weight
- Single input screen: weight (lbs or kg, per user's unit preference)
- Date defaults to today, can be changed for historical logging
- Optional comment field (e.g., "felt bloated," "after vacation")
- Log button — confirmation animation on save
- **AI Coach Feedback:** Immediately after saving, display a 1–3 sentence response from the AI coach in the selected persona's voice. Examples:
  - Trend context: "You're down 0.6 lbs from last week — the trend is moving in the right direction."
  - Plateau awareness: "Weight has been flat for 5 days — that's normal. Stay consistent, the trend will follow."
  - Celebration: "That's your lowest weight in 3 weeks. Keep it going."
- Feedback is generated by Claude with access to the last 14 days of weight entries and the user's goal

### Weight History View
- Visual chart with toggle: 1W / 1M / 3M / All time
- Data points are tappable to show date, weight, and comment
- Trend line overlay showing moving average (smooths daily fluctuation)
- **Goal weight** shown as a persistent dashed line across all chart views
- Summary stats: starting weight, current weight, total change, rate per week, estimated date to reach goal

### Notes
- Users naturally fluctuate 1–3 lbs daily due to water, food, etc. — trend line is the primary KPI, not daily number
- Consider adding a tooltip or coach note that explains this on first use

---

## 6. Macro & Meal Logging

### Log Entry Points
- From bottom nav "Log Food" tab
- Quick-add button on Dashboard summary card

### Meal Categories
Each log entry is categorized as:
- Breakfast
- Lunch
- Dinner
- Snack
- Miscellaneous

### Logging Methods

#### Method 1: Photo Upload
- User uploads or captures a photo of their meal
- Optional: add a text description alongside the photo (strongly recommended for accuracy)
- Claude Vision analyzes the image + description and returns:
  - Identified food items
  - Estimated portion sizes
  - Estimated calories
  - Macros: protein (g), carbs (g), fat (g), fiber (g)
- User sees the AI's estimate with an editable breakdown before confirming
- Edit mode: user can adjust quantities/items before saving
- Disclaimer displayed: "AI estimates may not be exact. Adjust as needed."

#### Method 2: Text Description
- User types a natural language description
  - e.g., "Grilled chicken breast about 6oz, half cup brown rice, side of broccoli with olive oil"
- Claude parses the description and returns estimated macros (same review + edit flow as photo)

#### Method 3: Staple Meals (saved favorites)
- Quick-add from a saved meal library (see below)

### Save as Staple Meal
At the confirmation screen (after AI estimates are reviewed and accepted), the user is presented with:
- **"Save as Staple Meal?"** toggle — on by default for first-time meals, off for repeat logs
- If toggled on: prompt for a name (pre-filled with AI-suggested name, e.g., "Chicken & Rice Bowl")
- Saved immediately to favorites library on confirm
- This replaces the need for a separate "add to favorites" flow — saving happens naturally at the end of every log

### Meal Feedback
After logging and confirming, AI provides a brief 1–3 sentence contextual comment in the coach's persona voice:
- e.g., "Solid lunch — you hit 45g of protein in one meal. You're at 72% of your daily goal already."
- e.g., "That's a high-fat meal for mid-day. You've got room for it today given your morning run, but keep dinner light."
- e.g., "You're 200 calories under pace for the day — make sure dinner keeps you from dipping too low."
- Feedback is generated by Claude with full context of today's meals, workout, and calorie goal

### Staple Meals / Favorites
- Any logged meal can be saved and named (e.g., "My Usual Chicken Bowl")
- Saved meals appear in a "Favorites" quick-access list in the Log Food screen
- Saved meals can be edited: add/remove sub-items, adjust portions
- Tapping a favorite pre-populates the log with saved macros for confirmation

### Daily Macro Overview
- Running tally visible in Log Food screen and Dashboard
- Shows: consumed / goal for calories, protein, carbs, fat
- Goal is dynamically set based on:
  - User's weight loss/gain target
  - Today's AI-estimated calorie burn (updates when workout is logged)
  - Historical calorie intake patterns (rolling 7-day context)

---

## 7. Workout & Activity Tracking

### Primary Source: Strava
- User connects Strava via OAuth during onboarding or in Settings
- HeatFit syncs workouts automatically (daily pull or webhook if available)
- Data pulled per workout:
  - Workout type (run, ride, swim, weight training, etc.)
  - Duration
  - Distance (where applicable)
  - Heart rate data (average + max, if available)
  - Elevation gain (where applicable)
  - Date and time

### Manual Entry (Fallback)
For users without Strava, or for workouts not tracked in Strava:
- Workout type (dropdown: Running, Cycling, Swimming, Weight Training, HIIT, Yoga, Other)
- Duration (minutes)
- Perceived effort: Easy / Moderate / Hard / Max
- Optional: distance, notes
- These inputs are used by the AI calorie burn estimator

### Workout Log View
- List of workouts by week
- Each entry shows: type icon, date, duration, AI calorie burn estimate
- Tap to see full details
- No goal-setting or programming in MVP — HeatFit reads workouts, it does not prescribe them

### AI Feedback on Workout Log
When a workout is synced from Strava or logged manually, the app surfaces a 1–3 sentence AI coach comment on the workout card:
- e.g., "5-mile run at a solid pace. That effort bumps your calorie budget by 380 — use it wisely."
- e.g., "Rest day logged. Recovery is part of the plan — your calorie goal has been adjusted down for today."
- e.g., "Back-to-back strength sessions. Make sure you're hitting your protein target today to support recovery."
- Feedback is generated by Claude with context of workout type, duration, AI calorie burn, and current weekly training load

---

## 8. AI Calorie Burn Estimation

### Problem
Strava and other fitness platforms systematically overestimate calorie burn — sometimes by 20–40%. HeatFit uses its own AI-powered estimation model to produce more accurate figures.

### Inputs Used
| Input | Source |
|---|---|
| Workout type | Strava or manual entry |
| Duration | Strava or manual entry |
| Distance | Strava (if available) |
| Average heart rate | Strava (if available) |
| User body weight | Most recent logged weight |
| User age | Derived from birthdate (if collected) or estimated from onboarding |
| Perceived effort | Manual entry fallback |

### Estimation Approach
Claude is prompted with these inputs and instructed to apply a MET-based calorie estimation model adjusted for:
- Body weight (heavier users burn more)
- Heart rate data where available (correlates to actual effort)
- Workout type efficiency (e.g., running burns more per minute than cycling at equivalent HR)

The model intentionally does not trust the Strava-reported calorie figure as ground truth.

### Output
- AI-estimated calorie burn displayed on Workout card and Dashboard
- Brief note shown to user: "HeatFit's estimate. Strava reported X — we think Y is closer."
- Estimate feeds into net calorie calculation for the day

---

## 9. AI Coach (Chat)

### Overview
An AI-powered coaching chat interface powered by Claude. The coach has full access to the user's data and responds in the voice of the selected persona.

### Context Bundle
Every chat session receives a system prompt containing:
- User profile: name, age, biological sex, height, current weight, goal, target rate
- Weight trend: last 7 days of weight entries
- Today's food log: all meals logged so far today with macro totals
- Today's workout: type, duration, AI calorie burn estimate
- Yesterday's summary: net calories, notable coach notes
- User preferences: persona (Jillian/Michael/Tiara), response length, emoji preference

### Personas (System Prompt Variants)
Each persona is a distinct system prompt style:

**Jillian** — Assertive, direct, data-driven. No sugarcoating. Celebrates wins loudly, calls out patterns honestly. Does not accept excuses but is never cruel.

**Michael** — Balanced and real. Mixes encouragement with accountability. Conversational tone, practical advice, keeps it grounded.

**Tiara** — Warm, supportive, and non-judgmental. Leads with empathy. Challenges the user gently. Focuses on progress over perfection.

### Chat Capabilities
The AI coach can:
- Answer questions about today's meals, workout, and net calories
- Give advice on meal choices relative to goals
- Explain the calorie burn estimate
- Motivate, reframe setbacks, celebrate wins
- Suggest how to adjust the rest of the day (e.g., "You've got 400 calories left — here's what works")
- Explain macro targets and why they're set where they are
- **Plan future meals** — user can describe upcoming meals (e.g., "I'm going out for Italian tonight") and the coach will suggest what to order or how to balance the rest of the day around it
- **Anticipate real-life hurdles** — user can flag upcoming events or situations (e.g., "I have a bachelor party this weekend" or "I'm traveling for work next week") and the coach will help them plan around it: calorie banking strategies, what to eat before an event, how to stay on track without being rigid
- **Weekly look-ahead** — user can ask "what should I focus on this week?" and the coach will analyze recent trends and provide a forward-looking game plan
- **Meal suggestions** — user can ask for meal ideas that fit their remaining macros for the day

The AI coach cannot:
- Provide medical advice or diagnose anything
- Prescribe workout programs (MVP)
- Directly edit logs on the user's behalf (MVP)

### Guardrails
- A disclaimer is shown on first use: "HeatFit's AI coach is not a medical professional. Always consult a doctor before making significant changes to your diet or exercise."
- Claude is instructed in its system prompt to decline medical/injury questions and refer the user to a professional

### Chat UI
- Standard message thread UI
- Persona avatar and name shown at top
- Option to switch persona in chat settings (resets tone, does not lose history)
- Chat history is stored per session; sessions persist across days
- "New conversation" button to start fresh

---

## 10. Email Notifications

All notifications are delivered via email (Resend or SendGrid). No push notifications in MVP. All send times are in the user's local timezone, set during onboarding.

### Notification Types

#### Morning Check-In
- **Default time:** 7:00am
- **Content:** Prompt to log today's weight + ask what workouts are planned for the day
- **Purpose:** Starts the day with a weight entry and pre-sets the calorie burn estimate, so the daily food goal is accurate before the first meal
- **Logic:** If workout is noted or logged, calorie goal updates. If no response, defaults to rest day TDEE

#### Breakfast Reminder
- **Default time:** 15 minutes before user's breakfast window start
- **Content:** "Time to log breakfast. [Log now →]"

#### Lunch Reminder
- **Default time:** 15 minutes before user's lunch window start
- **Content:** "Don't forget to log lunch. [Log now →]"

#### Dinner Reminder
- **Default time:** 15 minutes before user's dinner window start
- **Content:** "Log your dinner to close out the day strong. [Log now →]"

#### Day Recap
- **Default time:** 9:00pm
- **Content:** Brief end-of-day summary:
  - Net calories for the day vs. goal
  - Macro snapshot (protein / carbs / fat)
  - 1–2 sentence coach note in selected persona's voice
  - Link to open app for full view
- Generated by Claude with today's full data context

#### Weekly Summary
- **Default time:** Sunday 7:00pm
- **Content:**
  - Average daily net calories vs. goal for the week
  - Weight change over the week
  - Workouts completed
  - 2–3 sentence coach recap and one actionable focus for next week
  - Motivational note in persona's voice
- Generated by Claude with the full week's data

### Notification Management
- All 6 notification types are individually toggleable — users control exactly what they receive
- Send times for each are customizable in Settings post-onboarding
- Meal reminder times auto-adjust over time based on when the user actually logs (rolling 4-week average)

---

## 11. UI & Design Guidelines

### Mobile-First Priority
Design and develop for 390px viewport width first. Desktop and tablet are secondary breakpoints.

### Navigation
- **Bottom tab bar** (mobile): Home / Log Food / Chat / Weight / Workouts
- **Top nav** (desktop): same tabs in horizontal header
- Bottom tab bar stays fixed on all mobile screens
- Chat is centered in the nav — reflecting its role as the primary interactive feature, not an afterthought

### Design Principles
- Large, thumb-friendly tap targets (minimum 44x44px)
- High contrast text — readability in outdoor / gym lighting
- Minimal chrome — data and actions front and center
- Progress and momentum are always visible (streaks, trend lines, daily progress bars)

### Visual Identity
HeatFit should feel energetic but not overwhelming. Not a clinical health app, not a bro-gym app.
- **Primary palette:** Deep charcoal backgrounds + a heat-toned accent (amber/orange) for energy
- **Typography:** Clean, modern sans-serif — SF Pro (iOS system) or Inter
- **Charts:** Simple line + bar charts. No 3D, no clutter.
- **Personas:** Each coach has a simple avatar/icon and an associated accent color used in chat bubbles

### Key Screens (MVP)
1. Onboarding flow (7 steps)
2. Dashboard / Home
3. Log Food (photo/text/favorites)
4. Weight Tracker
5. Workout Log
6. Chat
7. Settings (profile, goals, Strava connection, preferences, notifications)

---

## 12. MVP vs V2 Feature Scope

### MVP (V1.0)
| Feature | Status |
|---|---|
| Auth (email + Google + Apple SSO) | MVP |
| Onboarding flow (7 steps) | MVP |
| Imperial / metric preference | MVP |
| Goal weight + projected goal date | MVP |
| Timezone detection + meal window setup | MVP |
| Dashboard with today's summary | MVP |
| Weight tracker + trend chart with goal line | MVP |
| AI coach feedback on weight log | MVP |
| Macro logging (text description) | MVP |
| Macro logging (photo upload via Claude Vision) | MVP |
| AI macro estimation (photo + text) | MVP |
| Meal review + edit before saving | MVP |
| Save as staple meal (inline at log confirmation) | MVP |
| Staple meals / favorites quick-add | MVP |
| AI coach feedback on meal log | MVP |
| Strava OAuth sync | MVP |
| Manual workout entry | MVP |
| AI calorie burn estimation | MVP |
| AI coach feedback on workout log | MVP |
| Daily calorie goal (dynamic) | MVP |
| AI Coach chat (Jillian / Michael / Tiara) | MVP |
| Full data context in every chat | MVP |
| Forward-looking meal planning in chat | MVP |
| Hurdle planning in chat (events, travel, etc.) | MVP |
| Email notifications (all 6 types, individually toggleable) | MVP |
| Weekly summary email | MVP |
| Settings (profile, goals, preferences, Strava, notifications) | MVP |
| Mobile-first responsive UI | MVP |

### V2 Roadmap
| Feature | Notes |
|---|---|
| Native iOS app | HealthKit access, native push notifications |
| Apple Health sync | Requires native iOS app |
| Social / accountability layer | Share progress, friends, challenges |
| Water intake tracking | Lightweight add to daily log |
| PR / personal record tracking | For strength training milestones |
| AI workout suggestions | Based on goals + current fitness level |
| Barcode / food database search | Manual fallback for packaged foods |
| Body composition tracking | Body fat %, measurements |
| Garmin / Whoop integration | Expand beyond Strava |
| In-app persona avatars + themes | Visual identity per coach |
| Habit streaks + gamification | Logging streaks, milestones, badges |

---

*HeatFit Spec v1.1 — built with Claude Code — Anthropic Claude API*
