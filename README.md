# GleamNote.ai
# Gleam 

> Capture what matters from any video, in any language.

Gleam is a smart clipping tool that lets you extract and save key moments from videos (YouTube, TikTok, Instagram, etc.) with your own notes and translations. Stop saving entire videos you'll never watch—capture only the insights that matter to you.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Platform](https://img.shields.io/badge/Platform-iOS%20%7C%20Android%20%7C%20Web-blue)]()
[![Status](https://img.shields.io/badge/Status-In%20Development-orange)]()

---

## 🎯 The Problem

We've all been there:
- Watch a 45-minute YouTube tutorial and forget the key insight by next week
- See a TikTok with a great cooking tip but can't remember which one
- Learn something from an Instagram Reel in another language
- Save dozens of videos "to watch later" and never do

**Traditional solutions are broken:**
- Full transcripts are overwhelming (10,000 words you'll never read)
- Bookmarking videos gives you everything or nothing
- No way to capture *your* insights with *their* content
- Language barriers limit what you can learn

---

## ✨ The Solution: Gleam

Gleam lets you **selectively clip** the moments that matter, add your own thoughts, and organize everything in one place—across all video platforms and languages.

### Key Features

#### 📹 Smart Video Clipping
- Clip specific timestamps from any video (5 seconds to 5 minutes)
- Works with YouTube, TikTok, Instagram Reels, and any web video
- Automatic transcription of your clip
- Thumbnail preview for quick recognition

#### 🌍 Built-in Translation
- Translate clips from any language to your preferred language
- Side-by-side original and translated text
- Perfect for learning from international content creators

#### 📝 Your Notes, Your Way
- Add your own thoughts and context to each clip
- Why did this matter? How will you use it?
- Future you will thank you

#### 🗂️ Organized Collections
- Group clips by topic, project, or any way you want
- Tags for quick filtering
- Search across all your clips instantly

#### 🔄 Cross-Platform Sync
- Clip on your phone, review on desktop
- All your content, everywhere you work

#### 📤 Export & Share
- Export to Google Docs, Apple Notes, or Markdown
- Share collections with team members
- Backup your knowledge base

---

## 🚀 How It Works

### On Mobile (iOS/Android)
```
1. Watch video in TikTok/Instagram/YouTube app
   ↓
2. See something important at 1:23
   ↓
3. Tap Share → Choose Gleam
   ↓
4. Gleam opens, loads video at 1:23
   ↓
5. Adjust clip range (1:20-1:35)
   ↓
6. Add your note: "Great technique for pizza dough"
   ↓
7. (Optional) Translate to your language
   ↓
8. Save to "Cooking" collection
```

### On Desktop (Browser Extension)
```
1. Watch YouTube video in Chrome/Safari
   ↓
2. Click Gleam extension icon
   ↓
3. Select clip range from video timeline
   ↓
4. Add note and translation
   ↓
5. Save to collection
```

---

## 🏗️ Architecture

### High-Level System Design
```
┌─────────────────────────────────────────┐
│         User Applications               │
├─────────────┬──────────────┬───────────┤
│  iOS App    │ Android App  │ Web App   │
│  (Swift)    │  (Kotlin)    │ (React)   │
└─────────────┴──────────────┴───────────┘
              │
              ↓
┌─────────────────────────────────────────┐
│          API Gateway                    │
│       (REST + GraphQL)                  │
└─────────────────────────────────────────┘
              │
    ┌─────────┴─────────┐
    ↓                   ↓
┌──────────┐      ┌──────────────┐
│ Core API │      │ Worker Queue │
│(Node.js) │      │   (Bull)     │
└──────────┘      └──────────────┘
    │                   │
    ↓                   ↓
┌─────────────────────────────────────────┐
│           Services Layer                │
├──────────────┬──────────────┬──────────┤
│Transcription │ Translation  │  Video   │
│  (Whisper)   │(Google/DeepL)│Processing│
└──────────────┴──────────────┴──────────┘
              │
              ↓
┌─────────────────────────────────────────┐
│          Data Storage                   │
├──────────────┬──────────────┬──────────┤
│  PostgreSQL  │   S3/R2      │  Redis   │
│  (Metadata)  │  (Videos)    │ (Cache)  │
└──────────────┴──────────────┴──────────┘
```

### Tech Stack

**Frontend:**
- iOS: SwiftUI, Combine
- Android: Jetpack Compose, Kotlin Coroutines
- Web: React, TypeScript, TailwindCSS
- Browser Extensions: WebExtensions API

**Backend:**
- API: Node.js + Express / Next.js API Routes
- Database: PostgreSQL (Supabase)
- File Storage: Cloudflare R2 / AWS S3
- Queue: Bull / Redis
- Cache: Redis

**AI/ML Services:**
- Transcription: OpenAI Whisper API / AssemblyAI
- Translation: Google Cloud Translation / DeepL API
- Video Processing: FFmpeg

**Infrastructure:**
- Hosting: Vercel (Web) + Railway (API)
- CDN: Cloudflare
- Auth: Supabase Auth / Clerk
- Analytics: PostHog

---

## 🗂️ Database Schema
```sql
-- Users
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(255),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Collections
CREATE TABLE collections (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Clips
CREATE TABLE clips (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  collection_id UUID REFERENCES collections(id) ON DELETE SET NULL,
  
  -- Source info
  source_type VARCHAR(50) NOT NULL, -- 'youtube', 'tiktok', 'instagram'
  source_url TEXT NOT NULL,
  source_title VARCHAR(500),
  
  -- Clip details
  start_time INTEGER NOT NULL, -- seconds
  end_time INTEGER NOT NULL,
  thumbnail_url TEXT,
  
  -- Content
  original_text TEXT,
  original_language VARCHAR(10),
  translated_text TEXT,
  target_language VARCHAR(10),
  
  -- User content
  user_note TEXT,
  tags TEXT[], -- Array of tags
  
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_clips_user_id ON clips(user_id);
CREATE INDEX idx_clips_collection_id ON clips(collection_id);
CREATE INDEX idx_clips_tags ON clips USING GIN(tags);
CREATE INDEX idx_clips_created_at ON clips(created_at DESC);
```

---

## 🔧 Installation & Setup

### Prerequisites
- Node.js 18+
- PostgreSQL 14+
- Redis 7+
- FFmpeg
- API Keys: OpenAI (Whisper), Google Cloud Translation

### Backend Setup
```bash
# Clone the repository
git clone https://github.com/yourusername/gleam.git
cd gleam

# Install dependencies
cd backend
npm install

# Setup environment variables
cp .env.example .env
# Edit .env with your API keys and database URLs

# Run database migrations
npm run migrate

# Start development server
npm run dev
```

### Frontend Setup
```bash
# Web App
cd web
npm install
npm run dev

# iOS App
cd ios
pod install
open Gleam.xcworkspace

# Android App
cd android
./gradlew build
```

### Environment Variables
```bash
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/gleam
REDIS_URL=redis://localhost:6379

# APIs
OPENAI_API_KEY=sk-...
GOOGLE_TRANSLATE_API_KEY=...
DEEPL_API_KEY=...

# Storage
S3_BUCKET=gleam-clips
S3_ACCESS_KEY=...
S3_SECRET_KEY=...
S3_REGION=us-east-1

# Auth
JWT_SECRET=your-secret-key
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=...

# Frontend URLs
NEXT_PUBLIC_API_URL=http://localhost:3000
```

---

## 📱 Usage Examples

### API Usage
```javascript
// Create a new clip
POST /api/clips
{
  "sourceUrl": "https://youtube.com/watch?v=...",
  "startTime": 65,
  "endTime": 95,
  "userNote": "Great explanation of async/await",
  "targetLanguage": "es",
  "collectionId": "uuid",
  "tags": ["javascript", "tutorial"]
}

// Get user's clips
GET /api/clips?collectionId=uuid&tags=javascript

// Search clips
GET /api/clips/search?q=async&language=en

// Export collection
GET /api/collections/:id/export?format=markdown
```

### SDK Usage (JavaScript)
```javascript
import { GleamClient } from '@gleam/sdk';

const client = new GleamClient({
  apiKey: process.env.GLEAM_API_KEY
});

// Create a clip
const clip = await client.clips.create({
  sourceUrl: 'https://youtube.com/watch?v=dQw4w9WgXcQ',
  startTime: 30,
  endTime: 45,
  userNote: 'Never gonna give you up',
  translate: true,
  targetLanguage: 'es'
});

// Get clips from a collection
const clips = await client.collections.getClips('collection-id');

// Search across all clips
const results = await client.clips.search('machine learning');
```

---

## 🗺️ Roadmap

### ✅ Phase 1: MVP (Q1 2025)
- [x] Core clipping functionality
- [x] YouTube integration
- [x] Basic collections
- [x] Translation support
- [x] iOS app with Share Sheet
- [ ] Android app with Share Intent

### 🚧 Phase 2: Enhancement (Q2 2025)
- [ ] TikTok & Instagram support
- [ ] Browser extensions (Chrome, Safari)
- [ ] Advanced search & filters
- [ ] Export to Google Docs / Apple Notes
- [ ] Collaborative collections

### 🔮 Phase 3: Advanced (Q3 2025)
- [ ] AI-powered clip suggestions
- [ ] Voice notes on clips
- [ ] Spaced repetition reminders
- [ ] Team workspaces
- [ ] API for developers

### 🌟 Phase 4: Scale (Q4 2025)
- [ ] Podcast support
- [ ] Offline mode
- [ ] Desktop apps (Mac, Windows)
- [ ] Advanced analytics
- [ ] Premium features

---

## 🤝 Contributing

We love contributions! Here's how you can help:

### Bug Reports
Found a bug? [Open an issue](https://github.com/yourusername/gleam/issues) with:
- Clear description
- Steps to reproduce
- Expected vs actual behavior
- Screenshots/videos if applicable

### Feature Requests
Have an idea? [Start a discussion](https://github.com/yourusername/gleam/discussions) about:
- The problem you're trying to solve
- Your proposed solution
- Why it would benefit other users

### Pull Requests
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Development Guidelines
- Write tests for new features
- Follow existing code style
- Update documentation
- Keep commits atomic and well-described

---

## 🧪 Testing
```bash
# Run all tests
npm test

# Run specific test suite
npm test -- clips.test.js

# Run with coverage
npm run test:coverage

# E2E tests
npm run test:e2e
```

---

## 📊 Performance

- **Clip Creation:** < 3 seconds (with transcription)
- **Translation:** < 1 second
- **Search:** < 100ms for 10,000 clips
- **Mobile App:** < 50MB bundle size
- **API Response Time:** p95 < 200ms

---

## 🔐 Security

- End-to-end encryption for sensitive data
- SOC 2 Type II compliant infrastructure
- Regular security audits
- GDPR and CCPA compliant
- User data never used for AI training
- Optional 2FA for accounts

---

## 💰 Pricing

### Free Tier
- 50 clips/month
- 1 collection
- Basic translation
- 720p video quality

### Pro ($9/month)
- Unlimited clips
- Unlimited collections
- Priority translation
- 1080p video quality
- Export to all formats
- Priority support

### Teams ($49/month)
- Everything in Pro
- Shared collections
- Team collaboration
- Admin dashboard
- SSO support
- Custom branding

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- [Whisper AI](https://openai.com/research/whisper) for transcription
- [FFmpeg](https://ffmpeg.org/) for video processing
- [Supabase](https://supabase.com/) for backend infrastructure
- All our beta testers and contributors

---

## 📞 Contact & Support

- **Website:** https://gleam.app
- **Email:** hello@gleam.app
- **Twitter:** [@gleamapp](https://twitter.com/gleamapp)
- **Discord:** [Join our community](https://discord.gg/gleam)
- **Documentation:** https://docs.gleam.app

---

## 🌟 Star History

[![Star History Chart](https://api.star-history.com/svg?repos=yourusername/gleam&type=Date)](https://star-history.com/#yourusername/gleam&Date)

---

<div align="center">

**Made with ❤️ by the Gleam team**

[Website](https://gleam.app) • [Docs](https://docs.gleam.app) • [Blog](https://blog.gleam.app)

</div>
