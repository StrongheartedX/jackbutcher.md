# How to build your own writing profile

This documents the exact process used to create `jackbutcher.md` from a Twitter archive. Follow these steps to build a writing profile for any voice.

## 1. Download your Twitter archive

Go to Settings > Your Account > Download an archive of your data on X/Twitter. It takes 24-48 hours. You'll get a `.zip` file.

The file you need is inside:

```
data/tweets.js
```

This is a JavaScript file, not JSON. It starts with:

```js
window.YTD.tweets.part0 = [ ... ]
```

## 2. Pre-process the archive

The raw archive contains every tweet you've ever posted, including retweets, replies, and low-signal noise. Build a script to filter it down.

Strip:
- Retweets (text starts with "RT @")
- Pure replies (text starts with "@")
- URL-only tweets (just a link, no commentary)
- Tweets with no text (image-only posts)

Extract these fields per tweet:
- `id`
- `full_text` (the tweet body)
- `created_at` (timestamp)
- `favorite_count` (likes)
- `retweet_count` (retweets)

Sort by `favorite_count` descending. Output as compact JSON.

Example script (TypeScript):

```ts
import { readFileSync, writeFileSync } from 'fs'

const raw = readFileSync('data/tweets.js', 'utf-8')
const json = raw.replace(/^window\.YTD\.tweets\.part0\s*=\s*/, '')
const tweets = JSON.parse(json)

const filtered = tweets
  .map((t: any) => t.tweet)
  .filter((t: any) => {
    const text = t.full_text
    if (text.startsWith('RT @')) return false
    if (text.startsWith('@')) return false
    const stripped = text.replace(/https?:\/\/\S+/g, '').trim()
    if (!stripped) return false
    return true
  })
  .map((t: any) => ({
    id: t.id,
    text: t.full_text.replace(/https?:\/\/\S+/g, '').trim(),
    date: t.created_at,
    likes: parseInt(t.favorite_count),
    rts: parseInt(t.retweet_count),
  }))
  .sort((a: any, b: any) => b.likes - a.likes)

writeFileSync('tweet-index.json', JSON.stringify(filtered))
console.log(`${filtered.length} tweets indexed`)
```

For reference: 50,796 tweets filtered down to 13,962.

## 3. Quantify the voice

Feed the top 300-500 tweets to an AI and run these analyses. Each one builds a section of the profile.

### 3a. Shape

Prompt:

```
Here are my top 500 tweets sorted by engagement. Calculate:
- Median word count per tweet
- % of tweets under 10 words
- % of tweets under 20 words
- % that are a single sentence
- Average word count by engagement tier (top 100 vs bottom 100)
- Correlation between length and engagement
```

### 3b. Perspective

Prompt:

```
Analyze pronoun usage across these tweets:
- What % contain "you" or "your"?
- What % contain "I" or "my"?
- What % contain "we" or "our"?
- In the top 100 tweets specifically, what are these ratios?
```

### 3c. Punctuation fingerprint

Prompt:

```
Count the average per tweet:
- Periods
- Commas
- Line breaks / newlines
- Question marks
- Exclamation points
- Colons
- Semicolons

Also: what % of tweets end with a period vs no punctuation?
Compare engagement for each.
```

### 3d. Structure templates

Prompt:

```
Classify these tweets by structure:
- Single sentence
- Two-part parallel ("X does A. Y does B.")
- Conditional ("If X, then Y.")
- Numbered list ("1. Do X. 2. Do Y.")
- Explicit contrast ("X vs. Y")
- Other

Give percentages for each category.
```

### 3e. Opening patterns

Prompt:

```
Classify the first word or phrase of each tweet:
- Observation/declaration (starts with a noun or statement)
- Numbered list (starts with "1.")
- Conditional (starts with "If" or "When")
- Imperative verb (starts with a command)
- Quote (starts with a direct quote)
- Question (starts with or is a question)

Give percentages.
```

### 3f. Verb mood

Prompt:

```
Classify each tweet by its dominant verb mood:
- Declarative (statement of fact)
- Imperative (command)
- Conditional (if/when)
- Interrogative (question)

Give percentages. Compare average engagement for each mood.
```

## 4. Extract rhetorical patterns

This is the qualitative layer. Feed the top 300 tweets and prompt:

```
Identify the recurring rhetorical moves in these tweets. For each move:
- Name it
- Describe the mechanics
- Give 3-5 examples from the data

Look for: contrast pairs, reframes, quantification of abstract ideas,
humor through understatement, uncomfortable truths, compressed frameworks.
```

## 5. Extract word-level mechanics

These are the micro-level devices. Run each as a separate analysis.

### 5a. Alliterative contrasts

```
Find every tweet where two compared or contrasted concepts start with
the same letter. Example: "Complexity" vs "Clarity" both start with C.

List every instance with the exact tweet text and which letter pair is used.
```

### 5b. Sound devices

```
Find examples of:
- Matched meter: couplets where both lines have equal syllable counts
- Chiasmus: A-B structure flips to B-A
- Circular loops: ending returns to beginning
- Internal rhyme: sound echoes within or across lines
- Negation flips: same words with "don't" added/removed
- Paradox: self-contradictory statements that reveal truth

Give exact tweet text for each.
```

### 5c. Closing patterns

```
Analyze how these tweets END:
- What are the 20 most common final words?
- What part of speech is the final word usually?
- Do tweets end with a period or no punctuation? Compare engagement.
- Are there structural closing patterns (punchline inversion, imperative, etc.)?
```

### 5d. Contrast frames

```
Find every tweet that contains a contrast. Classify the SYNTACTIC FRAME:
- "X does A. Y does B." (parallel declaration)
- "X, not Y." (negation)
- Single-sentence reframe
- "If X, then Y." (conditional reveal)
- Two words, no verb (juxtaposed pair)
- Numbered progression
- "X vs Y" (explicit versus)
- Any other frames

Rank by frequency. Give 3+ examples of each.
```

## 6. Map the negative space

What the voice never says is as important as what it does say.

```
Read through 500 tweets and identify categories of content that NEVER
or almost never appear. Check for:
- Personal emotions ("I feel...")
- Apologies or self-correction
- Current events or news commentary
- Personal biographical details
- Engagement asks (like/share/follow)
- Gratitude performances ("so grateful...")
- Vulnerability theater ("I'll be honest...")
- Motivational cliches ("believe in yourself")
- Own success metrics (revenue, followers)
- Pop culture references
- Political opinions
- Religious references
- Complaining about others

For each, confirm whether truly absent or count rare exceptions.
```

## 7. Build the banned words list

```
Based on the voice you've analyzed, generate a list of words and phrases
that this voice would NEVER use. Think about:
- Corporate jargon
- Filler phrases
- Hedging language
- Buzzwords
- Transition words that add no meaning

This should be the words that would break the voice if they appeared.
```

## 8. Create rewrite pairs

Pick 10 ideas that appear in the tweet archive. Write each one two ways:

1. How a generic writer would say it (long, hedging, jargon-filled)
2. How this voice actually said it (compressed, direct, no fat)

These pairs teach the compression pattern by example. They show an AI the gap between default writing and the target voice.

## 9. Select reference tweets

Pull 20-30 of the highest-performing tweets that best represent the voice. These serve as few-shot examples. Choose for variety across the rhetorical moves and structural patterns you've identified.

## 10. Assemble the file

Combine everything into a single markdown file. Recommended section order:

1. Voice (one-paragraph description)
2. Numbers (shape of the writing)
3. Perspective (pronoun ratios)
4. Punctuation (fingerprint)
5. Structure templates
6. What performs (engagement x length)
7. Sentence structure
8. Rhetorical patterns
9. Word-level mechanics
10. Closing patterns
11. Contrast frames
12. Verb mood
13. Colon as pivot (or other punctuation devices)
14. What the voice never says
15. Opening patterns
16. Signature vocabulary
17. Tone
18. Themes
19. Banned words
20. Anti-patterns (what he never does)
21. Long-form format rules
22. Rewrite pairs
23. Reference tweets

## Notes

- The more data you have, the better. 1,000 tweets is a minimum. 10,000+ gives you statistical confidence.
- Engagement data is critical. It tells you what the audience responds to vs what you just happened to write.
- Run each analysis separately. Trying to extract everything in one prompt produces shallow results.
- Validate findings with the author. Some patterns are intentional. Some are accidents. The file should only contain the intentional ones.
- The file is never finished. As you discover new patterns, add them.
