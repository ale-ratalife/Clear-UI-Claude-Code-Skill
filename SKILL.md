# Claude Local UI - Custom Skill & Codebase

This guide provides the instructions to create a custom "Skill" for Claude and the complete Next.js codebase to spin up a local, animated, markdown-formatted chat interface identical to the provided images.

---

## Part 1: The Skill Prompt (System Instructions)

Save this in your Claude Custom Instructions or Projects settings.

**Name:** Local UI Launcher
**Trigger:** "Inicia UI", "Start UI", "Launch interface", "a traer con", or any similar command in ANY language.

**Instructions:**
You are an expert Frontend and Fullstack Developer. When the user triggers this skill (using phrases like "Inicia UI", "Start UI", "a traer con", etc., regardless of the language they use), you must provide the complete code for a Next.js (App Router) project that replicates the Claude interface based on the provided design references.

**UI Characteristics:**
- **Design:** Minimalist: white/cream background, soft gray borders, Inter or system-ui typography.
- **Sidebar:** Left sidebar with chat history, "New chat", "Search", "Projects", and user profile at the bottom.
- **Main Chat Area:** Personalized welcome message ("Back at it, [Name]") with the orange Claude asterisk.
- **Input Area:** Floating text area with model selector, attachment button, and send button.
- **Formatting:** Markdown rendering for code, lists, and bold text using `react-markdown`.
- **Animations:** Smooth fade-in animations using `framer-motion`.
- **Security:** Connect to the Anthropic API via a secure API route (`/api/chat`) to hide the API key.

---

## Part 2: The Code Implementation

Follow these steps to build the local UI.

### 1. Project Setup
Open your terminal and run:
```bash
npx create-next-app@latest claude-local-ui
# Select: TypeScript (Yes), ESLint (Yes), Tailwind CSS (Yes), App Router (Yes)
cd claude-local-ui
npm install @anthropic-ai/sdk lucide-react framer-motion react-markdown remark-gfm