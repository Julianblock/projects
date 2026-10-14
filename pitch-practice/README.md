# Pitch Practice

AI role-play training for retail sales teams: reps rehearse customer conversations out loud with AI shoppers, get scored against their company's playbook, and managers coach from the results.

**Client:** Guitar Center  
**Role:** Full-stack and mobile engineer  
**Type:** B2B SaaS for retail sales teams  
**Period:** 2026  
**Status:** Web app in production; iOS app in internal TestFlight testing  
**Source:** Private (client work)

<p align="center">
  <img src="./marketing-home-hero.jpg" alt="Pitch Practice marketing site" width="800"/>
</p>

## Overview

Retail floor sales skills are usually learned on real customers, which costs sales. Role-play with a manager is rare and inconsistent, product and policy sheets get read once, and managers can't tell who is ready until sales numbers come in weeks later.

Pitch Practice was built for Guitar Center, which owns the product. It is designed for multi-store retail companies (music stores, home improvement, electronics, car dealerships, specialty retail) with reps on the floor, store managers coaching them, and owners who want to know which stores and people are ready.

How it works:

1. A company sets up its workspace: name, industry, logo (the brand palette is sampled from it), stores, and a team code. Industry starter scenarios, AI-drafted scenarios, and the company's own product and policy documents feed the AI customers.
2. A rep picks a scenario, for example "First electric guitar for a teenager" with a customer named Dana, and talks to her by live voice or text. Each AI customer has a persona, a mood, hidden objections, and a matching voice.
3. Every conversation is scored on a seven-skill rubric (discovery, product knowledge, value, objection handling, closing, rapport, policy compliance) plus any custom skills the company defines. The rep gets a written review with strong moments, missed chances, and coach notes.
4. Managers see an expertise heatmap, a store map, leaderboards, and conversation transcripts they can annotate. They assign multi-step training paths that reach reps by push notification and email.
5. Weekly goals, streaks, milestones, and rankings run on the web and in the iOS app.

## What I built

- Real-time voice role-play on the OpenAI Realtime API over WebRTC, with a server-side SDP proxy so the API key never reaches the browser. Reps choose hold-to-talk (mic button or Space bar) or hands-free voice detection; calls have live transcripts, a 6-minute limit, and an idle check.
- The AI customer engine: personas with traits, mood, goals, hidden information, and a gender-matched voice, grounded in the company's documents through retrieval (pgvector with HNSW). Scoring uses structured outputs and blends the seven-skill rubric with weighted custom skills.
- The manager dashboard: KPIs, an expertise heatmap (rep by category), a Leaflet store map with insight cards, leaderboards, conversation review with coach notes pinned to turns, and assignments with one-click nudges.
- White-label theming: a brand color, picked or sampled from the logo, is converted to OKLCH and expanded into a full palette with contrast checks. Web, email, and mobile all re-tint from that one color.
- A scenario studio where companies create AI characters (portrait, traits, mood, objections, voice), scenarios (objectives, opening line, difficulty, custom skills), and categories, each with a live preview and an AI drafting option.
- A native iOS app (Expo, React Native) with rep and manager modes, push-to-talk voice practice, self-serve company signup, the scenario studio, push notifications, universal links, and light and dark mode, delivered through EAS Build to TestFlight.
- Multi-tenant foundations: row-level security on every table, owner/admin/manager/rep roles scoped to stores, membership in multiple workspaces with per-device switching, and invite and team-code joins.
- Email: seven custom auth templates and eight alert types (assignments, nudges, due dates, coach notes, weekly recaps), sent by a database trigger through a Supabase Edge Function and co-branded with the company's logo and color.
- AI cost controls: metering on every OpenAI call, per-seat monthly budgets with alerts at 80% and 100%, and a soft cap that pauses voice while text practice keeps working.

## Technical highlights

### Real-time voice without exposing keys

The browser creates a WebRTC offer and sends it to the server, not to OpenAI. The server attaches the persona, the relevant playbook passages, and call instructions, then returns OpenAI's SDP answer. Audio then flows directly between the browser and OpenAI, both sides are transcribed, and finished turns are stored in order for scoring. On iPhone, push-to-talk runs a transcribe, chat, and text-to-speech pipeline in the character's voice.

### Scoring with structured outputs and custom skills

Each conversation is scored on the seven-skill rubric with an outcome, a summary, strong moments, missed chances, and turn references. Companies can add weighted custom skills per scenario; each gets a 0–100 score and a cited note, and the overall score is half the model's judgment and half the weighted custom-skill average.

### One brand color, one consistent theme

The brand color is converted to OKLCH. Surface, accent, chart, and button colors are derived from it, with chroma reduced until each color fits in sRGB and contrast checked for light and dark mode. The same math is ported to the React Native theme.

### Notifications that send once

Every notification is a database row. A Postgres hook fans it out to Expo push and to email through the Edge Function. Each path claims the row before sending, so nothing is delivered twice, and tapping a push for another workspace switches to it first.

### AI unit costs

| Session type | Before tuning | After tuning |
|---|---|---|
| Live voice (6-minute call) | about $0.45, no ceiling | about $0.07, at most about $0.12 |
| Mobile push-to-talk | about $0.05 | about $0.03 |
| Text (8 turns, a hint, scored) | under $0.01 | under $0.01 (measured $0.002) |

The reduction came from a smaller realtime model by default, shorter replies, minimal reasoning, cheaper transcription, 6-minute calls, and idle hang-up. Measured on real sessions, live voice became 74% cheaper and text 23% cheaper. At three voice sessions per rep per week, a 50-rep team costs about $45 a month to serve, against a default budget of $6 per seat.

## Architecture

```mermaid
flowchart LR
  subgraph Clients
    W["Web browser<br/>Next.js app"]
    M["iPhone app<br/>Expo + React Native"]
    E["Email inbox"]
  end

  subgraph Railway["Railway: Next.js 16"]
    RSC["Server components"]
    API["API route handlers"]
    RT["Realtime SDP proxy<br/>persona + retrieval"]
  end

  subgraph Supabase
    DB[("Postgres + RLS<br/>pgvector")]
    AUTH["Auth + email templates"]
    ST["Storage<br/>documents, portraits, logos"]
    EF["Edge Function<br/>send-email"]
  end

  subgraph OpenAI
    RTAPI["Realtime API<br/>WebRTC voice"]
    CHAT["Chat + scoring"]
    VOICE["Transcribe + TTS"]
    EMB["Embeddings"]
  end

  EXPO["Expo Push"]

  W --> RSC --> DB
  W --> API
  W -->|offer SDP| RT -->|answer SDP| RTAPI
  W <-->|WebRTC audio| RTAPI
  M -->|bearer token| API
  M --> DB
  API --> DB
  API --> CHAT
  API --> VOICE
  API --> EMB
  API --> ST
  DB -->|new notification| EF --> E
  DB -->|push hook| API --> EXPO --> M
  AUTH --> E
```

## Stack

| Layer | Technology |
|---|---|
| Web | Next.js 16 (App Router, server components, route handlers), React 19, TypeScript, Tailwind CSS v4, Leaflet |
| Mobile | Expo SDK 57, React Native 0.86, expo-router, Reanimated 4, expo-audio, expo-notifications, EAS Build and Submit |
| Data and auth | Supabase: Postgres with row-level security, Auth, Storage, pgvector (HNSW), pg_net, Edge Functions |
| AI | OpenAI Realtime API (WebRTC), gpt-4.1-mini for chat and scoring, text-embedding-3-small, gpt-4o-mini-transcribe, gpt-4o-mini-tts |
| Hosting | Railway, Supabase cloud, App Store Connect and TestFlight |

## By the numbers

| Metric | Web | iOS | Total |
|---|---|---|---|
| Commits | 1,071 | 496 | 1,567 |
| App source (TypeScript) | 43,774 lines | 27,932 lines | about 71,700 lines |
| Pages and screens | 49 pages | 41 screens | 90 |
| API route handlers | 96 | — | 96 |
| Database | 26 migrations, 27 tables, all with row-level security | — | — |
| End-to-end checks | 30 Playwright and Supabase scripts | — | — |

## Screenshots

### Marketing site

<p align="center">
  <img src="./marketing-managers.jpg" alt="Marketing page for managers" width="800"/>
</p>

<p align="center">
  <img src="./marketing-mobile-app.jpg" alt="Marketing page for the mobile app" width="800"/>
</p>

The home page hero is an animated phone running a scripted call with an interest meter and live captions. Separate pages cover the product, managers, and the mobile app.

### Rep home

<p align="center">
  <img src="./web-rep-home.jpg" alt="Rep home with assignments" width="800"/>
</p>

Assignments with step progress and due dates, plus level, streak, and rank, branded with the company's logo and colors.

### Scenario brief

<p align="center">
  <img src="./web-scenario-brief.jpg" alt="Scenario brief" width="800"/>
</p>

The customer's portrait, the objectives, past attempts, and a choice of live voice or text.

### Live voice session

<p align="center">
  <img src="./web-live-voice.jpg" alt="Live voice session" width="800"/>
</p>

Hold-to-talk or hands-free, with a live transcript, a call clock, and automatic scoring when the call ends.

### Score and review

<p align="center">
  <img src="./web-score-review.jpg" alt="Score and review" width="800"/>
</p>

Overall score, outcome, change since the last attempt, an attempts chart, the annotated conversation, and coach notes pinned to specific turns.

### Manager overview and expertise heatmap

<p align="center">
  <img src="./web-admin-overview.jpg" alt="Manager overview" width="800"/>
</p>

<p align="center">
  <img src="./web-expertise-heatmap.jpg" alt="Expertise heatmap" width="800"/>
</p>

Sessions, average score, active reps, close rate, top performers, and a rep-by-category heatmap.

### Store map

<p align="center">
  <img src="./web-store-map.jpg" alt="Store map" width="800"/>
</p>

Stores on a map, colored by score, experts, activity, momentum, or coverage, with cards for leading stores, expert hubs, the most improved, and stores that need attention.

### Team and assignments

<p align="center">
  <img src="./web-team.jpg" alt="Team management" width="800"/>
</p>

<p align="center">
  <img src="./web-assignment-detail.jpg" alt="Assignment detail" width="800"/>
</p>

Invites by link or team code, roles scoped to stores, and a multi-step assignment tracked by done, in progress, and overdue, with a single nudge for everyone unfinished.

### Scenario studio

<p align="center">
  <img src="./web-studio-scenarios.jpg" alt="Scenario library" width="800"/>
</p>

<p align="center">
  <img src="./web-scenario-editor.jpg" alt="Scenario editor with live preview" width="800"/>
</p>

<p align="center">
  <img src="./web-character-editor.jpg" alt="Character editor" width="800"/>
</p>

Reusable AI characters with portraits, traits, moods, hidden information, and voices, placed into scenarios with objectives and custom skills. Each editor shows what the rep will see.

### Brand and workspace settings

<p align="center">
  <img src="./web-settings-branding.jpg" alt="Brand and workspace settings" width="800"/>
</p>

Logo, primary color (or colors sampled from the logo), accent, and canvas style, with a live preview of the themed dashboard.

### iOS app: reps

<p align="center">
  <img src="./mobile-welcome.jpg" alt="Welcome screen" width="200"/>
  <img src="./mobile-rep-home-light.png" alt="Rep home, light mode" width="200"/>
  <img src="./mobile-rep-home-dark.png" alt="Rep home, dark mode" width="200"/>
  <img src="./mobile-voice-hold-to-talk.jpg" alt="Voice practice, hold to talk" width="200"/>
</p>

<p align="center">
  <img src="./mobile-score-reveal-dark.png" alt="Score reveal, dark mode" width="200"/>
  <img src="./mobile-rankings-dark.png" alt="Rankings, dark mode" width="200"/>
  <img src="./marketing-home-hero-mobile.jpg" alt="Marketing site on a phone" width="200"/>
</p>

Welcome carousel, home with a weekly goal ring, push-to-talk practice in the character's voice, the score reveal, and rankings, in light and dark mode.

### iOS app: managers, signup, and studio

<p align="center">
  <img src="./mobile-coach-dashboard-light.png" alt="Coach dashboard, light mode" width="200"/>
  <img src="./mobile-coach-dashboard-dark.png" alt="Coach dashboard, dark mode" width="200"/>
  <img src="./mobile-brand-colors.jpg" alt="Brand colors from logo" width="200"/>
  <img src="./mobile-character-creator-dark.jpg" alt="Character creator, dark mode" width="200"/>
</p>

A coach dashboard with KPIs and sparklines, the self-serve company signup that samples brand colors from the logo, and a step-by-step character creator.

### Email

<p align="center">
  <img src="./email-invite.jpg" alt="Invite email" width="340"/>
  <img src="./email-magic-link-dark.jpg" alt="Sign-in link email, dark mode" width="340"/>
</p>

Auth and alert emails carry the company's logo and button color alongside the Pitch Practice mark, and support dark mode.

## Status and next steps

- App Store release: external beta, store listing, and public launch (TestFlight is internal today)
- Android: configuration is in place; a Play Console build and listing are next
- Custom domain and a dedicated transactional email sender
- Packaging and billing: voice as a premium tier, using the plan and seat data already tracked per workspace
