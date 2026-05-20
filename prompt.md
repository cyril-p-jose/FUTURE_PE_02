Build a web app called "LocalBiz UGC AI" — an AI-powered prompt playground that generates UGC ad hooks, full scripts, and CTAs for local businesses of any kind (cafes, barbers, gyms, florists, etc.). Use TanStack Start v1 with React 19, Tailwind CSS v4, shadcn/ui components, and the Lovable AI Gateway via the Vercel AI SDK.

Tech Stack
TanStack Start v1 (file-based routing, no src/pages/)
Tailwind CSS v4 with @theme inline and oklch colors
sonner for toast notifications
jspdf for PDF exports
@ai-sdk/openai-compatible and ai for AI generation
lucide-react for icons
Design System
Define a light-mode-first design system in src/styles.css using these exact tokens:

--radius: 1rem
--background: oklch(0.985 0.008 255)
--foreground: oklch(0.22 0.04 260)
--primary: oklch(0.50 0.16 260)
--primary-glow: oklch(0.70 0.14 255)
--accent: oklch(0.75 0.10 195)
--secondary: oklch(0.95 0.025 250)
--muted-foreground: oklch(0.5 0.03 255)
--border: oklch(0.9 0.02 250)
--gradient-hero: linear-gradient(135deg, oklch(0.50 0.16 260), oklch(0.70 0.14 255) 55%, oklch(0.80 0.10 195))
--shadow-glow: 0 20px 60px -20px color-mix(in oklab, oklch(0.50 0.16 260) 35%, transparent)
--shadow-card: 0 8px 30px -12px color-mix(in oklab, oklch(0.50 0.16 260) 18%, transparent)
Add utility classes:

.text-gradient — clips the hero gradient to text
.bg-gradient-hero — applies the hero gradient
.shadow-glow / .shadow-card
Body background: a subtle fixed radial gradient from primary-glow (top-left) and accent (top-right), plus font-feature-settings: "ss01", "cv11".

Routes
src/routes/__root.tsx — Root layout with <QueryClientProvider>, HeadContent, Scripts. Add notFoundComponent and errorComponent that both show centered messages with a "Go home" / "Try again" button. Meta tags: og:type=website, og:site_name=LocalBiz UGC AI, twitter:card=summary_large_image, twitter:site=@LocalBizUGC.

src/routes/index.tsx — The main page. Add a head() with:

Title: "LocalBiz UGC AI — Prompt Playground for Local Businesses"
Description: "AI-powered UGC ad playground that generates Instagram Reels hooks, scripts, and CTAs for local businesses of any kind."
og:title, og:description, og:url=https://localbiz-ugc-ai.lovable.app/
Canonical link to https://localbiz-ugc-ai.lovable.app/
JSON-LD WebApplication schema with applicationCategory: BusinessApplication, price: 0, priceCurrency: USD
src/routes/api/generate.ts — Server route at /api/generate. POST handler:

Accepts JSON body: { kind: "hooks"|"script"|"ctas", business, type, audience, service, platform, tone }
Validates required fields (kind, business, service)
Reads LOVABLE_API_KEY from process.env
Creates a Lovable AI Gateway provider via @ai-sdk/openai-compatible with:
baseURL: "https://ai.gateway.lovable.dev/v1"
headers: { "Lovable-API-Key": key, "X-Lovable-AIG-SDK": "vercel-ai-sdk" }
Uses model google/gemini-3-flash-preview
Builds prompts per kind:
hooks: system = "You are a professional UGC ad copywriter for social media." Prompt asks for 10 short natural-feeling hooks as a numbered markdown list, each in quotation marks, no intro/outro.
script: system = "You are a professional UGC ad copywriter for social media marketing campaigns." Prompt asks for a 30s UGC script following Hook → Problem → Experience → Result → CTA. 5 short scenes, each labeled like "🎥 Scene 1 – Hook". End with "On-Screen Text".
ctas: system = "You are a professional UGC ad copywriter." Prompt asks for 8 short punchy CTAs under 8 words, friendly/modern, 1 emoji per line where natural, numbered markdown list.
Returns { text } on success. Handles 429 (rate limit) and 402 (credits) gracefully.
src/routes/sitemap[.]xml.ts — Server route returning XML sitemap with / entry (weekly, priority: 1.1.0).

public/robots.txt — User-agent: * Allow: / + Sitemap directive.

Page UI (/)
The page has a centered hero, a two-column playground (form left, output right), and a 3-card feature section.

Header: Logo icon (Sparkles) inside a gradient circle, app name "LocalBiz UGC AI", and a small pill badge "Prompt Engineering · 2026".

Hero: H1: "Generate <span className='text-gradient'>UGC ad scripts</span> that actually convert". Subtitle: "A prompt-engineered playground for local businesses of every kind. Produce Instagram Reels hooks, full UGC scripts, and conversion-focused CTAs in seconds."

Form card (left column, ~420px wide):

Section title "Campaign Brief" with Wand2 icon.
Fields (all blank by default, with grey placeholders):
Business name — placeholder: "e.g. The Corner Café"
Business type — placeholder: "e.g. Local Café & Coffee Shop"
Target audience (textarea, 2 rows) — placeholder: "e.g. Young professionals, students, remote workers"
Product / Service (textarea, 2 rows) — placeholder: "e.g. Signature cold brew and breakfast combo"
Platform (select) — options: Instagram Reels, TikTok, YouTube Shorts, Meta Ads. Default: Instagram Reels.
Tone — placeholder: "e.g. Natural, authentic, conversational"
Three generate buttons below the form in a row:
"Hooks" (Sparkles icon), "Ad Script" (Film icon, variant="hero"), "CTAs" (Megaphone icon).
Disabled while any generation is in progress; show Loader2 spinner on active button.
Small text below: "Powered by Lovable AI · Gemini 3 Flash. Prompts use the Hook → Problem → Solution → Result → CTA framework."
Output card (right column):

Tabs: Hooks | Ad Script | CTAs. Each tab has its icon.
Action buttons in the header: Copy, TXT, PDF. All disabled when no content.
Copy toggles a Check icon briefly with a toast.
TXT export — downloads a .txt file named {slug}-{tab}.txt with a header line including business name, tab label, platform, and service.
PDF export — uses jsPDF (A4, pt units) with business name + tab label as bold title, platform/service as subtitle in grey, and body text split to fit page width. Named {slug}-{tab}.pdf. Shows toast on success.
Output area states:
Loading: centered spinner with "Crafting {label}…"
Empty: centered gradient dashed box with Sparkles icon, "No {label} yet", subtext "Fill in the brief on the left and generate UGC content tailored to your business.", and a "Generate {Label}" button (variant="hero").
Content: rendered in a rounded box with prose prose-sm max-w-none whitespace-pre-wrap inside bg-secondary/40.
Feature cards (3-column row below):

"Human-Like Tone" — "Prompts instruct the AI to mimic real customer voice — never robotic, never salesy."
"Proven Framework" — "Every script: Hook → Problem → Solution → Result → CTA. Built for engagement."
"Platform Native" — "Optimized for Reels, Shorts, and Meta Ads — short attention spans, vertical video."
Footer: "Prompt Engineering Task 2 · LocalBiz UGC AI System · 2026"

Additional Notes
Use sonner <Toaster richColors position="top-center" />.
All form fields start blank — never pre-fill examples.
Ensure all shadcn components used (Button, Input, Label, Textarea, Tabs, Select) are properly styled and accessible.
The app should feel premium: rounded-3xl cards, backdrop blur, soft shadows, gradient accents, and generous whitespace.
