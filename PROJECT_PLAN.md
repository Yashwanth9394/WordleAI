# Progressive Hint Game Engine - Project Plan

## 🎯 Project Vision

Transform the basic Wordle clone into a **daily progressive hint game engine** that generates 5 AI-powered hints for any entity (word, place, concept, etc.). Players guess the answer as hints get progressively easier, demonstrating mastery by guessing early.

---

## 🏗️ System Architecture Overview

### Core Concept
- **Input:** Any entity (word, place, concept)
- **Processing:** AI generates 5 progressive hints using structured prompts
- **Output:** JSON schema with validated hints
- **Gameplay:** Reveal hints one-by-one until player guesses correctly
- **Scoring:** Earlier guess = higher mastery score

### Multi-Domain Support
Same engine, different prompt templates:
- 🔤 **Vocabulary** - English words, etymology, linguistics
- 🌍 **Geography** - Countries, cities, landmarks
- 🧪 **Science** - Concepts, elements, theories
- 🎭 **Pop Culture** - Movies, celebrities, events
- 📚 **History** - Events, figures, periods
- *Expandable to any domain*

---

## 🔄 Hint Generation Flow

### Step-by-Step Process

1. **Input Entity**
   - Daily word/concept selected (manual or automated)
   - Entity metadata: type, difficulty level, domain

2. **LLM API Call**
   - Send entity + structured prompt to OpenAI API
   - Use GPT-4 or GPT-4-turbo for quality
   - Temperature: 0.7-0.8 for creativity with consistency

3. **JSON Response**
   ```json
   {
     "word": "gregarious",
     "word_type": "adjective",
     "hints": [
       {"order": 1, "type": "domain/category", "text": "..."},
       {"order": 2, "type": "analogical/functional", "text": "..."},
       {"order": 3, "type": "contradictory/exclusion", "text": "..."},
       {"order": 4, "type": "semantic/cultural", "text": "..."},
       {"order": 5, "type": "etymological/structural", "text": "..."}
     ]
   }
   ```

4. **Validation Layer**
   - ✅ Exactly 5 hints present
   - ✅ Each hint ≤ 15 words
   - ✅ No target word leakage (no substrings, roots)
   - ✅ Progressive difficulty (each easier than previous)
   - ✅ Diverse hint types (no repetition)
   - ✅ Valid JSON structure

5. **Storage**
   - Save to Firebase Firestore or AWS DynamoDB
   - Schema: `{domain}/{date}/{entity_id}`
   - Include metadata: generation timestamp, version, domain

6. **Frontend Display**
   - Reveal hints sequentially on wrong guesses
   - Track guess attempts
   - Calculate mastery score
   - Show statistics and streaks

---

## 🧠 Vocabulary Domain - Base Prompt Template

### System Prompt (OpenAI API)

```
You are an expert linguist and game designer generating progressive word hints for an English vocabulary guessing game where players receive one hint at a time and try to guess the word. The player's goal is to guess the word as early as possible to demonstrate linguistic mastery.

Generate exactly 5 hints for the word provided that follow this difficulty progression:

**Hint 1 (Hardest):** Abstract context - domain/category or conceptual association. Player thinks: "Intriguing but unclear."

**Hint 2 (Medium-Hard):** Directional progress - analogical/riddled clue or functional hint. Player thinks: "I know the area now."

**Hint 3 (Medium):** Contrast or exclusion - eliminate wrong answers or clarify confusion. Player thinks: "Wait, maybe I was close."

**Hint 4 (Easy-Moderate):** Near-semantic reveal - synonym, descriptive definition, or cultural reference. Player thinks: "I can guess now."

**Hint 5 (Easiest):** Closing anchor - etymological origin, pattern/structural clue, or partial reveal. Player thinks: "I almost have it!"

**Word-Type Adaptation Rules:**
- **Adjective:** Focus on traits, qualities, and contrasts (not actions or things).
- **Verb:** Focus on purpose, effect, or common context of the action.
- **Noun (concrete):** Use category, use-case, or physical characteristics.
- **Abstract noun:** Use emotion, idea, or human experience analogies.
- **Technical/scientific term:** Mention its field first, then describe function.
- **Idiomatic/rare word:** Use origin, cultural clue, or metaphorical description.

**Hint Type Toolkit (use variety across the 5 hints):**
1. Domain/Category - Broad context, low specificity
2. Conceptual Association - Related area or function, still vague
3. Analogical/Riddle - Engages reasoning, indirect clues
4. Contradictory/Exclusion - Narrows down by removing confusion
5. Functional/Descriptive - Describes use or purpose clearly
6. Synonym/Semantic - Closely related meaning, near answer
7. Cultural/Trivia Reference - Pop culture or famous figure connection
8. Etymological/Origin - Reveals linguistic structure or root meaning
9. Pattern/Structural - Part of the word itself (letters, syllables, rhyme)
10. Partial Reveal - Letter position

**Rules:**
- Never include the target word, its root, or any substring (except in Hint 5 if using structural clue)
- Each hint ≤ 15 words
- Use diverse clue styles - don't repeat the same hint type
- Each hint must make the word easier to guess than the previous one
- Maintain emotional progression: curiosity → closeness → doubt → near-mastery → confirmation
- Tone: smart, conversational, rewarding
- Target: skilled English speakers who enjoy proving vocabulary depth

**Output Format (JSON only, no explanations):**
{
  "word": "<WORD>",
  "word_type": "adjective|verb|noun|abstract_noun|technical|idiomatic",
  "hints": [
    {"order": 1, "type": "domain/category or conceptual", "text": "..."},
    {"order": 2, "type": "analogical/functional", "text": "..."},
    {"order": 3, "type": "contradictory/exclusion", "text": "..."},
    {"order": 4, "type": "semantic/cultural", "text": "..."},
    {"order": 5, "type": "etymological/structural", "text": "..."}
  ]
}

Generate hints for this word: <WORD>
```

---

## 🌍 Domain-Specific Prompt Variations

### Geography Domain
- **Hint 1:** Continental/regional context
- **Hint 2:** Climate, culture, or economic clues
- **Hint 3:** Neighboring countries/exclusions
- **Hint 4:** Famous landmarks or products
- **Hint 5:** Capital city or coordinates hint

### Science Domain
- **Hint 1:** Field of study (biology, chemistry, physics)
- **Hint 2:** Function or application in real world
- **Hint 3:** What it's NOT (common misconceptions)
- **Hint 4:** Discoverer or famous equation/formula
- **Hint 5:** Symbol, atomic number, or structural hint

### Pop Culture Domain
- **Hint 1:** Era or medium (film, music, TV)
- **Hint 2:** Theme or genre association
- **Hint 3:** Similar but different references
- **Hint 4:** Director/creator or award won
- **Hint 5:** Release year or iconic quote

---

## 💾 Database Schema

### Firestore Structure
```
/games/{domain}/
  └── {date_YYYYMMDD}/
      ├── entity: "gregarious"
      ├── entity_type: "adjective"
      ├── difficulty: "medium"
      ├── hints: [array of 5 hint objects]
      ├── generated_at: timestamp
      ├── version: "1.0"
      └── metadata: {
           prompt_version: "vocab_v1",
           model: "gpt-4-turbo",
           validated: true
         }

/user_games/{user_id}/
  └── {game_id}/
      ├── domain: "vocabulary"
      ├── date: "20250103"
      ├── guesses: ["gregarious", "outgoing"]
      ├── hints_revealed: 4
      ├── solved: true
      ├── attempts: 2
      ├── score: 85
      ├── completed_at: timestamp
```

### DynamoDB Alternative
```
Table: GameHints
PK: domain#date (e.g., "vocabulary#20250103")
SK: entity_id

Table: UserProgress
PK: user_id
SK: domain#date
Attributes: guesses, hints_revealed, score, etc.
```

---

## 🎮 Gameplay Mechanics

### Scoring System
- **Guess on Hint 1:** 100 points (Master)
- **Guess on Hint 2:** 80 points (Expert)
- **Guess on Hint 3:** 60 points (Proficient)
- **Guess on Hint 4:** 40 points (Competent)
- **Guess on Hint 5:** 20 points (Learner)
- **Failed (6+ attempts):** 0 points

### Progressive Reveal
1. Player sees Hint 1
2. Input guess → Check answer
3. If wrong: Reveal Hint 2
4. Repeat until correct or 6 attempts
5. After game: Show all 5 hints + educational info

### Statistics Tracking
- Current streak (days played)
- Average hints needed
- Total games played by domain
- Mastery distribution chart
- Best domain performance

---

## 🚀 Implementation Roadmap

### Phase 1: MVP - Vocabulary Domain
- [ ] Set up API endpoint for OpenAI integration
- [ ] Implement validation layer for hint quality
- [ ] Create Firebase/DynamoDB schema
- [ ] Build hint generation service
- [ ] Modify frontend to show progressive hints
- [ ] Add guess submission and validation
- [ ] Implement scoring system
- [ ] Basic statistics page

### Phase 2: Enhanced Features
- [ ] Add Geography domain
- [ ] Add Science domain
- [ ] User authentication and progress tracking
- [ ] Daily puzzle automation
- [ ] Streak tracking and notifications
- [ ] Social sharing with spoiler protection
- [ ] Admin panel for word curation

### Phase 3: Monetization
- [ ] Free tier: 1 domain (vocabulary)
- [ ] Premium: All domains unlocked
- [ ] Educational tier: Classroom analytics
- [ ] Corporate tier: Custom domain creation
- [ ] API access for third-party integration

### Phase 4: Scale & Polish
- [ ] Mobile apps (React Native)
- [ ] Multiplayer race mode
- [ ] Community word suggestions
- [ ] Leaderboards by domain
- [ ] Achievement system
- [ ] Accessibility improvements

---

## 🔧 Technical Stack

### Current (Existing Wordle Clone)
- **Frontend:** React 17, TypeScript, Tailwind CSS
- **Build:** React Scripts (CRA)
- **Styling:** Tailwind + PostCSS
- **State:** React Hooks + Context API
- **Storage:** LocalStorage (current)
- **Database:** Firebase already configured

### Required Additions
- **Backend:** Node.js/Express API or Firebase Cloud Functions
- **AI:** OpenAI API (GPT-4 or GPT-4-turbo)
- **Database:** Firestore (preferred, already integrated) or DynamoDB
- **Validation:** Custom validation layer + Zod/Joi schema
- **Caching:** Redis (optional, for hint pre-generation)
- **Deployment:** Vercel (frontend) + Cloud Functions (backend)

---

## 💰 Business Model

### Target Markets

**B2C - Individual Learners**
- Free: Vocabulary domain, 1 game/day
- Premium ($4.99/mo): All domains, unlimited games, ad-free
- Annual: $39.99/year (2 months free)

**B2B - Educational Institutions**
- Classroom tier: $99/month (up to 50 students)
- School tier: $299/month (unlimited students)
- Features: Progress tracking, curriculum alignment, custom word lists

**B2B - Corporate Training**
- Professional tier: $199/month per team
- Enterprise: Custom pricing
- Features: Industry-specific domains (legal, medical, technical), analytics dashboard

### Revenue Projections (Conservative)
- **Year 1:** 10,000 users → 500 premium ($2,500/mo) + 5 schools ($1,495/mo) = ~$4,000/mo
- **Year 2:** 50,000 users → 2,500 premium ($12,500/mo) + 25 schools/corps ($7,500/mo) = ~$20,000/mo
- **Year 3:** 200,000 users → Scale to $50k-100k/mo with enterprise deals

---

## 🎯 Competitive Advantages

### vs. Wordle
- **Educational value:** Learn while playing, not just guess letters
- **Multiple domains:** Not limited to 5-letter words
- **Progressive difficulty:** Rewards expertise, not just luck
- **Replayability:** Multiple domains = multiple daily games

### vs. Duolingo/Educational Apps
- **Engaging format:** Game-first, education-second (no boring drills)
- **Mastery measurement:** Score reflects actual knowledge depth
- **Daily habit:** One puzzle per domain keeps commitment low
- **Social proof:** Share scores without spoiling answers

### Unique Selling Points
1. **AI-powered adaptive hints** - Never runs out of content
2. **Domain flexibility** - Easy to add new categories
3. **Emotional progression** - Psychologically engaging hint sequence
4. **Mastery-based scoring** - Not just completion, but efficiency
5. **Cross-domain learning** - Vocabulary + geography + science in one app

---

## 📊 Success Metrics

### MVP Success (3 months)
- 1,000 daily active users
- 70% completion rate on vocabulary games
- Average 3.5 hints per game (shows good difficulty balance)
- 50 paying premium users

### Product-Market Fit (6 months)
- 10,000 daily active users
- 3+ domains launched
- 5% conversion to premium
- 30+ day retention > 40%
- NPS score > 50

### Growth Phase (12 months)
- 50,000 daily active users
- 10 schools/corporations as B2B customers
- $20k+ MRR
- Featured in education/gaming publications

---

## 🛠️ Development Priorities

### Must-Have (MVP)
1. OpenAI API integration with vocabulary prompt
2. Hint validation system (no word leakage)
3. Progressive hint reveal UI
4. Guess submission and checking
5. Basic scoring and statistics
6. Firebase storage for hints

### Should-Have (Post-MVP)
1. User authentication
2. Streak tracking
3. Geography domain
4. Social sharing
5. Mobile-responsive improvements

### Nice-to-Have (Future)
1. Multiple domains
2. Multiplayer mode
3. Custom word lists
4. Admin content curation panel
5. API for third parties

---

## 🚧 Known Challenges & Solutions

### Challenge 1: AI Hint Quality Consistency
- **Problem:** GPT may generate hints that leak the answer or vary in quality
- **Solution:** Strong prompt engineering + validation layer + human review for daily puzzles
- **Fallback:** Cache pre-generated hints for popular words

### Challenge 2: Cost Management (OpenAI API)
- **Problem:** API costs can scale with users
- **Solution:** Pre-generate hints daily (1 API call per domain per day)
- **Alternative:** Use GPT-3.5-turbo for testing, GPT-4 for production

### Challenge 3: Answer Validation
- **Problem:** Multiple spellings, synonyms might be valid
- **Solution:** Use word lemmatization + maintain accepted alternatives list
- **Enhancement:** Partial credit for close answers

### Challenge 4: Keeping Content Fresh
- **Problem:** Running out of interesting words/entities
- **Solution:** Community submissions + curated lists by difficulty + AI variation generation

### Challenge 5: Preventing Cheating
- **Problem:** Players could use external tools to guess
- **Solution:** Time limits (optional), social pressure (streaks), focus on learning vs winning

---

## 📝 Next Steps When Resuming

### Immediate Actions
1. **Set up OpenAI API key** in environment variables
2. **Create API endpoint** for hint generation (Cloud Function or Express)
3. **Build validation module** with test cases
4. **Design database schema** in Firebase
5. **Modify Grid component** to show hints instead of letter tiles
6. **Implement guess checking** logic
7. **Add scoring calculation** 
8. **Test with 10-20 vocabulary words**

### Code Structure to Create
```
/reactwordle/
  /src/
    /api/
      - openai.ts (API calls)
      - validation.ts (hint quality checks)
    /services/
      - hintGenerator.ts (main logic)
      - gameManager.ts (guess checking, scoring)
    /components/
      /hints/
        - HintDisplay.tsx
        - HintReveal.tsx
      /game/
        - GuessInput.tsx
        - ScoreDisplay.tsx
    /lib/
      - prompts.ts (domain-specific prompts)
      - scoring.ts (scoring algorithms)
    /types/
      - hint.types.ts
      - game.types.ts
```

---

## 💡 Ideas for Future Expansion

- **Collaborative mode:** Teams guess together
- **Time trial:** Speed-based scoring
- **Themed weeks:** All hints related to a topic
- **Hint marketplace:** Users create/sell hint packs
- **AI tutor mode:** Explains why answer is correct
- **Language learning:** Same concept for foreign languages
- **Kids version:** Age-appropriate words and hints
- **Accessibility mode:** Audio hints for visually impaired
- **Tournament mode:** Compete in timed rounds

---

## 📚 Resources & References

### Useful Links
- OpenAI API Docs: https://platform.openai.com/docs
- Firebase Firestore: https://firebase.google.com/docs/firestore
- React TypeScript Patterns: https://react-typescript-cheatsheet.netlify.app/
- Prompt Engineering Guide: https://www.promptingguide.ai/

### Inspiration
- Original Wordle: https://www.nytimes.com/games/wordle
- Semantle (semantic similarity): https://semantle.com/
- Worldle (geography): https://worldle.teuteuf.fr/
- Nerdle (math): https://nerdlegame.com/

---

## ✅ Validation Checklist (Before Launch)

- [ ] OpenAI API generates valid JSON consistently
- [ ] No word leakage in hints (tested with 100+ words)
- [ ] Progressive difficulty actually increases guess success rate
- [ ] Scoring system feels fair and motivating
- [ ] Mobile experience is smooth (touch inputs work)
- [ ] Firebase costs are within budget projections
- [ ] Game state persists correctly across sessions
- [ ] Statistics accurately track user progress
- [ ] Social sharing doesn't spoil answers
- [ ] Accessibility standards met (WCAG 2.1 AA)

---

**Last Updated:** 2025-11-03  
**Status:** Planning Phase  
**Next Review:** When ready to start implementation

---

## 💭 Final Thoughts

This project transforms a saturated Wordle clone into a unique educational gaming platform with genuine market value. The progressive hint system powered by AI is defensible intellectual property, and the multi-domain approach creates natural expansion opportunities.

The key to success is maintaining hint quality through rigorous validation and focusing on the emotional journey of players as they progress through hints. This isn't just about guessing words—it's about demonstrating mastery and learning through an engaging daily ritual.

**Value Proposition in One Sentence:**  
*"Test and improve your vocabulary mastery by guessing words from AI-generated progressive hints—the earlier you guess, the more you prove you know."*

Start with vocabulary, nail the experience, then expand to other domains. The foundation you've built with the Wordle clone gives you a significant head start on the UI/UX, so focus on the hint generation and validation system as the key differentiator.
