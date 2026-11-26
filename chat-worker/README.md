# Camp chat helper

A small chat on the camp website that answers questions about COR Church Camp 2027, in English or Chinese. It answers only from [`camp-facts.md`](../camp-facts.md):

- **Confirmed info**: it answers in 1 to 3 short sentences.
- **Camp topics that aren't announced yet**: "This hasn't been announced yet. We'll share it as soon as it's confirmed."
- **Anything else**: "Sorry, I'm unable to answer questions not related to COR Church Camp 2027."

Everything runs on free plans. Questions and answers are not saved anywhere.

## How it fits together

```
   VISITOR'S PHONE                            CLOUDFLARE (free plan)
  ┌────────────────────────────┐            ┌─────────────────────────────────┐
  │ Camp website               │  question  │ Worker "cor-camp-chat"          │
  │  ├ chat bubble             │ ─────────▶ │  ├ Rate limiting: 20 a minute   │
  │  ├ 4 quick-answer buttons  │            │  ├ Turnstile: is it a person?   │
  │  └ Turnstile robot check   │ ◀───────── │  └ Workers AI: GLM 4.7 Flash    │
  └──────────────┬─────────────┘   answer   └────────────────┬────────────────┘
                 ▲                                           ▲
                 │ website                                   │ fact sheet
                 │                                           │ (re-read every 5 min)
  ┌──────────────┴───────────────────────────────────────────┴────────────────┐
  │ GITHUB PAGES (free)   repo dannellim/corchurchcamp2027presence            │
  │   index.html      the website, with the chat bubble                       │
  │   camp-facts.md   the fact sheet: the only thing the AI knows             │
  │   chat-worker/    the Worker's code (sent to Cloudflare by Wrangler)      │
  └───────────────────────────────────────────────────────────────────────────┘
```

| Service | What it does | Cost |
|---|---|---|
| **GitHub Pages** | Hosts the website and the fact sheet | Free |
| **Cloudflare Worker** (`cor-camp-chat`) | The go-between: checks each question, adds the fact sheet and rules, asks the AI. Keeps the Turnstile secret key out of the webpage. | Free (100,000 requests a day) |
| **Cloudflare Rate Limiting** | At most 20 questions a minute from one connection | Part of the Worker |
| **Cloudflare Turnstile** | Checks the visitor is a person, not a script. Usually invisible. | Free |
| **Cloudflare Workers AI** (GLM 4.7 Flash) | Writes the answer | Free up to 10,000 "Neurons" a day, about 700 typed questions. Resets at 8am Singapore time. |

On the free plan nothing can be billed. If the day's allowance runs out, the chat says so and starts working again at 8am.

## The journey of one question

```
    Phone           Turnstile         Worker            GitHub Pages     Workers AI
  (website)       (robot check)   (cor-camp-chat)      (camp-facts.md)    (GLM 4.7)
      │                 │                │                    │               │
 1  Tapping a quick button: answered on the phone. Nothing is sent anywhere.
      │                 │                │                    │               │
 2    │──── human? ────▶│                │                    │               │
      │◀──── token ─────│                │                    │               │
      │                 │                │                    │               │
 3    │──────── question + token ───────▶│                    │               │
      │   + last 2 Q&As │                │                    │               │
      │                 │                │                    │               │
 4    │                 │                │ from camp site?    │               │
      │                 │                │ under 20 a minute? │               │
      │                 │                │ under 300 chars?   │               │
      │                 │◀─ token OK? ───│                    │               │
      │                 │───── yes ─────▶│                    │               │
      │                 │                │                    │               │
 5    │                 │                │─── fact sheet? ───▶│               │
      │                 │                │◀───── sheet ───────│               │
      │                 │                │ (kept 5 min)       │               │
      │                 │                │                    │               │
 6    │                 │                │───── rules + sheet + question ────▶│
      │                 │                │◀────────── short answer ───────────│
      │                 │                │                    │               │
 7    │◀──────────── answer ─────────────│                    │               │
      │                 │                │                    │               │
```

1. **Quick buttons** ("When is camp?", "Where is it?", "How much is it?", "How do I sign up?") are answered by the page itself. They are free and always correct.
2. For a typed question, the page first gets a one-time token from **Turnstile** to show a person is asking.
3. The page sends the question, the token and the last 2 questions and answers (for follow-ups like "how far is that?") to the **Worker**.
4. The Worker checks that the request came from the camp website, that the connection hasn't asked more than 20 questions this minute, that the question is under 300 characters, and, with **Turnstile**, that the token is real.
5. The Worker reads the fact sheet from **GitHub Pages**. It keeps a copy for 5 minutes. Notes inside `<!-- -->` are removed first.
6. The Worker sends the rules, the fact sheet and the question to **Workers AI**. Thinking is switched off and answers are capped at 250 tokens, to stretch the free allowance.
7. The answer goes back to the chat bubble.

## Updating the camp info

Edit [`camp-facts.md`](../camp-facts.md) and push to GitHub. The chat uses the new version within about 15 minutes. Nothing needs redeploying.

If you change something the quick buttons answer (dates, venue, prices, sign-up), also update the `FAQ` answers near the bottom of [`index.html`](../index.html).

## Changing how it answers

The rules and the two fixed replies are in [`src/index.js`](src/index.js). The model and settings are in [`wrangler.jsonc`](wrangler.jsonc). After changing either, run `npm run deploy` in this folder.

## One-time setup

1. In this folder, install the tools: `npm install`
2. Sign in to Cloudflare: `npx wrangler login` (opens your browser)
3. Create the robot check: in the Cloudflare dashboard, go to **Turnstile**, then **Add widget**. Name it "COR camp chat", add the hostname `dannellim.github.io` and choose **Managed**. Keep the **site key** and **secret key** page open.
4. Deploy the Worker: `npm run deploy`. It prints the Worker's address, like `https://cor-camp-chat.<your-name>.workers.dev`.
5. Give the Worker the secret key: `npx wrangler secret put TURNSTILE_SECRET`, then paste the secret key.
6. In [`index.html`](../index.html), set `CHAT_API` to the Worker's address and `TURNSTILE_SITEKEY` to the site key, then push. The "Ask about camp" button appears once both are set.

## Trying it on your own computer

1. Copy `.dev.vars.example` to `.dev.vars`. It holds Cloudflare's public test keys.
2. In this folder: `npm run dev` (the Worker, on port 8787). This uses the real Workers AI, so questions count toward the daily allowance.
3. In the `Website` folder, in another terminal: `python3 -m http.server 8000`, then open http://localhost:8000

`npm test` checks the Worker's logic with stand-ins for Cloudflare. It needs no account and no internet.

## Files

| File | What it is |
|---|---|
| `src/index.js` | The Worker: checks, rules, fixed replies, call to Workers AI |
| `wrangler.jsonc` | Worker settings: model, fact sheet address, allowed website, rate limit |
| `test/worker.test.js` | Logic tests (`npm test`) |
| `.dev.vars.example` | Test keys for trying it locally |

This folder is part of the website repo, so GitHub Pages publishes it too. That's fine: it holds code and Cloudflare's public test keys only. The real Turnstile secret key is stored in Cloudflare, never in a file.
