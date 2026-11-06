# Beginner Tutorial: Shreevathsa Bhat's AI Portfolio

This tutorial explains how the portfolio website is organized, how the pages are rendered, how the Digital Twin chat works, and how the design is implemented.

## 1. What We Built

This is a personal portfolio website for Shreevathsa Bhat. It contains:

- A homepage at `/`
- About and career journey sections
- A future-ready portfolio page at `/portfolio`
- LinkedIn and email contact links
- An AI Digital Twin chat
- Responsive layouts for desktop and mobile
- Animated visual effects and accessibility support

The project uses the information extracted from `LinkedIn.pdf` as its factual source. Personal data is kept in one shared file so it can be updated without searching through every page.

## 2. Technology Summary

### Next.js

Next.js is a framework built on React. It provides routing, server rendering, API routes, metadata, and production tooling.

This project uses the Next.js App Router. In the App Router, folders and files inside `app/` define routes:

```text
app/page.tsx                 -> /
app/portfolio/page.tsx       -> /portfolio
app/api/chat/route.ts        -> POST /api/chat
```

### React

React lets us build the interface from reusable components. Components return JSX, which looks like HTML but can include JavaScript expressions.

React state is used by the Digital Twin chat to remember messages, the current input, loading status, and errors.

### TypeScript

TypeScript adds types to JavaScript. Types make the code easier to understand and help catch mistakes before the application runs.

For example:

```ts
type Experience = {
  period: string;
  title: string;
  organization: string;
  description: string;
};
```

This says that every experience object must contain four text values.

### Lucide React

`lucide-react` provides the icons used by the interface, including arrows, the send icon, mail, and the Digital Twin sparkle icon.

### OpenRouter and Gemma

OpenRouter is an API service that provides access to AI models. The Digital Twin first uses:

```text
google/gemma-4-31b-it:free
```

If that free provider is rate-limited with HTTP 429, the server retries through OpenRouter's automatic free router:

```text
openrouter/free
```

### CSS

The project uses global CSS files rather than a component library:

- `app/globals.css` contains the main design system and layout.
- `app/motion.css` contains animations and hover effects.
- `app/portfolio.css` contains styles specific to the portfolio page.

The design uses Space Grotesk for display headings, Manrope for body text, and DM Mono for small labels and metadata.

## 3. Project Structure

```text
AI_Portforlio/
├── app/
│   ├── api/chat/route.ts       AI server endpoint
│   ├── portfolio/page.tsx      Portfolio route
│   ├── globals.css             Main styles
│   ├── motion.css              Animation styles
│   ├── portfolio.css           Portfolio styles
│   ├── layout.tsx              Shared document layout
│   └── page.tsx                Homepage
├── components/
│   ├── chat-panel.tsx          Interactive Digital Twin
│   ├── site-footer.tsx         Shared footer
│   └── site-header.tsx         Shared navigation
├── lib/
│   └── profile.ts              Shared profile and journey data
├── public/                     Static files
├── .env                        Local secrets; never commit this file
├── package.json                Scripts and dependencies
└── LinkedIn.pdf                Original profile source
```

## 4. High-Level Walkthrough

When a visitor opens the homepage, this is what happens:

1. Next.js receives a request for `/`.
2. `app/layout.tsx` provides the HTML document and page metadata.
3. The layout imports the three global style files.
4. `app/page.tsx` renders the hero, About section, journey, Digital Twin, CTA, header, and footer.
5. The page reads profile details from `lib/profile.ts`.
6. The browser hydrates `ChatPanel`, which is the interactive client component.
7. When the visitor asks a question, the browser sends a `POST` request to `/api/chat`.
8. The server validates the message list and reads `OPENROUTER_API_KEY` from `.env`.
9. The server adds verified profile information to an AI system prompt.
10. The server calls OpenRouter and returns only the assistant's text.
11. The chat panel adds the answer to the visible conversation.

The API key never needs to be sent to the browser. This is important because anything included in client-side code can potentially be inspected by visitors.

## 5. The Root Layout

The root layout is in `app/layout.tsx`:

```tsx
import type { Metadata } from "next";
import "./globals.css";
import "./motion.css";
import "./portfolio.css";

export const metadata: Metadata = {
  title: "Shreevathsa Bhat | Computer Science",
  description: "The professional portfolio of Shreevathsa Bhat, MCA candidate and computer science enthusiast.",
};

export default function RootLayout({ children }: LayoutProps<"/">) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

`metadata` controls the browser title and search-engine description. The `children` value represents whichever route Next.js is currently rendering.

## 6. Shared Profile Data

The profile data lives in `lib/profile.ts`:

```ts
export const profile = {
  name: "Shreevathsa Bhat",
  shortName: "SB",
  headline: "MCA candidate building a sharper edge for what comes next.",
  location: "Mangaluru, Karnataka, India",
  email: "shreevathsabhat2004@gmail.com",
  linkedin: "https://www.linkedin.com/in/shreevathsa-bhat-726470297",
  focusAreas: ["Computer science", "Product thinking", "Systems mindset"],
};
```

The homepage, header, footer, portfolio page, and API route all import this object. This avoids duplicating the same email address or LinkedIn URL in several places.

The career data is an array:

```ts
export const journey: Experience[] = [
  {
    period: "2026 — now",
    title: "MCA candidate",
    organization: "PES University",
    description: "Going deeper into computer science...",
  },
];
```

The `Experience[]` type means `journey` must be an array of valid experience objects.

## 7. Building Repeated Content with `map`

The homepage displays focus areas with `.map()`:

```tsx
{profile.focusAreas.map((area, index) => (
  <div className="focus-item" key={area}>
    <span>0{index + 1}</span>
    {area}
  </div>
))}
```

`map()` runs once for every item in the array. React requires a stable `key` so it can identify each rendered item efficiently.

The career journey uses the same approach:

```tsx
{journey.map((entry, index) => (
  <article className="journey-item" key={entry.organization}>
    <span>{entry.period}</span>
    <h3>{entry.title}</h3>
    <p>{entry.description}</p>
  </article>
))}
```

This makes adding another education or career entry simple: add another object to `journey`.

## 8. The Digital Twin Client Component

`components/chat-panel.tsx` begins with:

```tsx
"use client";
```

By default, App Router components are server components. The chat must be a client component because it uses browser interactivity and React state.

The state values are:

```tsx
const [messages, setMessages] = useState<Message[]>([]);
const [input, setInput] = useState("");
const [isLoading, setIsLoading] = useState(false);
const [error, setError] = useState("");
```

They represent:

- `messages`: the conversation history
- `input`: the text currently in the input field
- `isLoading`: whether an AI request is in progress
- `error`: a user-facing error message

The form submits without refreshing the page:

```tsx
function handleSubmit(event: FormEvent<HTMLFormElement>) {
  event.preventDefault();
  void ask(input);
}
```

`preventDefault()` stops the browser's normal form navigation. The `ask()` function sends the request with `fetch()`:

```tsx
const response = await fetch("/api/chat", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ messages: nextMessages }),
});
```

The frontend calls our own API route, not OpenRouter directly. This keeps the API key on the server.

## 9. The API Route

The server endpoint is `app/api/chat/route.ts`:

```ts
export async function POST(request: Request) {
  const body = await request.json() as { messages?: unknown };

  if (!isValidMessages(body.messages)) {
    return Response.json(
      { error: "Please send a valid conversation." },
      { status: 400 },
    );
  }

  const apiKey = process.env.OPENROUTER_API_KEY;

  if (!apiKey) {
    return Response.json(
      { error: "The digital twin is not configured yet." },
      { status: 503 },
    );
  }
}
```

The `POST` function handles requests from the chat client. It first parses JSON, validates it, and checks that the secret exists.

The validation function limits the request:

```ts
function isValidMessages(value: unknown): value is ChatMessage[] {
  return Array.isArray(value)
    && value.length > 0
    && value.length <= 20
    && value.every((message) => {
      if (!message || typeof message !== "object") return false;
      const candidate = message as Partial<ChatMessage>;
      return (
        (candidate.role === "user" || candidate.role === "assistant")
        && typeof candidate.content === "string"
        && candidate.content.trim().length > 0
        && candidate.content.length <= 2000
      );
    });
}
```

This prevents empty conversations and very large individual messages from being forwarded.

## 10. Grounding the AI in Profile Facts

The API constructs a system prompt containing verified profile information:

```ts
const systemPrompt = `You are the Digital Twin of ${profile.name}.
Answer questions using only the verified profile below.
Never invent employers, projects, technologies, achievements, or dates.
When information is unavailable, invite the visitor to connect on LinkedIn.`;
```

The profile data is then sent to OpenRouter along with the conversation:

```ts
body: JSON.stringify({
  model: "google/gemma-4-31b-it:free",
  provider: { allow_fallbacks: true, sort: "throughput" },
  messages: [
    { role: "system", content: systemPrompt },
    ...body.messages,
  ],
}),
```

The system prompt is an instruction to the model. The conversation messages are the visitor's questions and previous answers.

If the requested model returns HTTP 429, the route makes a second request through OpenRouter's free router:

```ts
if (upstream.status === 429) {
  response = await fetch("https://openrouter.ai/api/v1/chat/completions", {
    method: "POST",
    headers,
    body: JSON.stringify({
      model: "openrouter/free",
      messages: [{ role: "system", content: systemPrompt }, ...body.messages],
    }),
  });
}
```

This is why the chat can continue working even if the specific Gemma provider pool is temporarily full.

## 11. The Portfolio Page

The `/portfolio` route is in `app/portfolio/page.tsx`. It is currently an honest “work in progress” portfolio rather than a page containing invented projects.

Its directions are data-driven:

```ts
const directions = [
  {
    number: "01",
    title: "Computer science",
    detail: "Deepening the fundamentals through an MCA in Computer Science at PES University.",
  },
  {
    number: "02",
    title: "Product thinking",
    detail: "Exploring how clear interfaces and thoughtful systems turn ideas into useful experiences.",
  },
];
```

A future project can be added as another data object without redesigning the entire page.

## 12. Styling and Animations

CSS custom properties define the design tokens:

```css
:root {
  --ink: #111318;
  --paper: #f1f1ed;
  --acid: #d8f653;
  --orange: #ff765b;
  --display: 'Space Grotesk', sans-serif;
  --body: 'Manrope', sans-serif;
}
```

Instead of repeating a color everywhere, the stylesheet uses `var(--orange)`. This makes a future color change much easier.

The layout uses CSS Grid for large content areas:

```css
.hero-grid {
  display: grid;
  grid-template-columns: 1.15fr 0.85fr;
}
```

At smaller screen sizes, media queries switch the layout to a single column:

```css
@media (max-width: 720px) {
  .hero-grid {
    display: block;
  }
}
```

Animations are kept in `app/motion.css`. For example:

```css
@keyframes rise {
  from {
    opacity: 0;
    transform: translateY(20px);
  }

  to {
    opacity: 1;
    transform: none;
  }
}
```

The stylesheet also respects visitors who prefer less motion:

```css
@media (prefers-reduced-motion: reduce) {
  .site-shell * {
    animation: none !important;
    transition: none !important;
  }
}
```

This is important for accessibility. A polished site should not force animation on someone who has disabled it in their operating system or browser.

## 13. Accessibility Details

The implementation includes several accessibility features:

- Semantic elements such as `header`, `nav`, `main`, `section`, `article`, and `footer`
- Heading IDs connected with `aria-labelledby`
- A labeled chat input
- `aria-live="polite"` so new chat responses can be announced
- `role="alert"` for chat errors
- Visible `:focus-visible` outlines for keyboard users
- Decorative icons marked with `aria-hidden="true"`
- Reduced-motion support
- External links use `target="_blank"` with `rel="noreferrer"`

## 14. Environment Variables and Security

The root `.env` file contains the OpenRouter secret:

```env
OPENROUTER_API_KEY=your_key_here
```

The real key should never be pasted into a client component, committed to Git, or placed in a variable beginning with `NEXT_PUBLIC_`.

The `.gitignore` file ignores `.env`. For a team project, it is useful to create a safe `.env.example` containing only the variable name:

```env
OPENROUTER_API_KEY=
```

A developer can copy `.env.example` to `.env` and add their own key locally.

If a key is ever committed or exposed publicly, revoke it and create a replacement in OpenRouter.

## 15. Running the Website

From the project root, install dependencies:

```bash
npm install
```

Start the development server:

```bash
npm run dev
```

Open:

```text
http://localhost:3000
```

Useful commands:

```bash
npm run lint   # Check code style and common mistakes
npm run build  # Create a production build
npm run start  # Run the production build
```

There is currently no automated test script, so manually check both `/` and `/portfolio`, submit a Digital Twin question, resize the browser, and test keyboard navigation.

## 16. Editing the Website

To change personal information, edit `lib/profile.ts`.

To change homepage structure or visible homepage text, edit `app/page.tsx`.

To change the portfolio page content, edit `app/portfolio/page.tsx`.

To change the navigation, edit `components/site-header.tsx`.

To change the footer contact links, edit `components/site-footer.tsx`.

To change the chat behavior or interface, edit `components/chat-panel.tsx` and `app/api/chat/route.ts`.

To change colors, spacing, responsive layout, or animations, edit the CSS files in `app/`.

## 17. Detailed Code Review Summary

### Good design decisions

1. Profile data is centralized in `lib/profile.ts`, which reduces duplication.
2. The API key is read only in the server route.
3. Chat input is validated before being sent to OpenRouter.
4. The AI prompt explicitly prevents invented career facts.
5. The chat has loading, empty, and error states.
6. The UI is responsive and supports reduced motion.
7. Header and footer are reusable components.
8. Repeated lists use arrays and `.map()` rather than duplicated markup.
9. External links use safe `rel="noreferrer"` attributes.
10. The portfolio page avoids fabricating projects that do not exist yet.

### Important code path

```text
Visitor asks a question
        |
        v
ChatPanel.ask()
        |
        v
POST /api/chat
        |
        v
Validate messages
        |
        v
Read OPENROUTER_API_KEY on server
        |
        v
Add verified profile system prompt
        |
        v
Call Gemma through OpenRouter
        |
        v
Retry through openrouter/free on HTTP 429
        |
        v
Return { message }
        |
        v
Render assistant response in ChatPanel
```

This separation is a common frontend pattern: the browser handles presentation and interaction, while the server handles secrets and communication with protected external services.

## 18. Five Self-Review Suggestions

1. **Preserve failed user questions.** In the chat error path, review whether restoring the previous `messages` value causes the submitted question to disappear. Keeping the failed question visible may make retrying clearer.

2. **Strengthen conversation validation.** Review whether client-supplied assistant messages should be accepted without additional checks. A production API could enforce a valid user/assistant message sequence.

3. **Add abuse protection.** Add rate limiting, request logging, and a total token or conversation budget before deploying the public `/api/chat` endpoint.

4. **Remove deployment-specific values.** Make the OpenRouter `HTTP-Referer` configurable instead of hard-coding `http://localhost:3000`, so production analytics and provider requirements stay accurate.

5. **Add automated tests and remove leftovers.** Add tests for successful responses, invalid input, missing configuration, rate-limit fallback, and upstream failure. Also review whether unused starter files such as `app/page.module.css` and the unused `pdf-parse` dependency should be removed.
