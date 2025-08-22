# Al-Koushar
A ChatGPT-powered site named Al-Koushar
# Create downloadable files for the user: the React component and a ZIP with instructions
import os, zipfile, textwrap, datetime, json, pathlib

base = "/mnt/data"
ts_name = "Al-Koushar-App.tsx"
readme_name = "MOBILE_STEPS.txt"
zip_name = "alkoushar-react-files.zip"

react_code = r'''import React, { useEffect, useMemo, useRef, useState } from "react";
import { motion } from "framer-motion";
import { Bot, Send, User, Sparkles, Settings, Shield, TerminalSquare, Rocket, Github, Info, Lock, Globe2 } from "lucide-react";
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle, CardDescription } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { Label } from "@/components/ui/label";

const SITE_TITLE = "Al‑Koushar AI";
const ASSISTANT_NAME = "Al‑Koushar Assistant";
const DEFAULT_SYSTEM_PROMPT = `You are ${ASSISTANT_NAME}, a friendly and helpful Hindi‑first assistant built for Md Imran Koushar.\nStyle: concise, clear Hinglish/Hindi, with practical steps.`;

function demoReply(userText) {
  const lower = userText.toLowerCase();
  if (!userText.trim()) return "Kuch likhiye, main madad karta hoon.";
  if (/(hello|hi|namaste|salaam)/.test(lower)) {
    return "Namaste! Main Al‑Koushar AI hoon. Aaj main aapke liye kya kar sakta hoon?";
  }
  if (/youtube|views|timing/.test(lower)) {
    return "YouTube ke liye best timing: Mon–Fri shaam 6–9 baje, Sat–Sun dopahar 12–3 aur shaam 6–9. Shorts roz, long video hafte me 2–3 baar post karein.";
  }
  if (/business|dukan|shop|profit/.test(lower)) {
    return "Business tip: Inventory ko A‑B‑C categories me baant kar reorder point set kijiye. Weekly sales report nikaal kar slow‑moving stock par discount run karein.";
  }
  if (/love|shayari|dosti|romance|mohabbat/.test(lower)) {
    return "Shayari: \nTere naam ki roshan raatein, meri saanson par likhi hain,\nJo tu saath ho to mushkilein bhi muskurati dikhi hain.";
  }
  if (/edius|video|editing/.test(lower)) {
    return "EDIUS me 'Offline' clips aayen to Bin (F11) me jaakar right‑click → Relink karein, sahi folder select karke 'Search subfolders' tick kijiye.";
  }
  return "Samjha. Is par ek chhota plan: 1) Goal clear karo 2) Steps list banao 3) Resources jodo 4) Test & improve. Agar aap detail den, main full plan likh doon.";
}

function useLocalStorage(key, initial) {
  const [state, setState] = React.useState(() => {
    try {
      const v = localStorage.getItem(key);
      return v !== null ? JSON.parse(v) : initial;
    } catch { return initial; }
  });
  React.useEffect(() => {
    try { localStorage.setItem(key, JSON.stringify(state)); } catch {}
  }, [key, state]);
  return [state, setState];
}

export default function App() {
  const [apiKey, setApiKey] = useLocalStorage("alkoushar_api_key", "");
  const [model, setModel] = useLocalStorage("alkoushar_model", "gpt-4o-mini");
  const [systemPrompt, setSystemPrompt] = useLocalStorage("alkoushar_system", DEFAULT_SYSTEM_PROMPT);
  const [messages, setMessages] = useState([
    { role: "assistant", content: `Salaam! Main ${ASSISTANT_NAME} hoon — aap Hindi/Hinglish me puchhiye, main turant madad karta hoon.` }
  ]);
  const [input, setInput] = useState("");
  const [loading, setLoading] = useState(false);
  const listRef = useRef(null);

  useEffect(() => {
    listRef.current?.scrollTo({ top: listRef.current.scrollHeight, behavior: "smooth" });
  }, [messages, loading]);

  const hasKey = useMemo(() => (apiKey || "").trim().length > 20, [apiKey]);

  async function sendMessage(e) {
    e?.preventDefault();
    const text = input.trim();
    if (!text || loading) return;
    setMessages(m => [...m, { role: "user", content: text }]);
    setInput("");
    setLoading(true);

    try {
      if (!hasKey) {
        await new Promise(r => setTimeout(r, 600));
        const reply = demoReply(text);
        setMessages(m => [...m, { role: "assistant", content: reply }]);
      } else {
        const body = {
          model,
          messages: [
            { role: "system", content: systemPrompt },
            ...messages,
            { role: "user", content: text }
          ],
          temperature: 0.6
        };
        const res = await fetch("https://api.openai.com/v1/chat/completions", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
            Authorization: `Bearer ${apiKey}`
          },
          body: JSON.stringify(body)
        });
        if (!res.ok) throw new Error("API error");
        const data = await res.json();
        const reply = data.choices?.[0]?.message?.content || "(No reply)";
        setMessages(m => [...m, { role: "assistant", content: reply }]);
      }
    } catch (err) {
      setMessages(m => [...m, { role: "assistant", content: "Error: reply generate nahi ho paya. Settings me API key check karein ya thodi der baad try karein." }]);
    } finally {
      setLoading(false);
    }
  }

  function clearChat() {
    setMessages([{ role: "assistant", content: `Namaste! Main ${ASSISTANT_NAME} hoon. Aap kya madad chahte ho?` }]);
  }

  return (
    <div className="min-h-screen bg-gradient-to-b from-slate-50 to-slate-100 text-slate-900">
      <header className="sticky top-0 z-10 backdrop-blur bg-white/70 border-b">
        <div className="max-w-6xl mx-auto px-4 py-3 flex items-center justify-between">
          <div className="flex items-center gap-3">
            <motion.div initial={{ scale: 0.9 }} animate={{ scale: 1 }} className="p-2 rounded-2xl bg-slate-900 text-white">
              <Sparkles size={18} />
            </motion.div>
            <div>
              <h1 className="text-xl font-bold">{SITE_TITLE}</h1>
              <p className="text-xs text-slate-500">by Md Imran <span className="font-semibold">Al‑Koushar</span></p>
            </div>
          </div>
          <div className="hidden md:flex items-center gap-2">
            <a href="#chat"><button className="rounded-2xl px-3 py-2 bg-slate-900 text-white">Try Chat</button></a>
            <a href="#about"><button className="rounded-2xl px-3 py-2 border">About</button></a>
          </div>
        </div>
      </header>

      <section className="max-w-6xl mx-auto px-4 pt-10 pb-6">
        <div className="grid md:grid-cols-2 gap-6 items-center">
          <div>
            <motion.h2 initial={{ y: 10, opacity: 0 }} animate={{ y:0, opacity: 1 }} className="text-3xl md:text-4xl font-extrabold leading-tight">
              Build your own AI — <span className="text-slate-600">Al‑Koushar style</span>
            </motion.h2>
            <p className="mt-3 text-slate-600">Hindi‑first AI chatbot jo aapke business, video aur daily kaam me turant madad kare. Neeche demo try kijiye ya apni API key daal kar full power unlock karein.</p>
            <div className="mt-4 flex gap-3">
              <a href="#chat"><button className="rounded-2xl px-3 py-2 bg-slate-900 text-white">Start Chat</button></a>
              <a href="#settings"><button className="rounded-2xl px-3 py-2 border">Settings</button></a>
            </div>
          </div>
          <motion.div initial={{ opacity: 0 }} animate={{ opacity: 1 }} className="">
            <div className="rounded-2xl shadow-lg border bg-white">
              <div className="p-4 border-b">
                <div className="text-lg font-semibold">Fast, Simple, Yours</div>
                <div className="text-sm text-slate-500">Lightweight UI • Multi‑tab layout • BYO API key</div>
              </div>
              <div className="grid grid-cols-3 gap-3 p-4">
                <div className="p-3 border rounded-2xl bg-white">
                  <div className="flex items-center gap-2 text-slate-700"><span className="opacity-80"><Bot/></span> <span className="font-semibold">ChatGPT‑style</span></div>
                  <div className="text-xs text-slate-500 mt-1">Clean chat UI with memory</div>
                </div>
                <div className="p-3 border rounded-2xl bg-white">
                  <div className="flex items-center gap-2 text-slate-700"><span className="opacity-80"><Shield/></span> <span className="font-semibold">Local control</span></div>
                  <div className="text-xs text-slate-500 mt-1">Key stored only in your browser</div>
                </div>
                <div className="p-3 border rounded-2xl bg-white">
                  <div className="flex items-center gap-2 text-slate-700"><span className="opacity-80"><TerminalSquare/></span> <span className="font-semibold">Dev‑friendly</span></div>
                  <div className="text-xs text-slate-500 mt-1">Single‑file React app</div>
                </div>
              </div>
            </div>
          </motion.div>
        </div>
      </section>

      <section className="max-w-6xl mx-auto px-4 pb-16" id="chat">
        <div className="rounded-2xl border bg-white">
          <div className="p-4 border-b">
            <div className="text-lg font-semibold">Chat with {ASSISTANT_NAME}</div>
            <div className="text-sm text-slate-500">{(apiKey || '').trim().length > 20 ? `Model: ${model}` : "Demo Mode active (no API key)"}</div>
          </div>
          <div className="p-4">
            <div ref={listRef} className="h-[460px] overflow-y-auto pr-2">
              <ul className="space-y-4">
                {messages.map((m, idx) => (
                  <li key={idx} className={`flex ${m.role === 'user' ? 'justify-end' : 'justify-start'}`}>
                    <div className={`max-w-[85%] rounded-2xl p-3 text-sm shadow-sm ${m.role === 'user' ? 'bg-slate-900 text-white' : 'bg-white border'}`}>
                      <div className="flex items-center gap-2 mb-1 text-xs opacity-70">
                        {m.role === 'user' ? <User size={14}/> : <Bot size={14}/>} {m.role === 'user' ? 'You' : ASSISTANT_NAME}
                      </div>
                      <div className="whitespace-pre-wrap leading-relaxed">{m.content}</div>
                    </div>
                  </li>
                ))}
                {loading && (
                  <li className="flex justify-start"><div className="rounded-2xl p-3 bg-white border text-sm">Typing…</div></li>
                )}
              </ul>
            </div>
            <form onSubmit={sendMessage} className="mt-3 flex gap-2">
              <input value={input} onChange={e=>setInput(e.target.value)} placeholder="Type your message… (Hindi/Hinglish)" className="rounded-2xl px-3 py-2 border flex-1" />
              <button type="submit" className="rounded-2xl px-3 py-2 bg-slate-900 text-white" disabled={loading}>Send</button>
              <button type="button" className="rounded-2xl px-3 py-2 border" onClick={clearChat}>Clear</button>
            </form>
            <div className="text-xs text-slate-500 mt-2">Settings me API key daal kar real AI replies enable karein.</div>
          </div>
        </div>

        <div className="mt-6 rounded-2xl border bg-white p-4" id="settings">
          <div className="text-lg font-semibold mb-2">Settings</div>
          <div className="grid md:grid-cols-2 gap-4">
            <div>
              <div className="text-sm font-medium mb-1">OpenAI API Key</div>
              <input type="password" placeholder="sk‑..." value={apiKey} onChange={e=>setApiKey(e.target.value)} className="rounded-2xl px-3 py-2 border w-full"/>
              <div className="text-xs text-slate-500 mt-1">Note: Browser me key rakhna sirf personal use ke liye theek hai.</div>
            </div>
            <div>
              <div className="text-sm font-medium mb-1">Model</div>
              <select className="w-full border rounded-2xl p-2" value={model} onChange={e=>setModel(e.target.value)}>
                <option value="gpt-4o-mini">gpt‑4o‑mini</option>
                <option value="gpt-4o">gpt‑4o</option>
                <option value="gpt-3.5-turbo">gpt‑3.5‑turbo</option>
              </select>
              <div className="text-sm font-medium mt-3 mb-1">System Prompt</div>
              <textarea rows={6} value={systemPrompt} onChange={e=>setSystemPrompt(e.target.value)} className="rounded-2xl px-3 py-2 border w-full"/>
            </div>
          </div>
        </div>
      </section>

      <footer className="border-t py-6 text-center text-sm text-slate-500">
        <div className="max-w-6xl mx-auto px-4 flex flex-col md:flex-row items-center justify-between gap-2">
          <div className="flex items-center gap-2">© {new Date().getFullYear()} Al‑Koushar. All rights reserved.</div>
          <div className="flex items-center gap-3">
            <a className="hover:text-slate-700" href="#about">About</a>
            <a className="hover:text-slate-700" href="#settings">Settings</a>
          </div>
        </div>
      </footer>
    </div>
  );
}'''

mobile_steps = """MOBILE STEP-BY-STEP (1 number: code copy/download on phone)

Goal: Is single React component ko mobile se le kar online editor me paste karna, ya future use ke liye safe rakhna.

A) DIRECT DOWNLOAD (sabse aasan)
1) Niche diye link "Al-Koushar-App.tsx" par tap karein — file download ho jayegi.
2) File Manager me 'Downloads' folder me dikhegi. Ye React component hai (TypeScript).

B) ONLINE EDITOR (phone se run karna)
Option 1 — CodeSandbox (mobile par chal jata hai):
1) codesandbox kholiye → "Create Sandbox" → template: "Next.js + Tailwind".
2) Left side me "Terminal" open kijiye (Menu → Terminal). Run:
   npx shadcn@latest init -y
   npx shadcn@latest add button card input textarea tabs label
3) 'app/page.tsx' file open kijiye → pura content delete karke, is downloaded file 'Al-Koushar-App.tsx' ka content paste kijiye.
4) Save karein. Preview pane me aapki site open ho jayegi.
5) Settings tab me apni OpenAI API Key paste karke real AI replies enable karein (optional).

Option 2 — StackBlitz:
1) stackblitz kholiye → "Start a new project" → "Next.js (TypeScript)".
2) Terminal me upar jaisi shadcn commands chalaein (step 2).
3) 'app/page.tsx' me content paste kar dein. Save karein → Preview ho jayega.

C) GITHUB + VERCEL (phone se publish)
1) github.com par naya repo banaye (e.g., alkoushar-ai).
2) CodeSandbox/StackBlitz me "Export to GitHub" kar dein (menu me hota hai), ya manually new file 'app/page.tsx' create karke paste karein.
3) vercel.com par login → "Add New Project" → apne repo ko import karein → deploy. Aapko live URL mil jayega.

Tips:
- Agar shadcn components add nahi kiye, to imports (button, card, input, textarea, tabs, label) wali lines error dengi.
- Phone par copy/paste easy banane ke liye: pehle Notepad app me paste karke phir editor me paste karein.
"""

# Write files
with open(os.path.join(base, ts_name), "w", encoding="utf-8") as f:
    f.write(react_code)

with open(os.path.join(base, readme_name), "w", encoding="utf-8") as f:
    f.write(mobile_steps)

# Zip them
zip_path = os.path.join(base, zip_name)
with zipfile.ZipFile(zip_path, "w", zipfile.ZIP_DEFLATED) as z:
    z.write(os.path.join(base, ts_name), arcname=ts_name)
    z.write(os.path.join(base, readme_name), arcname=readme_name)

zip_path, os.path.join(base, ts_name), os.path.join(base, readme_name)
