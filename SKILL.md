
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
```

### 2. Environment Variables
Create a `.env.local` file in the root directory and add your Anthropic API key:
```env
ANTHROPIC_API_KEY=your_api_key_here
```

### 3. Backend API Route
Create the file `app/api/chat/route.ts`. This keeps your API key secure.

```typescript
import { NextResponse } from 'next/server';
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

export async function POST(req: Request) {
  try {
    const { messages } = await req.json();

    const response = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20240620', // Or the latest model
      max_tokens: 4096,
      messages: messages,
      system: "You are a helpful assistant.",
    });

    return NextResponse.json({ message: response.content[0].text });
  } catch (error) {
    console.error(error);
    return NextResponse.json({ error: 'Error connecting to Claude' }, { status: 500 });
  }
}
```

### 4. Main Chat Interface
Replace the contents of `app/page.tsx` with this code:

```tsx
'use client';
import { useState } from 'react';
import { motion } from 'framer-motion';
import ReactMarkdown from 'react-markdown';
import { Paperclip, Send, ChevronDown } from 'lucide-react';
import { Sidebar } from '@/components/Sidebar';

export default function ChatUI() {
  const [messages, setMessages] = useState<{role: string, content: string}[]>([]);
  const [input, setInput] = useState('');
  const [isLoading, setIsLoading] = useState(false);

  const sendMessage = async () => {
    if (!input.trim()) return;

    const newMessages = [...messages, { role: 'user', content: input }];
    setMessages(newMessages);
    setInput('');
    setIsLoading(true);

    try {
      const res = await fetch('/api/chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ messages: newMessages }),
      });
      const data = await res.json();
      setMessages([...newMessages, { role: 'assistant', content: data.message }]);
    } catch (error) {
      console.error(error);
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div className="flex h-screen bg-[#FAFAF9] text-[#1A1A1A] font-sans">
      <Sidebar />

      {/* Main Chat Area */}
      <div className="flex-1 flex flex-col items-center justify-center relative">
        {messages.length === 0 ? (
          <motion.div 
            initial={{ opacity: 0, y: 20 }} 
            animate={{ opacity: 1, y: 0 }} 
            className="text-center"
          >
            {/* Claude Logo (Asterisk) */}
            <div className="text-[#D97757] text-5xl mb-4">✺</div>
            <h1 className="text-3xl font-serif text-gray-800">Back at it, Germán</h1>
          </motion.div>
        ) : (
          <div className="w-full max-w-3xl h-full overflow-y-auto pt-20 pb-32 px-4 flex flex-col gap-6">
            {messages.map((msg, i) => (
              <motion.div 
                key={i} 
                initial={{ opacity: 0 }} 
                animate={{ opacity: 1 }}
                className={`flex gap-4 ${msg.role === 'user' ? 'justify-end' : 'justify-start'}`}
              >
                {msg.role === 'assistant' && <div className="w-8 h-8 rounded-full bg-[#D97757] flex items-center justify-center text-white text-xs">✺</div>}
                <div className={`max-w-[80%] rounded-2xl p-4 ${msg.role === 'user' ? 'bg-gray-200' : 'bg-transparent'}`}>
                  <ReactMarkdown className="prose prose-sm md:prose-base prose-pre:bg-gray-800 prose-pre:text-white">
                    {msg.content}
                  </ReactMarkdown>
                </div>
              </motion.div>
            ))}
            {isLoading && <div className="text-gray-500 animate-pulse">Claude is thinking...</div>}
          </div>
        )}

        {/* Input Area (Floating) */}
        <div className="absolute bottom-8 w-full max-w-3xl px-4">
          <div className="bg-white border border-gray-200 rounded-2xl shadow-sm flex flex-col p-2 relative">
            <textarea
              value={input}
              onChange={(e) => setInput(e.target.value)}
              onKeyDown={(e) => e.key === 'Enter' && !e.shiftKey && (e.preventDefault(), sendMessage())}
              placeholder="How can I help you today?"
              className="w-full resize-none outline-none p-3 text-gray-700 bg-transparent max-h-40"
              rows={1}
            />
            <div className="flex justify-between items-center px-2 pb-1">
              <div className="flex items-center gap-2">
                 <button className="p-2 hover:bg-gray-100 rounded-lg text-gray-500"><Paperclip size={18}/></button>
              </div>
              <div className="flex items-center gap-3">
                <button className="flex items-center gap-1 text-sm text-gray-500 hover:text-gray-800">
                  Opus 4.6 Extended <ChevronDown size={14}/>
                </button>
                <button 
                  onClick={sendMessage}
                  disabled={isLoading || !input.trim()}
                  className="p-2 bg-[#D97757] text-white rounded-lg hover:bg-[#c46545] disabled:opacity-50"
                >
                  <Send size={16} />
                </button>
              </div>
            </div>
          </div>
          
          {/* Suggestion Buttons */}
          <div className="flex justify-center gap-2 mt-3">
            {['Write', 'Learn', 'Code', 'Life stuff', "Claudia's choice"].map((item) => (
              <button key={item} className="px-3 py-1.5 text-xs font-medium text-gray-600 bg-white border border-gray-200 rounded-full hover:bg-gray-50">
                {item}
              </button>
            ))}
          </div>
        </div>
      </div>
    </div>
  );
}
```

### 5. Sidebar Component
Create a `components` folder, and inside it, create `Sidebar.tsx`:

```tsx
import { Plus, Search, MessageSquare, Folder, LayoutTemplate, Code, Settings } from 'lucide-react';

export function Sidebar() {
  return (
    <div className="w-[260px] bg-[#F5F5F4] border-r border-gray-200 flex flex-col h-full p-4">
      <div className="flex items-center justify-between mb-6">
        <h2 className="font-semibold text-gray-800">Claude</h2>
        <button className="p-1.5 hover:bg-gray-200 rounded-md"><LayoutTemplate size={16}/></button>
      </div>

      <button className="flex items-center gap-2 w-full bg-white border border-gray-200 rounded-lg p-2 text-sm text-gray-700 hover:bg-gray-50 shadow-sm mb-4">
        <Plus size={16} /> New chat
      </button>

      <div className="space-y-1 mb-6 text-sm text-gray-600">
        <button className="flex items-center gap-3 w-full p-2 hover:bg-gray-200 rounded-lg"><Search size={16}/> Search</button>
        <button className="flex items-center gap-3 w-full p-2 hover:bg-gray-200 rounded-lg"><Settings size={16}/> Customize</button>
      </div>

      <div className="space-y-1 text-sm text-gray-600 flex-1">
        <p className="text-xs font-semibold text-gray-400 mb-2 px-2">Recent</p>
        <button className="flex items-center gap-3 w-full p-2 hover:bg-gray-200 rounded-lg"><MessageSquare size={16}/> Chats</button>
        <button className="flex items-center gap-3 w-full p-2 hover:bg-gray-200 rounded-lg"><Folder size={16}/> Projects</button>
        <button className="flex items-center gap-3 w-full p-2 hover:bg-gray-200 rounded-lg"><LayoutTemplate size={16}/> Artifacts</button>
        <button className="flex items-center gap-3 w-full p-2 hover:bg-gray-200 rounded-lg"><Code size={16}/> Code</button>
      </div>

      <div className="border-t border-gray-200 pt-4 mt-auto">
        <button className="flex items-center gap-3 w-full p-2 hover:bg-gray-200 rounded-lg text-left">
          <div className="w-8 h-8 rounded-full bg-black text-white flex items-center justify-center text-xs">GM</div>
          <div>
            <p className="text-sm font-medium text-gray-800">Germán Martínez</p>
            <p className="text-xs text-gray-500">Max plan</p>
          </div>
        </button>
      </div>
    </div>
  );
}
```

### 6. Run the App
Start your local server:
```bash
npm run dev
```
Open `http://localhost:3000` in your browser to interact with your local Claude UI.

---

## Note on "Cowork" (Agent Mode)
The first image shows an agentic mode ("Cowork") handling background tasks. The code above builds the standard chat UI (second image). To achieve the "Cowork" agentic behavior, you would need to implement **Anthropic Tool Calling** in the API route to allow Claude to execute functions locally, which requires a more advanced backend setup.
