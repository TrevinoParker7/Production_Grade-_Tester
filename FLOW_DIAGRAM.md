# 🔄 Complete Project Flow Diagram
## AI Reliability Scanner - Detailed ASCII Diagrams

---

## 1️⃣ PROJECT DIRECTORY STRUCTURE

```
ai-reliability-scanner/
│
├── 📄 package.json                           ← Project configuration & scripts
│   ├── "dev": "tsx server.ts"               ← Dev entry point
│   ├── "build": "vite build && esbuild"     ← Build command
│   └── "start": "node dist/server.cjs"      ← Production entry
│
├── 📄 tsconfig.json                          ← TypeScript configuration
├── 📄 vite.config.ts                         ← Frontend build config
├── 📄 .env.example                           ← Secrets template
├── 📄 .env                                   ← Actual secrets (not in git)
├── 📄 .gitignore                             ← Git exclusions
├── 📄 README.md                              ← Documentation
├── 📄 CLAUDE.md                              ← Developer guide
├── 📄 TUTORIAL.md                            ← Complete learning guide
├── 📄 FLOW_DIAGRAM.md                        ← THIS FILE
│
├── 📁 src/                                    ← Frontend + Engine code
│   │
│   ├── 📄 main.tsx                           ← React entry point
│   │   └── Calls: ReactDOM.createRoot()
│   │       └── Renders: <App />
│   │
│   ├── 📄 App.tsx                            ← Main React component
│   │   ├── State: targetUrl, scanning, results, logs
│   │   ├── Function: startScan()
│   │   │   ├── Creates EventSource
│   │   │   ├── Listens for SSE messages
│   │   │   └── Updates UI in real-time
│   │   └── Renders: Form + Logs + Results display
│   │
│   └── 📁 engine/                            ← Test logic
│       └── 📄 runner.ts                      ← Test functions
│           ├── runSecurityTest()
│           ├── runLoadTest()
│           ├── runChaosTest()
│           ├── runToolValidationTest()
│           ├── runMemoryTest()
│           └── runHallucinationTest()
│
├── 📁 public/                                 ← Static assets
│   └── 📄 index.html                         ← HTML skeleton
│       └── <div id="root"></div>
│
├── 📄 server.ts                              ← Express backend
│   ├── app = express()
│   ├── app.get("/api/health")                ← Health check endpoint
│   └── app.get("/api/scan")                  ← Main scan endpoint
│       ├── Extract targetUrl from query
│       ├── Set up SSE headers
│       ├── Create emit() function
│       └── Call runAllTests()
│
├── 📁 dist/                                  ← Build output (created after npm run build)
│   ├── 📄 index.html                         ← Built HTML
│   ├── 📁 assets/
│   │   ├── app.js (minified)
│   │   └── app.css (minified)
│   └── 📄 server.cjs                         ← Bundled server
│
└── 📁 node_modules/                          ← Installed packages (not in git)
    ├── react/
    ├── express/
    ├── typescript/
    └── ... (100+ more)
```

---

## 2️⃣ COMPLETE USER ACTION FLOW

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         STEP 1: USER OPENS APP                              │
└─────────────────────────────────────────────────────────────────────────────┘

    Browser                          Server                       Disk
       │                               │                            │
       ├─ npm run dev ─────────────────┤                            │
       │                               │                            │
       │◄──── Vite hot reload ─────────┤                            │
       │                               │                            │
       │  GET http://localhost:3000    │                            │
       ├────────────────────────────→  │                            │
       │                               │                            │
       │                               ├─ Load vite.config.ts ──────┤
       │                               ├─ Load App.tsx ────────────┤
       │                               ├─ Load main.tsx ──────────┤
       │                               │                            │
       │◄──────── Serve index.html ────┤                            │
       │   (with webpack bundle)       │                            │
       │                               │                            │
       ├─ Execute main.tsx             │                            │
       │   └─ ReactDOM.createRoot()    │                            │
       │      └─ Render <App />        │                            │
       │                               │                            │
       ✓ User sees:                     │                            │
       │  ┌──────────────────────────┐  │                            │
       │  │ AI Reliability Scanner   │  │                            │
       │  │ [Input: https://...]     │  │                            │
       │  │ [Button: Run Scan]       │  │                            │
       │  └──────────────────────────┘  │                            │
       │                                 │                            │


┌─────────────────────────────────────────────────────────────────────────────┐
│                      STEP 2: USER ENTERS URL AND CLICKS                      │
└─────────────────────────────────────────────────────────────────────────────┘

    React Component                    User Action
       │
       ├─ Input: targetUrl = "https://api.example.com"
       │
       ├─ Click: "Run Scan" button
       │   │
       │   └─ Trigger: onClick handler
       │      │
       │      └─ Call: startScan()
       │         │
       │         └─ setState(scanning = true)
       │            setState(logs = [])
       │            setState(results = null)
       │


┌─────────────────────────────────────────────────────────────────────────────┐
│                  STEP 3: OPEN SSE CONNECTION TO BACKEND                      │
└─────────────────────────────────────────────────────────────────────────────┘

    Browser (App.tsx)                           Server (server.ts)
       │                                            │
       │  const eventSource = new EventSource(      │
       │    `/api/scan?url=https://api.example.com` │
       ├─ GET /api/scan?url=... ──────────────────→ │
       │                                            │
       │                                            ├─ Validate URL
       │                                            │
       │                                            ├─ Set SSE Headers:
       │                                            │  Content-Type: text/event-stream
       │                                            │  Cache-Control: no-cache
       │                                            │  Connection: keep-alive
       │                                            │
       │                                            ├─ Create emit() function
       │                                            │
       │                                            ├─ Start: runAllTests()
       │                                            │
       │◄─── Connection stays open ────────────────┤ (don't send res.end() yet)
       │                                            │
       ✓ eventSource.readyState = OPEN             │
       │                                            │


┌─────────────────────────────────────────────────────────────────────────────┐
│              STEP 4: BACKEND ORCHESTRATES TEST PHASES (CORE LOGIC)           │
└─────────────────────────────────────────────────────────────────────────────┘

    Server.ts: runAllTests() function
    ════════════════════════════════════════════════════════════════════════
    
    async function runAllTests() {
      
      const finalScores = {
        security: 0,
        reliability: 0,
        chaos_resilience: 0,
        tool_integrity: 100,
        hallucination_risk: "high",
        production_readiness: 0
      };
      
      const findings = [];


      ┌──────────────────────────────────────────────────────────────────────┐
      │ PHASE 1: INGESTION (Check if target is reachable)                   │
      └──────────────────────────────────────────────────────────────────────┘
      
        emit("status", { phase: "Ingestion", active: true })
            │
            └─→ Browser receives SSE message
                └─→ UI updates: "Ingestion (running...)"
        
        emit("log", { message: "[Ingestion] Checking if target is reachable..." })
            │
            └─→ Browser appends to logs array
                └─→ UI updates: Shows log in console
        
        try {
          const controller = new AbortController();
          const id = setTimeout(() => controller.abort(), 8000);
          const resp = await fetch(targetUrl, { 
            method: "HEAD",  // Don't download body, just check headers
            signal: controller.signal 
          });
          clearTimeout(id);
          isReachable = resp.ok || resp.status < 500;
        } catch (e) {
          emit("log", { message: "Target unreachable or refused HEAD request" })
        }
        
        emit("status", { phase: "Ingestion", active: false })
            │
            └─→ UI updates: "Ingestion (complete)"


      ┌──────────────────────────────────────────────────────────────────────┐
      │ PHASE 2: SECURITY (Check for vulnerabilities)                        │
      └──────────────────────────────────────────────────────────────────────┘
      
        emit("status", { phase: "Security", active: true })
        
        const secResult = await runSecurityTest(targetUrl, headers, emit)
            │
            └─→ Call: src/engine/runner.ts
                │
                ├─ emit("log", "[Security] Checking for common vulnerabilities...")
                │
                ├─ Simulate test (real would check security headers)
                │
                └─ return { score: 82, findings: [...] }
        
        finalScores.security = 82
        findings.push(...secResult.findings)
        
        emit("status", { phase: "Security", active: false })


      ┌──────────────────────────────────────────────────────────────────────┐
      │ PHASE 3: CHAOS (Simulate failures under stress)                      │
      └──────────────────────────────────────────────────────────────────────┘
      
        emit("status", { phase: "Chaos", active: true })
        
        const chaosResult = await runChaosTest(targetUrl, headers, emit)
            └─→ Simulates: Multiple concurrent requests, timeouts, etc.
                └─ return { score: 88, findings: [...] }
        
        finalScores.chaos_resilience = 88
        findings.push(...chaosResult.findings)


      ┌──────────────────────────────────────────────────────────────────────┐
      │ PHASE 4: TOOL VALIDATION (Check function/tool integrity)             │
      └──────────────────────────────────────────────────────────────────────┘
      
        emit("status", { phase: "ToolValidation", active: true })
        
        const toolResult = await runToolValidationTest(targetUrl, headers, emit)
            └─ return { score: 90, findings: [...] }
        
        finalScores.tool_integrity = 90


      ┌──────────────────────────────────────────────────────────────────────┐
      │ PHASE 5: MEMORY (Check for memory leaks/state issues)                │
      └──────────────────────────────────────────────────────────────────────┘
      
        emit("status", { phase: "Memory", active: true })
        
        const memResult = await runMemoryTest(targetUrl, headers, emit)
            └─ return { score: 85, findings: [...] }
        
        finalScores.security = Math.max(0, finalScores.security - (100 - 85))
            └─ Penalize security if memory issues found


      ┌──────────────────────────────────────────────────────────────────────┐
      │ PHASE 6: BROWSER (Simulate user interactions)                        │
      └──────────────────────────────────────────────────────────────────────┘
      
        emit("status", { phase: "Browser", active: true })
        
        emit("log", { message: "[Browser] Mapping frontend routes..." })
        
        emit("log", { message: "[Browser] Executing session persistence..." })
        
        await new Promise(r => setTimeout(r, 1200))  // Simulate work
        
        emit("status", { phase: "Browser", active: false })


      ┌──────────────────────────────────────────────────────────────────────┐
      │ PHASE 7: LOAD (Stress test reliability)                              │
      └──────────────────────────────────────────────────────────────────────┘
      
        emit("status", { phase: "Load", active: true })
        
        const loadResult = await runLoadTest(targetUrl, headers, emit)
            └─ return { score: 75, findings: [...] }
        
        finalScores.reliability = 75


      ┌──────────────────────────────────────────────────────────────────────┐
      │ PHASE 8: HALLUCINATION (AI semantic analysis via Gemini)             │
      └──────────────────────────────────────────────────────────────────────┘
      
        emit("status", { phase: "Hallucination", active: true })
        
        const hallResult = await runHallucinationTest(targetUrl, headers, emit)
            │
            ├─ 1. Get responses from target AI
            │     const responses = await fetch(targetUrl/api/query, ...)
            │
            ├─ 2. Send to Gemini API
            │     const client = new GoogleGenAI(process.env.GEMINI_API_KEY)
            │     const analysis = await client.models.generateContent({
            │       contents: [{
            │         text: `Analyze if these responses hallucinate: ${responses}`
            │       }]
            │     })
            │
            └─ return { 
                 risk: "medium",
                 tool_integrity: 85,
                 findings: ["2 hallucinations detected"]
               }
        
        finalScores.hallucination_risk = "medium"
        finalScores.tool_integrity = Math.round((90 + 85) / 2)  // Average


      ┌──────────────────────────────────────────────────────────────────────┐
      │ CALCULATE PRODUCTION READINESS SCORE                                  │
      └──────────────────────────────────────────────────────────────────────┘
      
        let baseScore = Math.round(
          (82 + 75 + 88 + 87.5) / 4
        );
        // baseScore = 83
        
        if (finalScores.hallucination_risk === "high") {
          baseScore -= 20;  // High risk = -20 points
        } else if (finalScores.hallucination_risk === "medium") {
          baseScore -= 10;  // Medium risk = -10 points
        }
        // baseScore = 83 - 10 = 73
        
        finalScores.production_readiness = Math.max(0, baseScore);
        // Result: 73


      ┌──────────────────────────────────────────────────────────────────────┐
      │ SEND FINAL RESULTS TO FRONTEND                                       │
      └──────────────────────────────────────────────────────────────────────┘
      
        emit("complete", {
          target: "https://api.example.com",
          scores: {
            security: 82,
            reliability: 75,
            chaos_resilience: 88,
            tool_integrity: 87.5,
            hallucination_risk: "medium",
            production_readiness: 73
          },
          findings: [
            "Missing rate limiting",
            "Missing HTTPS enforcement",
            "2 hallucinations detected"
          ]
        })
        
        res.end()  // Close the SSE connection
    }
```

---

## 3️⃣ REAL-TIME SSE MESSAGE FLOW (Timeline)

```
TIME    FRONTEND (Browser)              SERVER (Node.js)              MESSAGE
═════════════════════════════════════════════════════════════════════════════════

0:00    User clicks "Run Scan"          
        ├─ startScan() called
        └─ EventSource opened ───────→  app.get("/api/scan") triggered
                                        ├─ Validate input
                                        └─ Start runAllTests()

0:05                                    Phase 1: Ingestion starts
                                        └─ emit("status", {...})
        ◄──── SSE Message ──────────────
        eventSource.onmessage fires
        setLogs(prev => [...prev, "Ingestion starting"])
        UI Updates: Shows "Ingestion (running...)"

0:08                                    emit("log", {message: "...reachable"})
        ◄──── SSE Message ──────────────
        setLogs(prev => [...prev, "Target is reachable"])
        UI Updates: Adds new log line

0:10                                    Phase 1 complete
                                        Phase 2: Security starts
                                        └─ emit("status", {...})
        ◄──── SSE Message ──────────────
        UI Updates: Now shows "Security (running...)"

0:12                                    emit("log", {message: "...Checked headers"})
        ◄──── SSE Message ──────────────
        setLogs(prev => [...prev, "[Security] Checked headers..."])
        UI Updates: New log appears

0:14                                    emit("log", {message: "Score: 82/100"})
        ◄──── SSE Message ──────────────
        UI Updates: New log appears

0:16                                    Phase 2 complete
                                        Phase 3-6: Similar pattern
                                        (logs and status updates streaming)

0:20                                    Phase 7: Hallucination
                                        ├─ Call Gemini API
                                        │  └─ POST to Google API
                                        │     └─ Wait for response
                                        └─ emit("log", {...})
        ◄──── SSE Message ──────────────
        UI Updates: "Analyzing with Gemini..."

0:24                                    emit("log", {message: "Risk: medium"})
        ◄──── SSE Message ──────────────
        setLogs(prev => [...prev, "Risk: medium"])

0:25                                    All phases complete
                                        Calculate finalScores
                                        └─ emit("complete", {...})
        ◄──── SSE Message (FINAL) ─────
        eventSource.onmessage fires
        setResults(data)
        setScanning(false)
        eventSource.close()
        
        UI Updates: Shows all score cards + findings

═════════════════════════════════════════════════════════════════════════════════
```

---

## 4️⃣ DETAILED CODE EXECUTION FLOW (With Line Numbers)

```
FILE: src/main.tsx
═══════════════════════════════════════════════════════════════════════════════

1  import React from 'react';
2  import ReactDOM from 'react-dom/client';
3  import App from '@/src/App';
4
5  ReactDOM.createRoot(document.getElementById('root')!)
6                     ↓ Find <div id="root"> in index.html
7                     ├─ Create a React root
8                     └─ Render <App /> component
9      .render(
10       <React.StrictMode>
11         <App />
12              ↓ Jump to: src/App.tsx
13       </React.StrictMode>,
14     );


FILE: src/App.tsx
═══════════════════════════════════════════════════════════════════════════════

1  import { useState } from 'react';
2
3  export default function App() {
4    const [targetUrl, setTargetUrl] = useState('');      // State: URL
5    const [scanning, setScanning] = useState(false);      // State: Scanning?
6    const [results, setResults] = useState<any>(null);    // State: Results
7    const [logs, setLogs] = useState<string[]>([]);       // State: Logs
8
9    const startScan = async () => {                       // ← Main function
10     setScanning(true);                                  // Update state
11     setLogs([]);                                        // Clear logs
12     setResults(null);                                   // Clear results
13
14     try {
15       const eventSource = new EventSource(              // ← OPEN SSE
16         `/api/scan?url=${encodeURIComponent(targetUrl)}`
17       );                                                ↓ Send GET to /api/scan
18                                                         ↓ Server receives
19       eventSource.onmessage = (event) => {              // ← LISTEN for messages
20         const { type, data } = JSON.parse(event.data);
21
22         if (type === 'log') {
23           setLogs(prev => [...prev, data.message]);     // Add log to UI
24         }
25         else if (type === 'status') {
26           console.log('Phase:', data.phase);
27         }
28         else if (type === 'complete') {                 // ← FINAL MESSAGE
29           setResults(data);                             // Show results
30           setScanning(false);
31           eventSource.close();                          // Close connection
32         }
33       };
34
35       eventSource.onerror = () => {
36         setScanning(false);
37         eventSource.close();
38       };
39     } catch (error) {
40       setLogs(prev => [...prev, `❌ ${error.message}`]);
41       setScanning(false);
42     }
43   };
44
45   return (
46     <div className="min-h-screen ...">                  {/* Tailwind styling */}
47       {/* Header */}
48       <h1>AI Reliability Scanner</h1>
49
50       {/* Input Section */}
51       <input
52         value={targetUrl}
53         onChange={(e) => setTargetUrl(e.target.value)}   // Update state as user types
54       />
55       <button
56         onClick={startScan}                              // ← TRIGGER startScan()
57       >
58         Run Scan
59       </button>
60
61       {/* Logs Section */}
62       {logs.length > 0 && (
63         <div className="...">                            {/* Show only if logs exist */}
63           {logs.map((log, i) => (
64             <div key={i}>{log}</div>                     {/* Each log on new line */}
65           ))}
66         </div>
67       )}
68
69       {/* Results Section */}
70       {results && (
71         <div className="...">
72           <h2>Scan Results for {results.target}</h2>
73
74           {/* Score Cards */}
75           <div className="grid ...">
76             {Object.entries(results.scores).map(([key, value]: any) => (
77               <div key={key}>
78                 <p>{key.replace(/_/g, ' ')}</p>          {/* Format: "security" → "Security" */}
79                 <p className={value >= 70 ? 'green' : 'red'}>
80                   {typeof value === 'number' ? `${value}%` : value}
81                 </p>
82               </div>
83             ))}
84           </div>
85
86           {/* Findings List */}
87           {results.findings.length > 0 && (
88             <ul>
89               {results.findings.map((finding: any, i: number) => (
90                 <li key={i}>{finding}</li>
91               ))}
92             </ul>
93           )}
94         </div>
95       )}
96     </div>
97   );
98 }


FILE: server.ts
═══════════════════════════════════════════════════════════════════════════════

1  import express from "express";
2  import path from "path";
3  import { createServer as createViteServer } from "vite";
4  import {
5    runSecurityTest,
6    runLoadTest,
7    ... other test imports
8  } from "./src/engine/runner";
9
10 async function startServer() {
11   const app = express();                                 // Create Express app
12   const PORT = 3000;
13
14   app.use(express.json());                               // Parse JSON bodies
15
16   // Health check
17   app.get("/api/health", (req, res) => {
18     res.json({ status: "ok" });
19   });
20
21   // Main scan endpoint
22   app.get("/api/scan", (req, res) => {                  // ← ROUTE: /api/scan
23     const targetUrl = req.query.url as string;          // Extract URL from query
24
25     if (!targetUrl) {
26       return res.status(400).json({ error: "Missing url parameter" });
27     }
28
29     // Set up SSE headers
30     res.setHeader("Content-Type", "text/event-stream"); // Tell browser: streaming
31     res.setHeader("Cache-Control", "no-cache");         // Don't cache
32     res.setHeader("Connection", "keep-alive");          // Keep open
33     res.flushHeaders();                                 // Send headers immediately
34
35     // Create emit helper
36     const emit = (type: string, data: any) => {
37       res.write(`data: ${JSON.stringify({ type, data })}\n\n`);
38                 ↓ Send message to client
39                 ↓ Browser receives → eventSource.onmessage fires
40     };
41
42     // Run tests asynchronously
43     async function runAllTests() {
44
45       const finalScores = {                             // Initialize scores
46         security: 0,
47         reliability: 0,
48         chaos_resilience: 0,
49         tool_integrity: 100,
50         hallucination_risk: "high",
51         production_readiness: 0
52       };
53
54       const findings: any[] = [];
55
56       try {
57         // ═══════════════════════════════════════════════════════════════
58         // PHASE 1: INGESTION
59         // ═══════════════════════════════════════════════════════════════
60
61         emit("status", { phase: "Ingestion", active: true });
62                ↓ Browser receives, UI updates
63
64         emit("log", {
65           message: "[Ingestion] Checking if target is reachable...",
66           timestamp: Date.now()
67         });
68                ↓ Browser appends to logs array
69
70         try {
71           const controller = new AbortController();
72           const id = setTimeout(() => controller.abort(), 8000);
73           const resp = await fetch(targetUrl, {
74             method: "HEAD",
75             signal: controller.signal
76           });
77           clearTimeout(id);
78           const isReachable = resp.ok || resp.status < 500;
79         } catch (e) {
80           emit("log", { message: `Target unreachable: ${e.message}` });
81         }
82
83         emit("status", { phase: "Ingestion", active: false });
84                ↓ UI updates: Ingestion complete
85
86         // ═══════════════════════════════════════════════════════════════
87         // PHASE 2: SECURITY
88         // ═══════════════════════════════════════════════════════════════
89
90         emit("status", { phase: "Security", active: true });
91
92         const secResult = await runSecurityTest(targetUrl, {}, emit);
93                           ↓ Jump to: src/engine/runner.ts
94
95         finalScores.security = secResult.score;
96         findings.push(...secResult.findings);
97
98         emit("status", { phase: "Security", active: false });
99
100        // ═══════════════════════════════════════════════════════════════
101        // PHASE 3-8: Similar pattern (Chaos, Tools, Memory, Browser, Load, Hallucination)
102        // ═══════════════════════════════════════════════════════════════
103
104        // PHASE 7: Hallucination uses Gemini API
105        emit("status", { phase: "Hallucination", active: true });
106
107        const hallResult = await runHallucinationTest(targetUrl, {}, emit);
108                          ↓ Calls Gemini API
109
110        finalScores.hallucination_risk = hallResult.risk;
111        finalScores.tool_integrity = Math.round(
112          (finalScores.tool_integrity + hallResult.tool_integrity) / 2
113        );
114        findings.push(...hallResult.findings);
115
116        // ═══════════════════════════════════════════════════════════════
117        // CALCULATE FINAL SCORE
118        // ═══════════════════════════════════════════════════════════════
119
120        let baseScore = Math.round(
121          (finalScores.security +
122           finalScores.reliability +
123           finalScores.chaos_resilience +
124           finalScores.tool_integrity) /
125          4
126        );
127        // Calculate average: (82 + 75 + 88 + 87.5) / 4 = 83.125 → 83
128
129        if (finalScores.hallucination_risk === "high") {
130          baseScore -= 20;  // Penalize high risk
131        } else if (finalScores.hallucination_risk === "medium") {
132          baseScore -= 10;  // Penalize medium risk
133        }
134        // Result: 83 - 10 = 73
135
136        finalScores.production_readiness = Math.max(0, baseScore);
137        // Ensure it's never below 0
138
139        // ═══════════════════════════════════════════════════════════════
140        // SEND FINAL RESULTS
141        // ═══════════════════════════════════════════════════════════════
142
143        emit("complete", {
144          target: targetUrl,
145          scores: finalScores,
146          findings
147        });
148                ↓ Browser receives "complete" event
149                ↓ eventSource.onmessage fires with type === 'complete'
150                ↓ setResults(data) called
151                ↓ UI displays all results
152
153        res.end();                                        // Close connection
154                ↓ eventSource closes
155
156      } catch (error: any) {
157        emit("error", { message: error.message });
158        res.end();
159      }
160    }
161
162    // Don't wait, start tests asynchronously
163    runAllTests();                                        // ← Start, don't wait
164
165  });
166
167  // Vite middleware for development
168  if (process.env.NODE_ENV !== "production") {
169    const vite = await createViteServer({
170      server: { middlewareMode: true },
171      appType: "spa",
172    });
173    app.use(vite.middlewares);                           // Hot reload
174  } else {
175    // Production: serve static files
176    const distPath = path.join(process.cwd(), 'dist');
177    app.use(express.static(distPath));
177    app.get('*', (req, res) => {
178      res.sendFile(path.join(distPath, 'index.html'));
179    });
180  }
181
182  app.listen(PORT, "0.0.0.0", () => {
183    console.log(`✓ Server running on http://localhost:${PORT}`);
184  });
185 }
186
187 startServer();


FILE: src/engine/runner.ts
═══════════════════════════════════════════════════════════════════════════════

1  export async function runSecurityTest(
2    targetUrl: string,
3    headers: Record<string, string>,
4    emit: (type: string, data: any) => void
5  ) {
6    // Called from server.ts line 92
7
8    emit("log", {
9      message: "[Security] Checking for common vulnerabilities...",
9      timestamp: Date.now()
10   });
11            ↓ Browser receives, UI updates
12
13   // Simulate test (real implementation would check actual headers)
14   await new Promise(r => setTimeout(r, 2000));  // Wait 2 seconds
15
16   return {
17     score: 82,
18     findings: [
19       "Missing HTTPS enforcement",
20       "Missing rate limiting"
21     ]
22   };
23            ↓ Return to server.ts line 92
24            ↓ finalScores.security = 82
25 }
26
27 export async function runHallucinationTest(
28   targetUrl: string,
29   headers: Record<string, string>,
30   emit: (type: string, data: any) => void
31 ) {
32   // Called from server.ts line 107
33
34   emit("log", {
35     message: "[Hallucination] Analyzing responses for truthfulness...",
36     timestamp: Date.now()
37   });
38
39   // In real implementation:
40   // 1. Get AI responses
41   // 2. Call Gemini API
41   // 3. Analyze for hallucinations
42
43   // Simulate
44   await new Promise(r => setTimeout(r, 2000));
45
46   return {
47     risk: "medium",
47     tool_integrity: 85,
48     findings: [
49       "Found 2 potential hallucinations in test responses"
50     ]
51   };
52            ↓ Return to server.ts line 107
53            ↓ finalScores.hallucination_risk = "medium"
54 }
```

---

## 5️⃣ COMPLETE DATA TRANSFORMATION FLOW

```
INPUT → PROCESSING → OUTPUT

┌─────────────────────────────────────────────────────────────────────────────┐
│ USER INPUT                                                                   │
└─────────────────────────────────────────────────────────────────────────────┘

    Input in Browser:
    ─────────────────
    targetUrl = "https://api.example.com"
    
    
┌─────────────────────────────────────────────────────────────────────────────┐
│ ENCODE TO URL PARAMETER                                                      │
└─────────────────────────────────────────────────────────────────────────────┘

    JavaScript:
    const url = `/api/scan?url=${encodeURIComponent(targetUrl)}`
    
    Result:
    /api/scan?url=https%3A%2F%2Fapi.example.com


┌─────────────────────────────────────────────────────────────────────────────┐
│ HTTP REQUEST TO SERVER                                                       │
└─────────────────────────────────────────────────────────────────────────────┘

    Browser:
    GET /api/scan?url=https%3A%2F%2Fapi.example.com HTTP/1.1
    Host: localhost:3000
    Connection: keep-alive
    
    Server receives and decodes:
    targetUrl = "https://api.example.com"  ← Decoded back


┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 1: INGESTION                                                           │
└─────────────────────────────────────────────────────────────────────────────┘

    Input: targetUrl = "https://api.example.com"
    
    Processing:
    ──────────
    const resp = await fetch(targetUrl, { method: "HEAD" });
    
    Output:
    ──────
    resp.status = 200
    isReachable = true


┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 2: SECURITY TEST                                                       │
└─────────────────────────────────────────────────────────────────────────────┘

    Input: targetUrl = "https://api.example.com"
    
    Processing:
    ──────────
    Check response headers:
    - Has HTTPS? YES ✓
    - Has X-Frame-Options? NO ✗
    - Has Content-Security-Policy? NO ✗
    - Has Rate-Limiting? NO ✗
    
    Vulnerabilities found: 3
    Score = 100 - (3 × 10) = 70
    
    Output:
    ──────
    {
      score: 70,
      findings: [
        "Missing X-Frame-Options",
        "Missing Content-Security-Policy",
        "No rate limiting"
      ]
    }


┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 7: HALLUCINATION (WITH GEMINI API)                                    │
└─────────────────────────────────────────────────────────────────────────────┘

    Input: targetUrl = "https://api.example.com"
    
    Step 1: Get AI responses
    ───────────────────────
    const testPrompts = ["What is 2+2?", "What is capital of France?"];
    const responses = [
      "2+2=4",  // Truthful
      "Berlin"  // WRONG! Should be Paris (hallucination)
    ];
    
    Step 2: Send to Gemini API
    ──────────────────────────
    const client = new GoogleGenAI(process.env.GEMINI_API_KEY);
    const analysis = await client.models.generateContent({
      model: "gemini-pro",
      contents: [{
        role: "user",
        parts: [{
          text: `Analyze if these are hallucinations:
          
          Q: What is 2+2?
          A: 2+2=4 ✓ Truthful
          
          Q: What is capital of France?
          A: Berlin ✗ HALLUCINATION (correct answer is Paris)`
        }]
      }]
    });
    
    Step 3: Process response
    ───────────────────────
    analysis.text = "Found 1 hallucination out of 2. Risk: low"
    
    Output:
    ──────
    {
      risk: "low",
      tool_integrity: 85,
      findings: [
        "Found 1 hallucination: Capital of France should be Paris, not Berlin"
      ]
    }


┌─────────────────────────────────────────────────────────────────────────────┐
│ ALL SCORES COLLECTED                                                         │
└─────────────────────────────────────────────────────────────────────────────┘

    finalScores = {
      security: 70,
      reliability: 75,
      chaos_resilience: 88,
      tool_integrity: 87,
      hallucination_risk: "low",
      production_readiness: 0  ← Will be calculated
    }


┌─────────────────────────────────────────────────────────────────────────────┐
│ SCORE AGGREGATION & CALCULATION                                              │
└─────────────────────────────────────────────────────────────────────────────┘

    Step 1: Calculate average
    ────────────────────────
    (70 + 75 + 88 + 87) / 4 = 320 / 4 = 80
    
    Step 2: Apply hallucination penalty
    ───────────────────────────────────
    hallucination_risk = "low"
    if (hallucination_risk === "high") baseScore -= 20;
    else if (hallucination_risk === "medium") baseScore -= 10;
    else if (hallucination_risk === "low") baseScore -= 0;
    
    Final: 80 - 0 = 80
    
    Step 3: Ensure >= 0
    ──────────────────
    Math.max(0, 80) = 80


┌─────────────────────────────────────────────────────────────────────────────┐
│ FINAL RESULT OBJECT                                                          │
└─────────────────────────────────────────────────────────────────────────────┘

    {
      target: "https://api.example.com",
      scores: {
        security: 70,
        reliability: 75,
        chaos_resilience: 88,
        tool_integrity: 87,
        hallucination_risk: "low",
        production_readiness: 80
      },
      findings: [
        "Missing X-Frame-Options",
        "Missing Content-Security-Policy",
        "No rate limiting",
        "Response time degrades under load",
        "Found 1 hallucination..."
      ]
    }


┌─────────────────────────────────────────────────────────────────────────────┐
│ SEND VIA SSE                                                                 │
└─────────────────────────────────────────────────────────────────────────────┘

    Server:
    ───────
    emit("complete", { ... });
    
    SSE Format:
    ──────────
    data: {"type":"complete","data":{...}}\n\n
    
    Browser receives:
    ────────────────
    eventSource.onmessage fires
    const { type, data } = JSON.parse(event.data);
    setResults(data);
    
    UI Updates:
    ──────────
    Display score cards with colors:
    - 80 (green, >= 70)
    - 75 (yellow, >= 50)
    - 88 (green, >= 70)
    - etc.
    
    Display findings list
    Display recommendations
```

---

## 6️⃣ STATE MANAGEMENT FLOW (React)

```
COMPONENT: <App />
═════════════════════════════════════════════════════════════════════════════════

State Variables
───────────────

┌─ targetUrl: string
│  Initially: ""
│  User types: "https://api.example.com"
│  Updates: onChange={(e) => setTargetUrl(e.target.value)}
│
├─ scanning: boolean
│  Initially: false
│  User clicks: setScanning(true)
│  Scan complete: setScanning(false)
│  Effect: Button disabled while scanning
│
├─ results: object | null
│  Initially: null
│  After scan: setResults({ scores, findings })
│  Condition: {results && <ShowResults />}
│
└─ logs: string[]
   Initially: []
   On each message: setLogs(prev => [...prev, newLog])
   Display: logs.map((log) => <div>{log}</div>)


State Flow Diagram
──────────────────

                    Initial State
                    ─────────────
                    targetUrl: ""
                    scanning: false
                    results: null
                    logs: []
                          │
                          │
                    ┌─────┴──────┐
                    ▼            ▼
              User types     (No action)
              "https://..."   
                    │            │
                    └─────┬──────┘
                          │
                          ▼
                    Input captured
                    setTargetUrl("https://...")
                    State: { targetUrl: "https://..." }
                          │
                          │
                          ▼
                    User clicks "Run Scan"
                    onClick={startScan}
                          │
                    ┌─────┴─────────────────┐
                    │ startScan() executes │
                    ├─────────────────────┤
                    │ setScanning(true)   │  ← Button now disabled
                    │ setLogs([])         │  ← Clear previous logs
                    │ setResults(null)    │
                    │                     │
                    │ EventSource opened  │
                    │ SSE connection ──→ SERVER
                    └─────────────────────┘
                          │
                          ▼
                    Waiting for server...
                          │
                    ┌─────┴────────────────────────────────────┐
                    │ Messages arrive from server              │
                    ├─────────────────────────────────────────┤
                    │ message 1: type="status"                │
                    │ message 2: type="log" ──→ setLogs(...)  │
                    │ message 3: type="log" ──→ setLogs(...)  │
                    │ message 4: type="log" ──→ setLogs(...)  │
                    │ ...                                     │
                    │ message N: type="complete" ─────┐       │
                    └────────────────────────────────┼────────┘
                                                     │
                                    ┌────────────────┴────────────┐
                                    ▼                             ▼
                            setResults(data)              setScanning(false)
                            
                            State updated:
                            {
                              results: { 
                                scores: {...},
                                findings: [...]
                              },
                              scanning: false
                            }
                                    │
                                    ▼
                            React re-renders
                            
                            Conditional render:
                            {results && <div>Show score cards</div>}
                                    │
                                    ▼
                            UI displays:
                            ┌─────────────────────┐
                            │ Score Cards         │
                            │ Findings List       │
                            │ Recommendations     │
                            └─────────────────────┘


State Dependency Chain
──────────────────────

1. targetUrl changes
   └─ Input value updates
      └─ (No automatic action)

2. Click "Run Scan"
   └─ startScan() called
      ├─ setScanning(true)
      │  └─ Button becomes disabled
      │     └─ Input becomes disabled
      │
      ├─ setLogs([])
      │  └─ Clear previous logs from UI
      │
      └─ EventSource opens
         └─ Waits for server messages
            ├─ message arrives: type="log"
            │  └─ setLogs(prev => [...prev, message])
            │     └─ New log appears in UI
            │
            └─ message arrives: type="complete"
               ├─ setResults(data)
               │  └─ Results section appears
               │
               └─ setScanning(false)
                  └─ Button becomes enabled again
```

---

## 7️⃣ NETWORK REQUEST/RESPONSE DETAILS

```
REQUEST (Browser → Server)
═════════════════════════════════════════════════════════════════════════════════

GET /api/scan?url=https%3A%2F%2Fapi.example.com HTTP/1.1
Host: localhost:3000
User-Agent: Mozilla/5.0...
Connection: keep-alive
Accept: text/event-stream          ← Browser tells server: I want SSE
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9


RESPONSE HEADERS (Server → Browser)
═════════════════════════════════════════════════════════════════════════════════

HTTP/1.1 200 OK
Content-Type: text/event-stream    ← Signals this is a stream, not static content
Cache-Control: no-cache            ← Don't cache this response
Connection: keep-alive             ← Keep connection open
Transfer-Encoding: chunked         ← Send in chunks as available

(Connection stays open, server sends data as it becomes available)


RESPONSE BODY (Server → Browser) - Chunked Over Time
═════════════════════════════════════════════════════════════════════════════════

[0:00 - Immediately]
data: {"type":"status","data":{"phase":"Ingestion","active":true}}\n\n

[0:05 - After 5 seconds]
data: {"type":"log","data":{"message":"[Ingestion] Checking...","timestamp":1234}}\n\n

[0:08 - After 8 seconds]
data: {"type":"log","data":{"message":"Target is reachable","timestamp":1234}}\n\n

[0:10 - After 10 seconds]
data: {"type":"status","data":{"phase":"Ingestion","active":false}}\n\n
data: {"type":"status","data":{"phase":"Security","active":true}}\n\n

[0:12 - After 12 seconds]
data: {"type":"log","data":{"message":"[Security] Checking for...","timestamp":1234}}\n\n

... (more messages) ...

[0:25 - After 25 seconds]
data: {"type":"complete","data":{"target":"https://api.example.com","scores":{...},"findings":[...]}}\n\n

(Connection closes)


HOW BROWSER PROCESSES EACH CHUNK
═════════════════════════════════════════════════════════════════════════════════

Browser receives first chunk:
┌─────────────────────────────────────────────────────────────────┐
│ data: {"type":"status","data":{"phase":"Ingestion"...}}\n\n    │
└─────────────────────────────────────────────────────────────────┘
            ↓
   EventSource automatically parses
            ↓
   Extracts message: {"type":"status","data":{"phase":"Ingestion...}}
            ↓
   Fires: eventSource.onmessage(event)
            ↓
   JavaScript:
   const { type, data } = JSON.parse(event.data);
            ↓
   type = "status"
   data = { phase: "Ingestion", active: true }
            ↓
   if (type === 'status') {
     console.log('Phase:', data.phase);  ← Ingestion
   }


Browser receives second chunk (5 sec later):
┌──────────────────────────────────────────────────────────────────────┐
│ data: {"type":"log","data":{"message":"[Ingestion] Checking..."}}   │
└──────────────────────────────────────────────────────────────────────┘
            ↓
   eventSource.onmessage fires AGAIN
            ↓
   type = "log"
   data = { message: "[Ingestion] Checking..." }
            ↓
   if (type === 'log') {
     setLogs(prev => [...prev, data.message]);
   }
            ↓
   State updates
   logs = ["[Ingestion] Checking..."]
            ↓
   React re-renders
   <div>{logs[0]}</div> appears in UI
```

---

## 8️⃣ BUILD & DEPLOYMENT FLOW

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DEVELOPMENT (npm run dev)                            │
└─────────────────────────────────────────────────────────────────────────────┘

                           ts-node / tsx
                              ▼
                    server.ts (TypeScript)
                         ↓      ↓
                    React  Express
                         ↓      ↓
                    Vite HMR  Middleware
                         ↓
    User makes change in src/App.tsx
         ↓
    Vite detects change
         ↓
    Recompile only App.tsx
         ↓
    Send update to browser
         ↓
    React Hot Module Replacement
         ↓
    Component re-renders instantly
         ↓
    No page refresh! ← Very fast development


┌─────────────────────────────────────────────────────────────────────────────┐
│                    BUILD (npm run build)                                     │
└─────────────────────────────────────────────────────────────────────────────┘

Vite Frontend Build
────────────────────

src/main.tsx
├─ Import: react, react-dom
├─ Import: @/src/App.tsx
│  ├─ Import: lucide-react
│  ├─ Import: motion
│  └─ Uses: Tailwind classes
│
├─ Parse JSX → JavaScript
├─ Minify code
├─ Remove unused imports
├─ Create source maps
│
└─ Output:
   dist/index.html      (HTML skeleton)
   dist/assets/app.js   (bundled + minified)
   dist/assets/app.css  (compiled Tailwind)
   dist/assets/vendor.js (React, React-DOM, etc.)


esbuild Server Build
────────────────────

server.ts
├─ Import: express
├─ Import: ./src/engine/runner.ts
├─ Import: @google/genai
│
├─ Inline all imports
├─ Convert to CommonJS (dist/server.cjs)
├─ Mark external packages (don't bundle)
│  ├─ express
│  ├─ @google/genai
│  └─ etc. (users install these separately)
│
├─ Minify
├─ Create source map
│
└─ Output:
   dist/server.cjs (single bundle)


Full dist/ Directory After Build
─────────────────────────────────

dist/
├── index.html                    (40 bytes) - Skeleton
├── assets/
│   ├── app.js                    (120 KB)  - Frontend code
│   ├── app.css                   (15 KB)   - Tailwind styles
│   ├── vendor.js                 (200 KB)  - React, React-DOM
│   └── app.js.map                (180 KB)  - Source map for debugging
├── server.cjs                    (50 KB)   - Backend code
└── server.cjs.map                (100 KB)  - Server source map


┌─────────────────────────────────────────────────────────────────────────────┐
│                  PRODUCTION (npm start)                                      │
└─────────────────────────────────────────────────────────────────────────────┘

Starting Server
───────────────

npm start
  └─ Runs: node dist/server.cjs
     
     server.cjs loads:
     ├─ Express framework
     ├─ ./src/engine/runner.ts code (bundled inside)
     ├─ Loads .env file
     │  └─ GEMINI_API_KEY = "..."
     │
     └─ Listens on :3000


Handling Requests
─────────────────

User: GET http://example.com:3000
  │
  └─ Express receives
     │
     ├─ Check route: GET /api/scan? ← No match
     │  └─ Check: GET /api/health? ← No match
     │
     └─ Fall-through: Check if static file
        ├─ express.static(distPath)
        │  └─ Look in dist/
        │     ├─ Is it dist/index.html? YES
        │     └─ Serve it
        │
        └─ Browser receives dist/index.html
           │
           ├─ Parse HTML
           ├─ Load <script src="/assets/app.js">
           │  └─ Download dist/assets/app.js
           │
           ├─ Load <link href="/assets/app.css">
           │  └─ Download dist/assets/app.css
           │
           └─ Execute app.js
              └─ React mounts <App />


Handling /api/scan Requests
────────────────────────────

User: GET http://example.com:3000/api/scan?url=https://...
  │
  └─ Express checks routes
     │
     ├─ Route: GET /api/scan ← MATCH!
     │
     └─ Run handler: (req, res) => { ... }
        ├─ Extract targetUrl
        ├─ Set SSE headers
        ├─ Call: runAllTests()
        │  └─ This code is bundled inside dist/server.cjs
        │     ├─ All test functions available
        │     ├─ All imports resolved
        │     └─ Can call Gemini API
        │
        └─ Stream responses to client


File Size Comparison
────────────────────

Development (npm run dev):
- Entire src/ tree on disk
- node_modules/ ~500 MB
- TypeScript compilation on-demand
- Large downloads each request

Production (npm start):
- Only dist/server.cjs: 50 KB
- Only dist/assets/: 400 KB total
- Everything pre-compiled
- Minimal memory usage
- Fast startup time
- Result: Much faster, lighter, cheaper to host
```

---

## 9️⃣ ERROR HANDLING FLOW

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        SCENARIO: TARGET URL UNREACHABLE                      │
└─────────────────────────────────────────────────────────────────────────────┘

server.ts:
  Phase 1: Ingestion
    │
    ├─ emit("status", { phase: "Ingestion", active: true })
    │
    ├─ try {
    │   const resp = await fetch(targetUrl, { method: "HEAD" })
    │        ↓ Throws Error: "net::ERR_NAME_NOT_RESOLVED"
    │
    │ } catch (e) {
    │   emit("log", { message: `Target unreachable: ${e.message}` })
    │        ↓ Browser receives error message
    │        ↓ User sees: "Target unreachable: net::ERR_NAME_NOT_RESOLVED"
    │
    │   // Don't throw, continue with other tests
    │ }
    │
    └─ emit("status", { phase: "Ingestion", active: false })
       └─ Continue to Phase 2


Browser (App.tsx):
  eventSource.onmessage receives error log
    │
    ├─ type = "log"
    ├─ data.message = "Target unreachable: ..."
    │
    └─ setLogs(prev => [...prev, data.message])
       └─ UI shows error message (not in red, just logged)


Final Result:
  ✓ Scan completes despite error
  ✓ Other phases still run
  ✓ User sees what happened


┌─────────────────────────────────────────────────────────────────────────────┐
│                  SCENARIO: GEMINI API KEY MISSING                            │
└─────────────────────────────────────────────────────────────────────────────┘

server.ts:
  Phase 7: Hallucination
    │
    ├─ emit("status", { phase: "Hallucination", active: true })
    │
    ├─ try {
    │   const client = new GoogleGenAI(process.env.GEMINI_API_KEY)
    │                                   ↓ undefined (not set in .env)
    │
    │   const analysis = await client.models.generateContent(...)
    │        ↓ Throws Error: "API key is required"
    │
    │ } catch (error) {
    │   emit("log", { message: `Hallucination test failed: ${error.message}` })
    │        └─ User sees error but continues
    │ }
    │
    └─ emit("status", { phase: "Hallucination", active: false })
       └─ Continue (don't update hallucination_risk, use default)


Result:
  ✓ Scan doesn't crash
  ✓ User informed of failure
  ✓ Other scores still calculated
  ✗ hallucination_risk stays "high" (default)


┌─────────────────────────────────────────────────────────────────────────────┐
│                  SCENARIO: SERVER CRASHES DURING SCAN                        │
└─────────────────────────────────────────────────────────────────────────────┘

server.ts:
  runAllTests()
    │
    ├─ Phase 2 completes successfully
    ├─ Phase 3 starts
    │
    ├─ try {
    │   ... some code throws unexpected error ...
    │        ↓ Unhandled error bubbles up
    │
    │ } catch (error: any) {
    │   emit("error", { message: error.message })
    │        ↓ Send error to user
    │
    │   res.end()  ← Close connection gracefully
    │        ↓ No hanging connection
    │ }


Browser:
  eventSource.onerror fires
    │
    ├─ setScanning(false)
    │  └─ UI shows "Scan stopped"
    │
    └─ eventSource.close()
       └─ Stop listening for more data


Result:
  ✓ User informed
  ✓ Connection cleaned up
  ✗ Partial results lost (no final_scores sent)
  ✓ Server doesn't crash (error caught)
```

---

## 🔟 ENVIRONMENT & CONFIGURATION FLOW

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DEVELOPMENT SETUP                                    │
└─────────────────────────────────────────────────────────────────────────────┘

.env.example (Template)
───────────────────────
GEMINI_API_KEY=your_key_here
APP_URL=http://localhost:3000
NODE_ENV=development


.env (Actual)
─────────────
GEMINI_API_KEY=AIzaSyD... (your actual key)
APP_URL=http://localhost:3000
NODE_ENV=development

NOT in Git (in .gitignore)


Development Process
───────────────────

1. npm run dev
   └─ tsx server.ts
      ├─ Load .env using dotenv
      │  └─ process.env.GEMINI_API_KEY = "AIzaSyD..."
      │
      ├─ Load vite.config.ts
      │  └─ Pass GEMINI_API_KEY to frontend
      │
      └─ Start Express + Vite


2. Browser loads http://localhost:3000
   └─ vite.config.ts defines:
      ├─ define: { 'process.env.GEMINI_API_KEY': ... }
      │  └─ Makes API key available in frontend
      │
      └─ Vite injects into bundle


3. Frontend code:
   const apiKey = process.env.GEMINI_API_KEY
        ↓ "AIzaSyD..." (injected by Vite)


4. Backend code (server.ts):
   const client = new GoogleGenAI(process.env.GEMINI_API_KEY)
        ↓ process.env loaded from .env


┌─────────────────────────────────────────────────────────────────────────────┐
│                      PRODUCTION DEPLOYMENT                                   │
└─────────────────────────────────────────────────────────────────────────────┘

1. Build
   npm run build
   └─ Vite embeds GEMINI_API_KEY into dist/assets/app.js
   └─ esbuild bundles server.ts (no .env referenced)


2. Deploy dist/ to server
   └─ Copy dist/ to Cloud Run / Docker / VPS


3. Set environment variables
   └─ Export GEMINI_API_KEY="..."
   └─ Export NODE_ENV="production"
   └─ Export APP_URL="https://yourapp.com"


4. Start server
   npm start
   └─ node dist/server.cjs
      ├─ Load .env (if exists)
      ├─ Load process.env.GEMINI_API_KEY
      └─ Serve prebuilt files from dist/


5. Browser loads https://yourapp.com
   └─ Receives dist/index.html
   └─ Loads dist/assets/app.js (with embedded API key)


Configuration File: tsconfig.json
──────────────────────────────────

{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "jsx": "react-jsx",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "isolatedModules": true,
    "paths": {
      "@/*": ["./*"]           ← Import alias
    }
  }
}

Effect:
├─ import { x } from '@/src/engine/runner'
│  └─ Resolved to: /home/user/project/src/engine/runner.ts
│
└─ Instead of:
   import { x } from '../../../src/engine/runner'


Configuration File: vite.config.ts
──────────────────────────────────

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, '.', '');
  return {
    plugins: [react(), tailwindcss()],
    define: {
      'process.env.GEMINI_API_KEY': JSON.stringify(env.GEMINI_API_KEY),
    },
    resolve: {
      alias: {
        '@': path.resolve(__dirname, '.'),
      },
    },
  };
});

Effect:
├─ mode = "development" or "production"
├─ Load .env variables (if mode === development)
├─ Inject into frontend code
├─ React plugin handles .jsx/.tsx
└─ Tailwind plugin handles CSS
```

---

## 1️⃣1️⃣ COMPLETE SCAN TIMELINE (With actual times)

```
TIME    FRONTEND                    BACKEND                         NETWORK
════════════════════════════════════════════════════════════════════════════════

0:00    User enters URL             
        "https://api.example.com"   
        
        Clicks "Run Scan"           
        startScan() called           
        ┌──────────────────────────┐
        ├─ setScanning(true)        ├──── EventSource("/api/scan?url=...")
        ├─ setLogs([])              │     ▼ GET request sent
        └─ setResults(null)         └─────→ Server receives

0:01                                app.get("/api/scan") handler
                                    ├─ Validate URL
                                    ├─ Set SSE headers
                                    ├─ Create emit()
                                    └─ Call runAllTests()
                                       │
                                       ├─ emit("status": "Ingestion")
                                       ├─ WAIT 0.8s
        ◄─────────── SSE message ──┤
        eventSource.onmessage fires │
        UI: "Ingestion (running)"   │

0:06                                ├─ emit("log": "Checking...")
        ◄─────────── SSE message ──┤
        UI: Add log line            │

0:07                                ├─ Ping target server
                                    │  └─ await fetch(targetUrl)
                                    │     └─ Response: 200 OK
                                    │
                                    ├─ emit("log": "Target reachable")
        ◄─────────── SSE message ──┤
        UI: Add log line            │

0:08                                ├─ emit("status": "Ingestion complete")
        ◄─────────── SSE message ──┤
        UI: "Ingestion (complete)"  │

0:09                                ├─ Phase 2: Security
                                    ├─ emit("status": "Security")
        ◄─────────── SSE message ──┤
        UI: "Security (running)"    │
                                    │
                                    ├─ Call runSecurityTest()
                                    │  └─ emit("log": "Checking headers...")
        ◄─────────── SSE message ──┤
        UI: Add log line            │
                                    │
                                    ├─ WAIT 2s (simulate test)
                                    │
                                    ├─ emit("log": "Score: 82/100")
        ◄─────────── SSE message ──┤
        UI: Add log line            │

0:12                                ├─ emit("status": "Security complete")
        ◄─────────── SSE message ──┤
        UI: "Security (complete)"   │

0:13                                ├─ Phase 3: Chaos
                                    └─ ... similar pattern ...

0:16                                ├─ emit("status": "Chaos complete")
        ◄─────────── SSE message ──┤

                                    ├─ Phase 4-6 ...
                                    │  (Tool Validation, Memory, Browser)

0:22                                ├─ Phase 7: Load test
                                    ├─ emit("log": "Sending 100 requests...")
        ◄─────────── SSE message ──┤
        UI: Add log line            │
                                    │
                                    ├─ WAIT 2s
                                    │
                                    ├─ emit("log": "Score: 75/100")
        ◄─────────── SSE message ──┤

0:25                                ├─ Phase 8: Hallucination
                                    ├─ emit("log": "Sending to Gemini...")
        ◄─────────── SSE message ──┤
        UI: Add log line            │
                                    │
                                    ├─ fetch(targetUrl/query)
                                    │  └─ Get AI responses
                                    │
                                    ├─ GoogleGenAI.generateContent()
                                    │  └─ Call Gemini API (internet)
                                    │
                                    ├─ Parse response
                                    │
                                    ├─ emit("log": "Risk: medium")
        ◄─────────── SSE message ──┤
        UI: Add log line            │

0:28                                ├─ Calculate final scores
                                    │
                                    ├─ baseScore = 83
                                    ├─ hallucination penalty = -10
                                    ├─ production_readiness = 73
                                    │
                                    ├─ emit("complete": {...})
        ◄─────────── SSE message ──┤
        eventSource.onmessage       │
        setResults(data)            │
        setScanning(false)          │
        eventSource.close()         │
        
        UI renders:
        ┌──────────────────────┐   └─ res.end() close connection
        │ Score Cards          │
        │ Security: 82%  🟢    │
        │ Reliability: 75% 🟡  │
        │ Chaos: 88%    🟢     │
        │ Ready: 73%    🟡     │
        │                      │
        │ Findings:            │
        │ • Missing headers    │
        │ • Found 2 issues     │
        └──────────────────────┘

0:29    User can:
        - View results
        - Export report
        - Start new scan
        - Share with team
```

---

This comprehensive ASCII diagram set shows every aspect of the project's flow, from file structure to real-time data streaming to final UI rendering. Each diagram builds on the previous one to give a complete understanding of how all components work together!

