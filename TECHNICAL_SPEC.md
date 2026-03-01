# Technical Specification - Progressive Hint Game Engine

## 🏗️ System Architecture

### High-Level Components

```
┌─────────────────────────────────────────────────────────────┐
│                        Frontend (React)                      │
│  - Game UI (Hint Display, Guess Input, Scoring)            │
│  - User Dashboard (Stats, Streaks, History)                 │
│  - Admin Panel (Word Curation, Hint Review)                 │
└──────────────────┬──────────────────────────────────────────┘
                   │ REST API / Firebase SDK
┌──────────────────▼──────────────────────────────────────────┐
│                  Backend Services Layer                      │
│  - Hint Generation Service (OpenAI Integration)             │
│  - Validation Service (Quality Control)                     │
│  - Game Manager (Logic, Scoring, State)                     │
│  - User Service (Auth, Progress, Stats)                     │
└──────────────────┬──────────────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────────────┐
│                    Data Layer                                │
│  - Firebase Firestore (Hints, User Data, Game State)       │
│  - Firebase Auth (User Authentication)                      │
│  - Redis Cache (Optional - Pre-generated Hints)             │
└─────────────────────────────────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────────────┐
│                 External Services                            │
│  - OpenAI API (GPT-4 / GPT-4-turbo)                        │
│  - Analytics (Google Analytics / Plausible)                 │
│  - Email Service (SendGrid for notifications)               │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔌 API Endpoints

### Hint Generation Service

#### POST `/api/hints/generate`
Generate hints for a specific entity.

**Request:**
```json
{
  "entity": "gregarious",
  "domain": "vocabulary",
  "date": "2025-01-03",
  "options": {
    "model": "gpt-4-turbo",
    "temperature": 0.7
  }
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "entity": "gregarious",
    "word_type": "adjective",
    "hints": [
      {
        "order": 1,
        "type": "domain/category",
        "text": "This describes a fundamental aspect of social psychology.",
        "word_count": 9,
        "validated": true
      },
      // ... 4 more hints
    ],
    "metadata": {
      "generated_at": "2025-01-03T10:00:00Z",
      "model": "gpt-4-turbo",
      "prompt_version": "vocab_v1",
      "validation_passed": true
    }
  }
}
```

**Error Response:**
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "Hint 3 contains substring of target word",
    "details": {
      "hint_order": 3,
      "violation": "word_leakage"
    }
  }
}
```

---

#### GET `/api/hints/daily/:domain`
Retrieve daily hint set for a specific domain.

**Request:**
```
GET /api/hints/daily/vocabulary?date=2025-01-03
```

**Response:**
```json
{
  "success": true,
  "data": {
    "domain": "vocabulary",
    "date": "2025-01-03",
    "entity": "gregarious",
    "hints": [/* array of 5 hints */],
    "difficulty": "medium",
    "cached": true
  }
}
```

---

### Game Management

#### POST `/api/game/start`
Start a new game session.

**Request:**
```json
{
  "user_id": "user_123",
  "domain": "vocabulary",
  "date": "2025-01-03"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "game_id": "game_abc123",
    "domain": "vocabulary",
    "date": "2025-01-03",
    "hints_available": 5,
    "max_attempts": 6,
    "started_at": "2025-01-03T10:15:00Z"
  }
}
```

---

#### POST `/api/game/guess`
Submit a guess for the current game.

**Request:**
```json
{
  "game_id": "game_abc123",
  "guess": "gregarious"
}
```

**Response (Correct):**
```json
{
  "success": true,
  "data": {
    "correct": true,
    "attempts": 3,
    "hints_revealed": 3,
    "score": 60,
    "mastery_level": "Proficient",
    "game_completed": true,
    "answer": "gregarious",
    "all_hints": [/* array of all 5 hints */]
  }
}
```

**Response (Incorrect):**
```json
{
  "success": true,
  "data": {
    "correct": false,
    "attempts": 2,
    "hints_revealed": 2,
    "next_hint": {
      "order": 3,
      "type": "contradictory/exclusion",
      "text": "Not about being friendly; more about seeking company."
    },
    "attempts_remaining": 4
  }
}
```

---

#### GET `/api/game/stats/:user_id`
Get user statistics and progress.

**Response:**
```json
{
  "success": true,
  "data": {
    "user_id": "user_123",
    "overall_stats": {
      "games_played": 45,
      "games_won": 42,
      "win_rate": 0.933,
      "current_streak": 12,
      "max_streak": 18,
      "average_hints": 3.2,
      "total_score": 2860
    },
    "by_domain": {
      "vocabulary": {
        "games_played": 30,
        "average_score": 68,
        "mastery_distribution": {
          "Master": 5,
          "Expert": 12,
          "Proficient": 10,
          "Competent": 2,
          "Learner": 1
        }
      }
    },
    "recent_games": [/* last 10 games */]
  }
}
```

---

## 💾 Database Schemas

### Firestore Collections

#### `hints` Collection
Stores generated hints for each domain/date/entity.

```javascript
// Document ID: vocabulary_20250103_gregarious
{
  domain: "vocabulary",
  date: "2025-01-03",
  entity: "gregarious",
  entity_type: "adjective",
  difficulty: "medium",
  hints: [
    {
      order: 1,
      type: "domain/category",
      text: "This describes a fundamental aspect of social psychology.",
      word_count: 9,
      validated: true
    },
    // ... 4 more hints
  ],
  metadata: {
    generated_at: Timestamp,
    model: "gpt-4-turbo",
    prompt_version: "vocab_v1",
    temperature: 0.7,
    validation_passed: true,
    manual_review: false
  },
  usage_stats: {
    games_played: 1523,
    average_attempts: 3.4,
    completion_rate: 0.89
  }
}
```

**Indexes:**
- `domain + date` (for daily lookup)
- `difficulty` (for filtering)
- `metadata.generated_at` (for cleanup/archival)

---

#### `games` Collection
Individual game sessions.

```javascript
// Document ID: auto-generated game_id
{
  game_id: "game_abc123",
  user_id: "user_123",
  domain: "vocabulary",
  date: "2025-01-03",
  entity: "gregarious",
  
  // Game state
  guesses: ["outgoing", "sociable", "gregarious"],
  hints_revealed: [1, 2, 3],
  current_hint: 3,
  attempts: 3,
  max_attempts: 6,
  
  // Result
  solved: true,
  score: 60,
  mastery_level: "Proficient",
  
  // Timestamps
  started_at: Timestamp,
  completed_at: Timestamp,
  time_taken_seconds: 145,
  
  // Context
  device: "mobile",
  version: "1.0.0"
}
```

**Indexes:**
- `user_id + date` (for user history)
- `domain + date` (for daily stats)
- `completed_at` (for recent games)

---

#### `users` Collection
User profiles and statistics.

```javascript
// Document ID: user_id
{
  user_id: "user_123",
  email: "user@example.com",
  username: "vocab_master",
  
  // Subscription
  tier: "premium", // free | premium | educational | enterprise
  subscription_status: "active",
  subscription_expires: Timestamp,
  
  // Overall Stats
  stats: {
    total_games: 45,
    total_wins: 42,
    win_rate: 0.933,
    current_streak: 12,
    max_streak: 18,
    total_score: 2860,
    average_hints: 3.2,
    
    by_domain: {
      vocabulary: {
        games_played: 30,
        games_won: 28,
        average_score: 68,
        best_score: 100,
        mastery_counts: {
          Master: 5,
          Expert: 12,
          Proficient: 10,
          Competent: 2,
          Learner: 1
        }
      },
      geography: { /* similar structure */ }
    }
  },
  
  // Preferences
  settings: {
    daily_reminder: true,
    reminder_time: "09:00",
    theme: "dark",
    difficulty_preference: "medium"
  },
  
  // Metadata
  created_at: Timestamp,
  last_played: Timestamp,
  last_login: Timestamp
}
```

**Indexes:**
- `stats.current_streak` (for leaderboards)
- `stats.total_score` (for leaderboards)
- `tier` (for analytics)

---

#### `daily_words` Collection
Curated words for each day and domain.

```javascript
// Document ID: vocabulary_20250103
{
  domain: "vocabulary",
  date: "2025-01-03",
  entity: "gregarious",
  entity_type: "adjective",
  difficulty: "medium",
  
  // Curation
  curated_by: "admin_id",
  curated_at: Timestamp,
  approved: true,
  
  // Alternative answers (if applicable)
  alternatives: [],
  
  // Educational info (shown after game)
  educational_content: {
    definition: "Fond of company; sociable.",
    etymology: "From Latin gregarius 'belonging to a flock'",
    example_sentences: [
      "She was gregarious by nature and loved attending parties.",
      "Gregarious animals like wolves live in packs."
    ],
    related_words: ["sociable", "outgoing", "convivial"],
    word_origin_year: "1668"
  },
  
  // Scheduling
  scheduled_for: Timestamp,
  published: true
}
```

---

## 🧩 Core Service Modules

### 1. Hint Generator Service

```typescript
// services/hintGenerator.ts

interface HintGenerationOptions {
  model?: 'gpt-4' | 'gpt-4-turbo' | 'gpt-3.5-turbo';
  temperature?: number;
  maxRetries?: number;
}

interface GeneratedHints {
  entity: string;
  word_type: string;
  hints: Hint[];
  metadata: GenerationMetadata;
}

class HintGeneratorService {
  private openai: OpenAI;
  private validator: HintValidator;
  
  async generateHints(
    entity: string,
    domain: string,
    options?: HintGenerationOptions
  ): Promise<GeneratedHints> {
    // 1. Get domain-specific prompt template
    const prompt = this.buildPrompt(entity, domain);
    
    // 2. Call OpenAI API
    const response = await this.openai.chat.completions.create({
      model: options?.model || 'gpt-4-turbo',
      messages: [
        { role: 'system', content: prompt },
        { role: 'user', content: entity }
      ],
      temperature: options?.temperature || 0.7,
      response_format: { type: 'json_object' }
    });
    
    // 3. Parse JSON response
    const hints = JSON.parse(response.choices[0].message.content);
    
    // 4. Validate hints
    const validation = await this.validator.validate(hints, entity);
    if (!validation.passed) {
      // Retry or log error
      throw new ValidationError(validation.errors);
    }
    
    // 5. Return validated hints
    return {
      ...hints,
      metadata: {
        generated_at: new Date(),
        model: options?.model || 'gpt-4-turbo',
        prompt_version: this.getPromptVersion(domain),
        validation_passed: true
      }
    };
  }
  
  private buildPrompt(entity: string, domain: string): string {
    // Load domain-specific prompt from prompts.ts
    return DomainPrompts[domain].replace('<WORD>', entity);
  }
}
```

---

### 2. Hint Validator Service

```typescript
// services/hintValidator.ts

interface ValidationResult {
  passed: boolean;
  errors: ValidationError[];
  warnings: ValidationWarning[];
}

interface ValidationError {
  hint_order: number;
  type: 'word_leakage' | 'word_count' | 'duplicate_type' | 'progression';
  message: string;
}

class HintValidator {
  async validate(hints: any, targetWord: string): Promise<ValidationResult> {
    const errors: ValidationError[] = [];
    const warnings: ValidationWarning[] = [];
    
    // Rule 1: Exactly 5 hints
    if (hints.hints.length !== 5) {
      errors.push({
        hint_order: 0,
        type: 'count',
        message: `Expected 5 hints, got ${hints.hints.length}`
      });
    }
    
    // Rule 2: No word leakage
    for (const hint of hints.hints) {
      if (this.containsWord(hint.text, targetWord)) {
        errors.push({
          hint_order: hint.order,
          type: 'word_leakage',
          message: `Hint ${hint.order} contains target word or substring`
        });
      }
    }
    
    // Rule 3: Word count ≤ 15
    for (const hint of hints.hints) {
      const wordCount = hint.text.split(/\s+/).length;
      if (wordCount > 15) {
        errors.push({
          hint_order: hint.order,
          type: 'word_count',
          message: `Hint ${hint.order} has ${wordCount} words (max 15)`
        });
      }
    }
    
    // Rule 4: No duplicate hint types
    const types = hints.hints.map(h => h.type);
    const uniqueTypes = new Set(types);
    if (types.length !== uniqueTypes.size) {
      warnings.push({
        message: 'Some hint types are repeated'
      });
    }
    
    // Rule 5: Progressive difficulty (heuristic check)
    // Check if later hints contain more specific keywords
    
    return {
      passed: errors.length === 0,
      errors,
      warnings
    };
  }
  
  private containsWord(text: string, word: string): boolean {
    const textLower = text.toLowerCase();
    const wordLower = word.toLowerCase();
    
    // Check exact word
    if (textLower.includes(wordLower)) return true;
    
    // Check root forms (simple stemming)
    const roots = this.getWordRoots(wordLower);
    for (const root of roots) {
      if (root.length > 3 && textLower.includes(root)) {
        return true;
      }
    }
    
    return false;
  }
  
  private getWordRoots(word: string): string[] {
    // Simple root extraction (improve with NLP library)
    const roots = [word];
    
    // Remove common suffixes
    const suffixes = ['ing', 'ed', 'ly', 'ous', 'ious', 'ness', 'ment'];
    for (const suffix of suffixes) {
      if (word.endsWith(suffix)) {
        roots.push(word.slice(0, -suffix.length));
      }
    }
    
    return roots;
  }
}
```

---

### 3. Game Manager Service

```typescript
// services/gameManager.ts

interface GameState {
  game_id: string;
  user_id: string;
  entity: string;
  guesses: string[];
  hints_revealed: number[];
  solved: boolean;
  score?: number;
}

class GameManager {
  private db: Firestore;
  
  async startGame(userId: string, domain: string, date: string): Promise<GameState> {
    // 1. Get daily hint set
    const hints = await this.getDailyHints(domain, date);
    
    // 2. Create game document
    const gameId = this.generateGameId();
    const gameState: GameState = {
      game_id: gameId,
      user_id: userId,
      domain,
      date,
      entity: hints.entity,
      guesses: [],
      hints_revealed: [],
      current_hint: 0,
      attempts: 0,
      max_attempts: 6,
      solved: false,
      started_at: new Date()
    };
    
    // 3. Save to database
    await this.db.collection('games').doc(gameId).set(gameState);
    
    return gameState;
  }
  
  async submitGuess(gameId: string, guess: string): Promise<GuessResult> {
    // 1. Load game state
    const game = await this.loadGame(gameId);
    
    // 2. Normalize guess
    const normalizedGuess = this.normalizeGuess(guess);
    const normalizedAnswer = this.normalizeGuess(game.entity);
    
    // 3. Check correctness
    const correct = normalizedGuess === normalizedAnswer;
    
    // 4. Update game state
    game.guesses.push(guess);
    game.attempts++;
    
    if (correct) {
      game.solved = true;
      game.score = this.calculateScore(game.hints_revealed.length + 1);
      game.mastery_level = this.getMasteryLevel(game.score);
      game.completed_at = new Date();
    } else if (game.attempts < game.max_attempts) {
      // Reveal next hint
      game.current_hint++;
      game.hints_revealed.push(game.current_hint);
    } else {
      // Game lost
      game.solved = false;
      game.score = 0;
      game.completed_at = new Date();
    }
    
    // 5. Save updated state
    await this.saveGame(game);
    
    // 6. Update user stats if game completed
    if (game.solved || game.attempts >= game.max_attempts) {
      await this.updateUserStats(game);
    }
    
    return this.buildGuessResult(game, correct);
  }
  
  private calculateScore(hintsUsed: number): number {
    const scoreMap = {
      1: 100, // Master
      2: 80,  // Expert
      3: 60,  // Proficient
      4: 40,  // Competent
      5: 20   // Learner
    };
    return scoreMap[hintsUsed] || 0;
  }
  
  private getMasteryLevel(score: number): string {
    if (score === 100) return 'Master';
    if (score >= 80) return 'Expert';
    if (score >= 60) return 'Proficient';
    if (score >= 40) return 'Competent';
    if (score >= 20) return 'Learner';
    return 'Incomplete';
  }
  
  private normalizeGuess(text: string): string {
    return text.toLowerCase().trim().replace(/[^\w]/g, '');
  }
}
```

---

## 🔐 Security & Authentication

### Firebase Authentication
- Email/Password
- Google OAuth
- Anonymous (for free tier trial)

### API Security
- JWT tokens for authenticated requests
- Rate limiting: 100 requests/hour per IP (free tier)
- API key required for hint generation (internal only)
- CORS configured for production domain only

### Data Access Rules (Firestore)
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // Hints are readable by all authenticated users
    match /hints/{hintId} {
      allow read: if request.auth != null;
      allow write: if request.auth.token.admin == true;
    }
    
    // Users can only access their own games
    match /games/{gameId} {
      allow read, write: if request.auth != null 
                          && request.auth.uid == resource.data.user_id;
    }
    
    // Users can only access their own profile
    match /users/{userId} {
      allow read, write: if request.auth != null 
                          && request.auth.uid == userId;
    }
    
    // Daily words readable by all
    match /daily_words/{wordId} {
      allow read: if request.auth != null;
      allow write: if request.auth.token.admin == true;
    }
  }
}
```

---

## 📊 Analytics & Monitoring

### Key Metrics to Track

**User Engagement**
- DAU (Daily Active Users)
- MAU (Monthly Active Users)
- Games per user per day
- Retention (D1, D7, D30)
- Session duration

**Game Performance**
- Average hints per game by domain
- Completion rate
- Score distribution
- Time to complete
- Guess patterns

**Business Metrics**
- Free → Premium conversion rate
- Churn rate
- MRR (Monthly Recurring Revenue)
- CAC (Customer Acquisition Cost)
- LTV (Lifetime Value)

**Technical Metrics**
- API response times
- OpenAI API costs per domain
- Database read/write costs
- Error rates
- Validation failure rates

### Monitoring Tools
- Firebase Analytics (user behavior)
- Cloud Monitoring (infrastructure)
- Sentry (error tracking)
- Custom dashboard for game metrics

---

## 💰 Cost Estimation

### OpenAI API Costs
- GPT-4-turbo: ~$0.01 per hint set (5 hints)
- Daily generation: 3 domains × $0.01 = $0.03/day = ~$10/month
- With caching/pre-generation, very affordable

### Firebase Costs (Spark Plan - Free Tier)
- 50K reads, 20K writes per day (sufficient for MVP)
- 1GB storage (plenty for hint data)

### Firebase Costs (Blaze Plan - Pay as you go)
- Estimated 100K users:
  - Reads: ~500K/day = $0.18/day = $5.40/month
  - Writes: ~100K/day = $0.54/day = $16.20/month
  - Storage: ~5GB = $1.25/month
  - **Total: ~$25/month**

### Hosting (Vercel)
- Free tier for frontend (generous limits)
- Pro plan if needed: $20/month

### Total Monthly Cost (100K users)
- Infrastructure: ~$50-75/month
- Profit margin at $20K MRR: 99.6%

---

## 🚀 Deployment Strategy

### Environments
1. **Development** - Local + Firebase emulators
2. **Staging** - Separate Firebase project, testing domain
3. **Production** - Main Firebase project, production domain

### CI/CD Pipeline
```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - Checkout code
      - Run unit tests
      - Run integration tests
      - Run validation tests
  
  deploy-functions:
    needs: test
    steps:
      - Deploy Firebase Cloud Functions
      - Run smoke tests
  
  deploy-frontend:
    needs: test
    steps:
      - Build React app
      - Deploy to Vercel
      - Run E2E tests
```

### Rollback Strategy
- Keep previous 5 function versions
- Instant rollback via Firebase console
- Frontend versioning via Vercel deployments

---

## 🧪 Testing Strategy

### Unit Tests
- Hint validation logic
- Scoring calculations
- Word normalization
- Prompt building

### Integration Tests
- OpenAI API integration
- Firebase read/write operations
- Authentication flows
- Game state management

### End-to-End Tests (Playwright/Cypress)
- Complete game flow
- Hint revelation sequence
- Statistics updates
- Social sharing

### Validation Tests
- Test 100+ words through hint generation
- Verify no word leakage
- Ensure progressive difficulty
- Check response times

---

## 📱 Frontend Component Structure

### Key React Components

```
src/
├── components/
│   ├── game/
│   │   ├── HintDisplay.tsx        # Shows current hint
│   │   ├── HintList.tsx           # Shows all revealed hints
│   │   ├── GuessInput.tsx         # Input field for guesses
│   │   ├── ScoreDisplay.tsx       # Current score/mastery
│   │   ├── AttemptsCounter.tsx    # Remaining attempts
│   │   └── GameComplete.tsx       # End game summary
│   ├── stats/
│   │   ├── StatsOverview.tsx      # Overall statistics
│   │   ├── DomainStats.tsx        # Per-domain breakdown
│   │   ├── StreakDisplay.tsx      # Current/max streak
│   │   └── MasteryChart.tsx       # Score distribution
│   ├── modals/
│   │   ├── HowToPlay.tsx          # Tutorial
│   │   ├── SettingsModal.tsx      # User preferences
│   │   ├── UpgradeModal.tsx       # Premium upsell
│   │   └── ShareModal.tsx         # Social sharing
│   └── layout/
│       ├── Navbar.tsx             # Top navigation
│       ├── DomainSelector.tsx     # Switch domains
│       └── Footer.tsx             # Links/info
├── hooks/
│   ├── useGame.ts                 # Game state management
│   ├── useHints.ts                # Hint data fetching
│   ├── useStats.ts                # Statistics data
│   └── useAuth.ts                 # Authentication
├── services/
│   ├── api.ts                     # API client
│   ├── firebase.ts                # Firebase config
│   └── analytics.ts               # Event tracking
├── types/
│   ├── game.types.ts              # Game interfaces
│   ├── hint.types.ts              # Hint interfaces
│   └── user.types.ts              # User interfaces
└── utils/
    ├── scoring.ts                 # Score calculations
    ├── validation.ts              # Input validation
    └── formatting.ts              # Display helpers
```

---

## 🔄 State Management

### Game State (React Context)
```typescript
interface GameContextType {
  currentGame: GameState | null;
  hints: Hint[];
  revealedHints: Hint[];
  currentScore: number;
  attempts: number;
  isGameActive: boolean;
  
  // Actions
  startGame: (domain: string) => Promise<void>;
  submitGuess: (guess: string) => Promise<GuessResult>;
  revealNextHint: () => void;
  endGame: () => void;
}
```

### User State (React Context)
```typescript
interface UserContextType {
  user: User | null;
  isAuthenticated: boolean;
  stats: UserStats | null;
  subscription: SubscriptionInfo;
  
  // Actions
  login: (email: string, password: string) => Promise<void>;
  logout: () => Promise<void>;
  updateProfile: (updates: Partial<User>) => Promise<void>;
  refreshStats: () => Promise<void>;
}
```

---

## 🎨 UI/UX Considerations

### Hint Revelation Animation
- Smooth fade-in transition
- Highlight newly revealed hint
- Dim previous hints slightly
- Progress indicator showing hints used

### Feedback Mechanisms
- Incorrect guess: Shake animation + red flash
- Correct guess: Confetti + success sound
- Near miss: Yellow highlight (if implementing fuzzy matching)

### Responsive Design
- Mobile-first approach
- Touch-friendly input
- Keyboard shortcuts for desktop
- Landscape mode optimization

### Accessibility
- ARIA labels for screen readers
- Keyboard navigation
- High contrast mode
- Focus indicators
- Skip to main content link

---

**Last Updated:** 2025-11-03  
**Status:** Planning Phase  
**Next:** Implement Hint Generator Service + Validation
