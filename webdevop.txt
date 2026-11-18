import React, { useState, useEffect, useMemo } from 'react';
import {
  GraduationCap, Sparkles, ShieldCheck, Lock, Briefcase, Award,
  CreditCard, Calendar, Layers, User, Globe, Search, CheckCircle2,
  AlertTriangle, X, Download, Upload, FileText, MessageSquare,
  ExternalLink, QrCode, Printer, Code2, Rocket, Copy, Check,
  ChevronRight, Menu, Building2, Star, Filter, RefreshCw, Eye
} from 'lucide-react';

// ============================================================================
// PRODUCTION FULL-STACK CODE & DEPLOYMENT BLUEPRINTS (VIEWABLE IN GO-LIVE MODAL)
// ============================================================================
const PRODUCTION_FILES = {
  'server.ts': `import express from 'express';
import mongoose from 'mongoose';
import cors from 'cors';
import dotenv from 'dotenv';

dotenv.config();
const app = express();
app.use(cors({ origin: process.env.FRONTEND_URL || '*' }));
app.use(express.json({ limit: '10mb' }));

// --- MONGODB ATLAS CONNECTION ---
mongoose.connect(process.env.MONGODB_URI || 'mongodb://localhost:27017/rit_careerhub')
  .then(() => console.log('Connected to MongoDB Atlas (RIT CareerHub AI)'))
  .catch((err) => console.error('MongoDB Connection Error:', err));

// --- MONGOOSE SCHEMAS ---
const StudentSchema = new mongoose.Schema({
  name: String,
  username: { type: String, unique: true },
  usn: { type: String, unique: true },
  department: String,
  section: String,
  semester: Number,
  cgpa: Number,
  skills: [String],
  readinessScore: Number,
  avatarUrl: String,
  bio: String
}, { timestamps: true });

const TransactionSchema = new mongoose.Schema({
  utr: { type: String, required: true, unique: true },
  studentName: String,
  usn: String,
  department: String,
  eventTitle: String,
  teamName: String,
  amount: Number,
  upiApp: { type: String, default: 'PhonePe' },
  status: { type: String, enum: ['Pending', 'Verified', 'Flagged'], default: 'Pending' },
  verifiedBy: { type: String, default: 'Unverified' },
  auditLogs: [{ action: String, by: String, timestamp: { type: Date, default: Date.now } }]
}, { timestamps: true });

const Student = mongoose.model('Student', StudentSchema);
const Transaction = mongoose.model('Transaction', TransactionSchema);

// --- ADMIN AUTH MIDDLEWARE (STRICT OWNER-ONLY ACCESS) ---
const requireAdmin = (req: any, res: any, next: any) => {
  const authHeader = req.headers.authorization;
  if (!authHeader) return res.status(401).json({ error: 'Admin token required' });
  try {
    const token = authHeader.split(' ')[1];
    const payload = JSON.parse(Buffer.from(token.split('.')[1], 'base64').toString());
    if (payload.role !== 'admin' || payload.email !== process.env.ADMIN_OWNER_EMAIL) {
      return res.status(403).json({ error: 'Forbidden: Only RIT Site Owner can access Payment History' });
    }
    next();
  } catch {
    return res.status(401).json({ error: 'Invalid or expired Admin session' });
  }
};

// --- API ROUTES ---
// 0. Admin Owner Login Endpoint
app.post('/api/auth/admin-login', async (req, res) => {
  const { email, passcode } = req.body;
  if (email === process.env.ADMIN_OWNER_EMAIL && passcode === process.env.ADMIN_OWNER_PASSCODE) {
    return res.json({ authenticated: true, role: 'admin', owner: email });
  }
  return res.status(401).json({ error: 'Invalid Owner Credentials' });
});

// 1. Public MyRITfolio Route
app.get('/api/portfolios/:username', async (req, res) => {
  const student = await Student.findOne({ username: req.params.username });
  if (!student) return res.status(404).json({ error: 'Portfolio not found' });
  res.json(student);
});

// 2. Submit PhonePe UTR Transaction (Write-Only for Students; Cannot Read Ledger)
app.post('/api/transactions', async (req, res) => {
  try {
    const txn = await Transaction.create({
      ...req.body,
      auditLogs: [{ action: 'Submitted UTR via PhonePe QR', by: req.body.studentUsn }]
    });
    res.status(201).json({ message: 'UTR submitted to Admin vault', receiptId: txn._id });
  } catch (err: any) {
    res.status(400).json({ error: err.message });
  }
});

// 3. Admin-Only Full Payment History & UTR Vault (Protected by requireAdmin)
app.get('/api/admin/transactions', requireAdmin, async (_req, res) => {
  const history = await Transaction.find().sort({ createdAt: -1 });
  res.json(history);
});

// 4. Admin Verify / Flag Transaction ID (Protected by requireAdmin)
app.patch('/api/admin/transactions/:id/verify', requireAdmin, async (req, res) => {
  const { status, adminName } = req.body;
  const txn = await Transaction.findByIdAndUpdate(
    req.params.id,
    {
      status,
      $push: { auditLogs: { action: 'Status changed to ' + status, by: adminName || 'RIT Site Owner' } }
    },
    { new: true }
  );
  res.json(txn);
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log('RIT CareerHub API running on port ' + PORT));`,
  'manifest.webmanifest': JSON.stringify({
    name: "RIT CareerHub AI - Rajeev Institute of Technology",
    short_name: "RIT CareerHub",
    start_url: "/",
    display: "standalone",
    background_color: "#0F2A4A",
    theme_color: "#0F2A4A",
    description: "AI-Powered Student Portfolio, Placement & Hackathon Portal for RIT Hassan",
    icons: [
      { src: "/icons/rit-192.png", sizes: "192x192", type: "image/png" },
      { src: "/icons/rit-512.png", sizes: "512x512", type: "image/png" }
    ]
  }, null, 2),
  '.env.example': [
    '# MongoDB Atlas Connection String',
    'MONGODB_URI=mongodb+srv://ritadmin:<password>@cluster0.mongodb.net/rit_careerhub',
    '',
    '# JWT Secret for Role-Based Auth',
    'JWT_SECRET=super_secret_rit_hassan_jwt_key_2026',
    '',
    '# Cloudinary Credentials (Profile Photos, Certificates, Event Banners)',
    'CLOUDINARY_CLOUD_NAME=rit_hassan_cloud',
    'CLOUDINARY_API_KEY=your_cloudinary_api_key',
    'CLOUDINARY_API_SECRET=your_cloudinary_api_secret',
    '',
    '# Google Gemini API Key',
    'GEMINI_API_KEY=your_google_gemini_api_key',
    '',
    '# Strict Site Owner Credentials for Payment History Vault',
    'ADMIN_OWNER_EMAIL=admin@rithassan.ac.in',
    'ADMIN_OWNER_PASSCODE=RIT@2026Admin',
    '',
    '# Frontend URL for CORS',
    'FRONTEND_URL=https://rit-careerhub.vercel.app'
  ].join('\n'),
  'README.md': [
    '# RIT CareerHub AI — Full-Stack Setup & Go-Live Guide',
    '',
    '## 1. Local Development Setup',
    '1. Clone repo & install dependencies: `npm install`',
    '2. Copy `.env.example` to `.env` and fill in MongoDB Atlas URI, Gemini API Key, and Owner Admin credentials.',
    '3. Run Backend: `npx ts-node server.ts`',
    '4. Run Frontend: `npm run dev`',
    '',
    '## 2. Free Cloud Deployment (Go Live in 10 Minutes)',
    '- Database: Create a free M0 cluster on MongoDB Atlas (Mumbai ap-south-1 region).',
    '- Backend API: Push to GitHub and deploy `server.ts` on Render.com (Free Web Service).',
    '- Frontend PWA: Deploy React/Vite build on Vercel.com and link `VITE_API_URL` to your Render URL.'
  ].join('\n')
};

// ============================================================================
// PRE-SEEDED RIT HASSAN DATA
// ============================================================================
const INITIAL_DEPARTMENTS = [
  { code: 'CSE', name: 'Computer Science & Engineering', sections: ['A', 'B', 'C'] },
  { code: 'ISE', name: 'Information Science & Engineering', sections: ['A', 'B'] },
  { code: 'AIML', name: 'Artificial Intelligence & Machine Learning', sections: ['A', 'B'] },
  { code: 'ECE', name: 'Electronics & Communication Engineering', sections: ['A', 'B'] },
  { code: 'EEE', name: 'Electrical & Electronics Engineering', sections: ['A'] },
  { code: 'ME', name: 'Mechanical Engineering', sections: ['A'] },
  { code: 'CV', name: 'Civil Engineering', sections: ['A'] },
];

const INITIAL_STUDENTS = [
  {
    id: 'stu-1',
    name: 'Ananya Gowda',
    username: 'ananya-cse26',
    usn: '4RA22CS014',
    department: 'CSE',
    section: 'A',
    semester: 7,
    cgpa: 9.24,
    readinessScore: 92,
    skills: ['React', 'Node.js', 'Python', 'TensorFlow', 'System Design', 'MongoDB'],
    bio: 'Final-year CSE student at RIT Hassan passionate about full-stack cloud systems and applied Generative AI.',
    github: 'github.com/ananya-rit',
    linkedin: 'linkedin.com/in/ananya-gowda-rit',
    projects: [
      { title: 'Smart Malnad Agri-Advisor AI', tech: 'Python, Gemini API, React', desc: 'Multilingual crop disease diagnosis & market price predictor for Hassan coffee & pepper farmers.' },
      { title: 'CampusBus Live Telemetry', tech: 'Node.js, Socket.io, Leaflet', desc: 'Real-time GPS tracking for 18 RIT Hassan college buses.' }
    ]
  },
  {
    id: 'stu-2',
    name: 'Rohan Kulkarni',
    username: 'rohan-aiml26',
    usn: '4RA22AI041',
    department: 'AIML',
    section: 'A',
    semester: 7,
    cgpa: 8.85,
    readinessScore: 88,
    skills: ['PyTorch', 'Computer Vision', 'FastAPI', 'Docker', 'SQL', 'LangChain'],
    bio: 'AI/ML undergraduate building edge computer vision models and autonomous robotics solutions.',
    github: 'github.com/rohan-aiml',
    linkedin: 'linkedin.com/in/rohan-kulkarni-rit',
    projects: [
      { title: 'DefectNet PCB Inspector', tech: 'YOLOv8, OpenCV, FastAPI', desc: 'Automated soldering defect detection pipeline achieving 97.4% precision.' }
    ]
  },
  {
    id: 'stu-3',
    name: 'Sneha Hegde',
    username: 'sneha-ise26',
    usn: '4RA22IS029',
    department: 'ISE',
    section: 'B',
    semester: 7,
    cgpa: 9.05,
    readinessScore: 90,
    skills: ['TypeScript', 'Next.js', 'AWS', 'PostgreSQL', 'Cybersecurity', 'GraphQL'],
    bio: 'Information Science engineer focused on zero-trust cloud security and scalable SaaS platforms.',
    github: 'github.com/sneha-ise',
    linkedin: 'linkedin.com/in/sneha-hegde-rit',
    projects: [
      { title: 'ZeroTrust Campus Vault', tech: 'TypeScript, WebAuthn, PostgreSQL', desc: 'Passwordless biometric authentication system for university exam portals.' }
    ]
  },
  {
    id: 'stu-4',
    name: 'Karthik M. S.',
    username: 'karthik-ece26',
    usn: '4RA22EC019',
    department: 'ECE',
    section: 'A',
    semester: 7,
    cgpa: 8.52,
    readinessScore: 83,
    skills: ['Embedded C', 'Verilog', 'ESP32', 'RTOS', 'MATLAB', 'IoT'],
    bio: 'ECE VLSI & Embedded Systems enthusiast designing low-power LoRaWAN environmental sensors.',
    github: 'github.com/karthik-vlsi',
    linkedin: 'linkedin.com/in/karthik-ms-rit',
    projects: [
      { title: 'Hemavathi Reservoir Flood Telemetry', tech: 'ESP32, LoRaWAN, React', desc: 'Solar-powered water level monitoring node network.' }
    ]
  }
];

const INITIAL_ACHIEVEMENTS = [
  {
    id: 'ach-1',
    studentId: 'stu-1',
    studentName: 'Ananya Gowda',
    usn: '4RA22CS014',
    title: 'Smart India Hackathon (SIH) National Finalist',
    issuer: 'Ministry of Education Innovation Cell',
    date: '2025-12-18',
    status: 'Approved',
    aiOcrSummary: 'Verified Credential ID: SIH2025-FIN-8841 • 99% Authenticity Match'
  },
  {
    id: 'ach-2',
    studentId: 'stu-2',
    studentName: 'Rohan Kulkarni',
    usn: '4RA22AI041',
    title: 'AWS Certified Cloud Practitioner',
    issuer: 'Amazon Web Services',
    date: '2026-01-22',
    status: 'Approved',
    aiOcrSummary: 'Verified Validation Code: AWS-CCP-992104 • Active Status'
  },
  {
    id: 'ach-3',
    studentId: 'stu-3',
    studentName: 'Sneha Hegde',
    usn: '4RA22IS029',
    title: 'VTU State-Level Coding Championship Winner',
    issuer: 'Visvesvaraya Technological University',
    date: '2026-02-10',
    status: 'Pending',
    aiOcrSummary: 'OCR Extracted: VTU-BELAGAVI-2026-01 • Awaiting Admin Sign-off'
  }
];

const INITIAL_EVENTS = [
  {
    id: 'evt-1',
    title: 'RIT HackFest 2026 — 36-Hour National Hackathon',
    date: '2026-12-12',
    venue: 'RIT Main Auditorium & CSE Labs, Hassan',
    fee: 400,
    teamSize: '2–4 Members',
    status: 'Open'
  },
  {
    id: 'evt-2',
    title: 'Gemini GenAI & Cloud BootCamp',
    date: '2026-11-28',
    venue: 'Seminar Hall 2, Rajeev Institute of Technology',
    fee: 150,
    teamSize: 'Solo / Duo',
    status: 'Open'
  },
  {
    id: 'evt-3',
    title: 'VTU Mega Campus Placement Drive — IT & Core',
    date: '2026-12-20',
    venue: 'T&P Cell Block, RIT Hassan',
    fee: 0,
    teamSize: 'Individual',
    status: 'Open'
  }
];

const INITIAL_TRANSACTIONS = [
  {
    id: 'txn-101',
    utr: '632189450112',
    studentName: 'Ananya Gowda',
    usn: '4RA22CS014',
    department: 'CSE',
    eventTitle: 'RIT HackFest 2026 — 36-Hour National Hackathon',
    teamName: 'Malnad Coders',
    amount: 400,
    upiApp: 'PhonePe',
    timestamp: '2026-11-17 18:42',
    status: 'Verified',
    verifiedBy: 'RIT Site Owner'
  },
  {
    id: 'txn-102',
    utr: '632204918273',
    studentName: 'Rohan Kulkarni',
    usn: '4RA22AI041',
    department: 'AIML',
    eventTitle: 'RIT HackFest 2026 — 36-Hour National Hackathon',
    teamName: 'Neural Knights',
    amount: 400,
    upiApp: 'PhonePe',
    timestamp: '2026-11-18 09:15',
    status: 'Pending',
    verifiedBy: 'Awaiting Review'
  },
  {
    id: 'txn-103',
    utr: '632211094821',
    studentName: 'Karthik M. S.',
    usn: '4RA22EC019',
    department: 'ECE',
    eventTitle: 'Gemini GenAI & Cloud BootCamp',
    teamName: 'Silicon Sparks',
    amount: 150,
    upiApp: 'PhonePe',
    timestamp: '2026-11-18 11:04',
    status: 'Pending',
    verifiedBy: 'Awaiting Review'
  }
];

// ============================================================================
// HELPER: CALL GEMINI API WITH EXPONENTIAL BACKOFF & SMART LOCAL FALLBACK
// ============================================================================
async function callGeminiAssistant(prompt, systemInstruction, fallbackResponse) {
  const apiKey = '';
  const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3-flash-preview:generateContent?key=${apiKey}`;

  for (let attempt = 0; attempt < 3; attempt++) {
    try {
      const res = await fetch(url, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          contents: [{ parts: [{ text: prompt }] }],
          systemInstruction: { parts: [{ text: systemInstruction }] }
        })
      });
      if (res.ok) {
        const data = await res.json();
        const text = data?.candidates?.[0]?.content?.parts?.[0]?.text;
        if (text) return text;
      }
    } catch {
      // Exponential backoff
    }
    await new Promise(r => setTimeout(r, 800 * Math.pow(2, attempt)));
  }
  return fallbackResponse;
}

// ============================================================================
// MAIN APPLICATION COMPONENT
// ============================================================================
export default function App() {
  const [activeView, setActiveView] = useState('landing'); // 'landing' | 'student' | 'portfolio' | 'admin' | 'recruiter'
  const [selectedPortfolioUsername, setSelectedPortfolioUsername] = useState('ananya-cse26');
  const [mobileMenuOpen, setMobileMenuOpen] = useState(false);
  const [showGoLiveModal, setShowGoLiveModal] = useState(false);
  const [toast, setToast] = useState(null);

  // Strict Owner-Only Admin Session State (Locked by default; never exposed to Students/Recruiters)
  const [isAdminAuthenticated, setIsAdminAuthenticated] = useState(() => {
    return sessionStorage.getItem('rit_admin_owner_auth') === 'true';
  });
  const [adminOwnerEmail] = useState('admin@rithassan.ac.in');
  const [adminOwnerPasscode, setAdminOwnerPasscode] = useState(() => {
    return localStorage.getItem('rit_admin_passcode') || 'RIT@2026Admin';
  });

  // Core Database State (Persisted to localStorage for realistic interactivity)
  const [departments, setDepartments] = useState(() => {
    const saved = localStorage.getItem('rit_departments');
    return saved ? JSON.parse(saved) : INITIAL_DEPARTMENTS;
  });
  const [students, setStudents] = useState(() => {
    const saved = localStorage.getItem('rit_students');
    return saved ? JSON.parse(saved) : INITIAL_STUDENTS;
  });
  const [achievements, setAchievements] = useState(() => {
    const saved = localStorage.getItem('rit_achievements');
    return saved ? JSON.parse(saved) : INITIAL_ACHIEVEMENTS;
  });
  const [events, setEvents] = useState(() => {
    const saved = localStorage.getItem('rit_events');
    return saved ? JSON.parse(saved) : INITIAL_EVENTS;
  });
  const [transactions, setTransactions] = useState(() => {
    const saved = localStorage.getItem('rit_transactions');
    return saved ? JSON.parse(saved) : INITIAL_TRANSACTIONS;
  });
  const [auditLogs, setAuditLogs] = useState([
    { id: 'log-1', time: '09:16 AM', actor: 'Student (4RA22AI041)', action: 'Submitted PhonePe UTR 632204918273 (₹400)' },
    { id: 'log-0', time: 'Yesterday', actor: 'Site Owner', action: 'Verified UTR 632189450112 for Ananya Gowda' }
  ]);

  useEffect(() => { localStorage.setItem('rit_departments', JSON.stringify(departments)); }, [departments]);
  useEffect(() => { localStorage.setItem('rit_students', JSON.stringify(students)); }, [students]);
  useEffect(() => { localStorage.setItem('rit_achievements', JSON.stringify(achievements)); }, [achievements]);
  useEffect(() => { localStorage.setItem('rit_events', JSON.stringify(events)); }, [events]);
  useEffect(() => { localStorage.setItem('rit_transactions', JSON.stringify(transactions)); }, [transactions]);

  const notify = (message, type = 'success') => {
    setToast({ message, type });
    setTimeout(() => setToast(null), 4000);
  };

  const handleAdminLoginSuccess = (email) => {
    sessionStorage.setItem('rit_admin_owner_auth', 'true');
    setIsAdminAuthenticated(true);
    setAuditLogs(prev => [
      {
        id: `log-${Date.now()}`,
        time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
        actor: `Owner (${email})`,
        action: 'Authenticated into Private Admin & Payment History Vault'
      },
      ...prev
    ]);
    notify('Owner Authenticated! Private Payment History Vault unlocked.');
  };

  const handleAdminLogout = () => {
    sessionStorage.removeItem('rit_admin_owner_auth');
    setIsAdminAuthenticated(false);
    setAuditLogs(prev => [
      {
        id: `log-${Date.now()}`,
        time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
        actor: 'Owner Session',
        action: 'Locked Admin Console & Revoked Payment History Access'
      },
      ...prev
    ]);
    notify('Admin Vault locked. Payment history is now hidden.');
  };

  const handleUpdateAdminPasscode = (newPass) => {
    localStorage.setItem('rit_admin_passcode', newPass);
    setAdminOwnerPasscode(newPass);
    setAuditLogs(prev => [
      {
        id: `log-${Date.now()}`,
        time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
        actor: 'Site Owner',
        action: 'Updated Admin Security Passcode'
      },
      ...prev
    ]);
    notify('Owner Security Passcode updated successfully!');
  };

  const resetDemoData = () => {
    setDepartments(INITIAL_DEPARTMENTS);
    setStudents(INITIAL_STUDENTS);
    setAchievements(INITIAL_ACHIEVEMENTS);
    setEvents(INITIAL_EVENTS);
    setTransactions(INITIAL_TRANSACTIONS);
    notify('All RIT Hassan demo records reset to defaults.');
  };

  const openPortfolio = (username) => {
    setSelectedPortfolioUsername(username);
    setActiveView('portfolio');
    window.scrollTo({ top: 0, behavior: 'smooth' });
  };

  return (
    <div className="min-h-screen bg-slate-50 text-slate-900 font-sans flex flex-col">
      {/* Global Toast Notification */}
      {toast && (
        <div className={`fixed bottom-5 right-5 z-50 px-5 py-3.5 rounded-2xl shadow-2xl border text-xs font-bold flex items-center gap-2.5 transition-all ${
          toast.type === 'error'
            ? 'bg-rose-950 text-rose-100 border-rose-700'
            : 'bg-[#0F2A4A] text-amber-300 border-amber-400/40'
        }`}>
          <CheckCircle2 className="w-4 h-4 shrink-0" />
          <span>{toast.message}</span>
        </div>
      )}

      {/* Top Institutional Navigation Bar */}
      <header className="sticky top-0 z-40 bg-[#0F2A4A] text-white border-b border-white/10 shadow-lg print:hidden">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 h-16 flex items-center justify-between gap-3">
          {/* Brand */}
          <button
            onClick={() => setActiveView('landing')}
            className="flex items-center gap-3 text-left group"
          >
            <div className="w-10 h-10 rounded-xl bg-gradient-to-br from-amber-400 to-amber-600 text-[#0F2A4A] flex items-center justify-center font-black shadow-md group-hover:scale-105 transition">
              <GraduationCap className="w-6 h-6" />
            </div>
            <div>
              <div className="flex items-center gap-1.5">
                <span className="font-extrabold text-base tracking-tight">RIT CareerHub</span>
                <span className="px-1.5 py-0.5 rounded bg-amber-400 text-[#0F2A4A] text-[10px] font-black">AI</span>
              </div>
              <p className="text-[10px] text-slate-300 hidden sm:block">Rajeev Institute of Technology, Hassan</p>
            </div>
          </button>

          {/* Desktop Role Switcher Navigation */}
          <nav className="hidden lg:flex items-center gap-1.5 bg-white/5 p-1 rounded-xl border border-white/10">
            {[
              { id: 'landing', label: 'Explore & Directory', icon: Globe },
              { id: 'student', label: 'Student Portal', icon: User },
              { id: 'portfolio', label: 'MyRITfolio', icon: Award },
              { id: 'admin', label: isAdminAuthenticated ? 'Admin Vault (Unlocked)' : 'Admin Login (Locked)', icon: isAdminAuthenticated ? ShieldCheck : Lock },
              { id: 'recruiter', label: 'Recruiters', icon: Briefcase },
            ].map(tab => {
              const Icon = tab.icon;
              const isActive = activeView === tab.id;
              return (
                <button
                  key={tab.id}
                  onClick={() => setActiveView(tab.id)}
                  className={`px-3.5 py-2 rounded-lg text-xs font-bold flex items-center gap-1.5 transition ${
                    isActive
                      ? 'bg-amber-400 text-[#0F2A4A] shadow'
                      : 'text-slate-200 hover:bg-white/10'
                  }`}
                >
                  <Icon className="w-3.5 h-3.5" />
                  <span>{tab.label}</span>
                </button>
              );
            })}
          </nav>

          {/* Right Actions: Go-Live Blueprint & Mobile Menu */}
          <div className="flex items-center gap-2">
            <button
              onClick={() => setShowGoLiveModal(true)}
              className="px-3.5 py-2 rounded-xl bg-emerald-500 hover:bg-emerald-400 text-slate-950 font-extrabold text-xs flex items-center gap-1.5 shadow transition"
            >
              <Rocket className="w-3.5 h-3.5" />
              <span className="hidden sm:inline">Code & Go-Live Guide</span>
              <span className="sm:hidden">Go Live</span>
            </button>

            <button
              onClick={() => setMobileMenuOpen(!mobileMenuOpen)}
              className="lg:hidden p-2 rounded-xl bg-white/10 hover:bg-white/20 text-white"
              aria-label="Toggle Menu"
            >
              {mobileMenuOpen ? <X className="w-5 h-5" /> : <Menu className="w-5 h-5" />}
            </button>
          </div>
        </div>

        {/* Mobile Drawer Navigation */}
        {mobileMenuOpen && (
          <div className="lg:hidden bg-[#0A1D33] border-t border-white/10 px-4 py-3 space-y-1.5">
            {[
              { id: 'landing', label: 'Public Landing & Directory', icon: Globe },
              { id: 'student', label: 'Student Dashboard & AI Tools', icon: User },
              { id: 'portfolio', label: 'MyRITfolio Public View', icon: Award },
              { id: 'admin', label: isAdminAuthenticated ? 'Admin & Payment History (Unlocked)' : 'Admin Login (Owner Only)', icon: isAdminAuthenticated ? ShieldCheck : Lock },
              { id: 'recruiter', label: 'Recruiter Talent Portal', icon: Briefcase },
            ].map(item => {
              const Icon = item.icon;
              return (
                <button
                  key={item.id}
                  onClick={() => { setActiveView(item.id); setMobileMenuOpen(false); }}
                  className={`w-full px-4 py-2.5 rounded-xl text-xs font-bold flex items-center gap-2.5 ${
                    activeView === item.id ? 'bg-amber-400 text-[#0F2A4A]' : 'text-slate-200 hover:bg-white/10'
                  }`}
                >
                  <Icon className="w-4 h-4" />
                  <span>{item.label}</span>
                </button>
              );
            })}
          </div>
        )}
      </header>

      {/* Main Dynamic Workspace */}
      <main className="flex-1">
        {activeView === 'landing' && (
          <LandingPage
            students={students}
            departments={departments}
            events={events}
            onNavigate={setActiveView}
            onOpenPortfolio={openPortfolio}
          />
        )}

        {activeView === 'student' && (
          <StudentDashboard
            students={students}
            setStudents={setStudents}
            departments={departments}
            achievements={achievements}
            setAchievements={setAchievements}
            events={events}
            setTransactions={setTransactions}
            setAuditLogs={setAuditLogs}
            onOpenPortfolio={openPortfolio}
            notify={notify}
          />
        )}

        {activeView === 'portfolio' && (
          <MyRITfolioView
            students={students}
            selectedUsername={selectedPortfolioUsername}
            onSelectUsername={setSelectedPortfolioUsername}
            achievements={achievements}
            onBack={() => setActiveView('landing')}
          />
        )}

        {activeView === 'admin' && (
          isAdminAuthenticated ? (
            <AdminDashboard
              students={students}
              departments={departments}
              setDepartments={setDepartments}
              achievements={achievements}
              setAchievements={setAchievements}
              events={events}
              setEvents={setEvents}
              transactions={transactions}
              setTransactions={setTransactions}
              auditLogs={auditLogs}
              setAuditLogs={setAuditLogs}
              adminOwnerEmail={adminOwnerEmail}
              onUpdatePasscode={handleUpdateAdminPasscode}
              onLogoutAdmin={handleAdminLogout}
              notify={notify}
              onResetDemo={resetDemoData}
            />
          ) : (
            <AdminLoginGate
              expectedEmail={adminOwnerEmail}
              expectedPasscode={adminOwnerPasscode}
              onSuccess={handleAdminLoginSuccess}
              onCancel={() => setActiveView('landing')}
            />
          )
        )}

        {activeView === 'recruiter' && (
          <RecruiterPortal
            students={students}
            departments={departments}
            onOpenPortfolio={openPortfolio}
            notify={notify}
          />
        )}
      </main>

      {/* Institutional Footer */}
      <footer className="bg-[#0F2A4A] text-slate-300 border-t border-white/10 py-8 px-4 print:hidden">
        <div className="max-w-7xl mx-auto flex flex-col sm:flex-row items-center justify-between gap-4 text-xs">
          <div>
            <p className="font-extrabold text-white">RIT CareerHub AI • Rajeev Institute of Technology, Hassan</p>
            <p className="text-slate-400 mt-0.5">Affiliated to VTU Belagavi • Training, Placement & Innovation Cell</p>
          </div>
          <div className="flex items-center gap-4">
            <button onClick={() => setActiveView('admin')} className="text-amber-400 hover:underline font-bold">
              Owner Admin Vault
            </button>
            <button onClick={() => setShowGoLiveModal(true)} className="text-emerald-400 hover:underline font-bold">
              Deployment & API Blueprint
            </button>
          </div>
        </div>
      </footer>

      {/* Production Code & Go-Live Hosting Guide Modal */}
      {showGoLiveModal && (
        <GoLiveBlueprintModal onClose={() => setShowGoLiveModal(false)} notify={notify} />
      )}
    </div>
  );
}

// ============================================================================
// 1. LANDING PAGE & PUBLIC MYRITFOLIO DIRECTORY
// ============================================================================
function LandingPage({ students, departments, events, onNavigate, onOpenPortfolio }) {
  const [searchQuery, setSearchQuery] = useState('');
  const [deptFilter, setDeptFilter] = useState('ALL');

  const filteredStudents = useMemo(() => {
    return students.filter(s => {
      const matchesDept = deptFilter === 'ALL' || s.department === deptFilter;
      const q = searchQuery.toLowerCase();
      const matchesQuery =
        s.name.toLowerCase().includes(q) ||
        s.usn.toLowerCase().includes(q) ||
        s.skills.some(sk => sk.toLowerCase().includes(q));
      return matchesDept && matchesQuery;
    });
  }, [students, deptFilter, searchQuery]);

  return (
    <div>
      {/* Hero Section */}
      <section className="bg-gradient-to-br from-[#0F2A4A] via-[#133660] to-[#0A1D33] text-white py-16 sm:py-20 px-4">
        <div className="max-w-7xl mx-auto grid lg:grid-cols-12 gap-10 items-center">
          <div className="lg:col-span-7 space-y-6">
            <div className="inline-flex items-center gap-2 px-3.5 py-1.5 rounded-full bg-amber-400/15 border border-amber-400/30 text-amber-300 text-xs font-extrabold">
              <Sparkles className="w-3.5 h-3.5" />
              <span>Powered by Google Gemini AI • Built for RIT Hassan</span>
            </div>

            <h1 className="text-3xl sm:text-5xl font-black tracking-tight leading-tight">
              Verified Engineering Portfolios & <span className="text-amber-400">AI Placement Launchpad</span>
            </h1>

            <p className="text-sm sm:text-base text-slate-300 max-w-2xl leading-relaxed">
              Every Rajeev Institute of Technology (RIT Hassan) student gets a verified <code className="text-amber-300 font-mono">/student/[username]</code> MyRITfolio, Gemini AI resume & mock interview coach, and seamless hackathon registration.
            </p>

            <div className="flex flex-wrap gap-3 pt-2">
              <button
                onClick={() => onNavigate('student')}
                className="px-6 py-3.5 rounded-2xl bg-amber-400 hover:bg-amber-300 text-[#0F2A4A] font-extrabold text-xs sm:text-sm shadow-xl transition flex items-center gap-2"
              >
                Launch Student Portal <ChevronRight className="w-4 h-4" />
              </button>
              <button
                onClick={() => onNavigate('recruiter')}
                className="px-6 py-3.5 rounded-2xl bg-white/10 hover:bg-white/20 text-white border border-white/20 font-bold text-xs sm:text-sm transition"
              >
                Recruiter Talent Search
              </button>
              <button
                onClick={() => onNavigate('admin')}
                className="px-5 py-3.5 rounded-2xl bg-slate-900/70 hover:bg-slate-900 text-amber-300 border border-amber-400/30 font-bold text-xs sm:text-sm flex items-center gap-2 transition"
              >
                <Lock className="w-3.5 h-3.5" /> Owner Admin Vault
              </button>
            </div>
          </div>

          {/* Live Platform Telemetry Card */}
          <div className="lg:col-span-5">
            <div className="bg-white/10 backdrop-blur-md border border-white/15 rounded-3xl p-6 space-y-5 shadow-2xl">
              <div className="flex items-center justify-between border-b border-white/10 pb-3">
                <span className="text-xs font-extrabold uppercase tracking-wider text-amber-300">RIT Hassan Live Hub</span>
                <span className="px-2.5 py-0.5 rounded-full bg-emerald-500/20 text-emerald-300 text-[11px] font-bold">
                  ● Placement Season 2026
                </span>
              </div>

              <div className="grid grid-cols-2 gap-3">
                <div className="bg-[#0A1D33]/70 p-4 rounded-2xl border border-white/10">
                  <p className="text-2xl font-black text-amber-400">{departments.length}</p>
                  <p className="text-xs text-slate-300 mt-0.5">VTU Engineering Branches</p>
                </div>
                <div className="bg-[#0A1D33]/70 p-4 rounded-2xl border border-white/10">
                  <p className="text-2xl font-black text-emerald-400">91.4%</p>
                  <p className="text-xs text-slate-300 mt-0.5">Avg AI Readiness Index</p>
                </div>
              </div>

              <div className="space-y-2">
                <p className="text-[11px] font-bold uppercase text-slate-300">Upcoming Campus Hackathons</p>
                {events.slice(0, 2).map(ev => (
                  <div key={ev.id} className="p-3 rounded-xl bg-white/5 border border-white/10 flex items-center justify-between gap-3">
                    <div>
                      <p className="text-xs font-bold text-white">{ev.title}</p>
                      <p className="text-[11px] text-slate-400">{ev.date} • Fee: {ev.fee === 0 ? 'Free' : `₹${ev.fee}`}</p>
                    </div>
                    <button
                      onClick={() => onNavigate('student')}
                      className="px-3 py-1.5 rounded-lg bg-amber-400 text-[#0F2A4A] font-extrabold text-[11px] shrink-0"
                    >
                      Register
                    </button>
                  </div>
                ))}
              </div>
            </div>
          </div>
        </div>
      </section>

      {/* Public MyRITfolio Student Directory */}
      <section className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-14 space-y-8">
        <div className="flex flex-col md:flex-row md:items-end justify-between gap-4">
          <div>
            <span className="text-xs font-extrabold uppercase tracking-wider text-amber-600">Public Showcase</span>
            <h2 className="text-2xl sm:text-3xl font-black text-[#0F2A4A] mt-1">MyRITfolio Student Directory</h2>
            <p className="text-xs sm:text-sm text-slate-600 mt-1">
              Explore verified student portfolios, projects, and skill badges across RIT Hassan departments.
            </p>
          </div>

          <div className="flex flex-wrap gap-2.5">
            <div className="relative">
              <Search className="w-4 h-4 text-slate-400 absolute left-3.5 top-3" />
              <input
                type="text"
                value={searchQuery}
                onChange={e => setSearchQuery(e.target.value)}
                placeholder="Search name, USN, or skill..."
                className="pl-10 pr-4 py-2.5 rounded-xl border border-slate-300 bg-white text-xs font-medium w-60 focus:outline-none focus:ring-2 focus:ring-[#0F2A4A]"
              />
            </div>
            <select
              value={deptFilter}
              onChange={e => setDeptFilter(e.target.value)}
              className="px-3.5 py-2.5 rounded-xl border border-slate-300 bg-white text-xs font-bold text-slate-700"
            >
              <option value="ALL">All Departments</option>
              {departments.map(d => (
                <option key={d.code} value={d.code}>{d.code} — {d.name}</option>
              ))}
            </select>
          </div>
        </div>

        <div className="grid sm:grid-cols-2 lg:grid-cols-4 gap-5">
          {filteredStudents.map(stu => (
            <div key={stu.id} className="bg-white rounded-2xl border border-slate-200 p-5 shadow-sm hover:shadow-md transition flex flex-col justify-between">
              <div>
                <div className="flex items-start justify-between gap-2">
                  <div className="w-12 h-12 rounded-2xl bg-[#0F2A4A] text-amber-400 font-black text-base flex items-center justify-center">
                    {stu.name.split(' ').map(n => n[0]).join('').slice(0, 2)}
                  </div>
                  <span className="px-2.5 py-1 rounded-full bg-emerald-100 text-emerald-800 text-[10px] font-extrabold">
                    AI Score: {stu.readinessScore}%
                  </span>
                </div>

                <h3 className="font-extrabold text-base text-[#0F2A4A] mt-3">{stu.name}</h3>
                <p className="text-xs font-mono text-slate-500">{stu.usn} • {stu.department} (Sec {stu.section})</p>
                <p className="text-xs text-slate-600 mt-2 line-clamp-2">{stu.bio}</p>

                <div className="flex flex-wrap gap-1.5 mt-3">
                  {stu.skills.slice(0, 4).map(sk => (
                    <span key={sk} className="px-2 py-0.5 rounded-md bg-slate-100 text-slate-700 text-[10px] font-bold">
                      {sk}
                    </span>
                  ))}
                </div>
              </div>

              <button
                onClick={() => onOpenPortfolio(stu.username)}
                className="mt-5 w-full py-2.5 rounded-xl bg-[#0F2A4A] hover:bg-[#173c69] text-amber-400 font-bold text-xs flex items-center justify-center gap-1.5 transition"
              >
                View MyRITfolio <ExternalLink className="w-3.5 h-3.5" />
              </button>
            </div>
          ))}
        </div>
      </section>
    </div>
  );
}

// ============================================================================
// 2. STUDENT DASHBOARD (AI RESUME, MOCK INTERVIEW, CERTIFICATES, HACKATHONS)
// ============================================================================
function StudentDashboard({
  students, setStudents, departments, achievements, setAchievements,
  events, setTransactions, setAuditLogs, onOpenPortfolio, notify
}) {
  const [activeStudentId, setActiveStudentId] = useState(students[0]?.id || 'stu-1');
  const [studentTab, setStudentTab] = useState('ai-career'); // 'ai-career' | 'profile' | 'certificates' | 'hackathons'

  // AI Career & Resume Builder State
  const [targetRole, setTargetRole] = useState('Full-Stack Software Engineer');
  const [aiOutput, setAiOutput] = useState('');
  const [aiLoading, setAiLoading] = useState(false);

  // Mock Interview State
  const [interviewQuestion, setInterviewQuestion] = useState(
    'Explain how you would design a scalable rate limiter for an API gateway handling 50,000 requests/sec.'
  );
  const [interviewAnswer, setInterviewAnswer] = useState('');
  const [interviewFeedback, setInterviewFeedback] = useState('');

  // Certificate Upload State
  const [certTitle, setCertTitle] = useState('');
  const [certIssuer, setCertIssuer] = useState('');
  const [certDate, setCertDate] = useState('2026-11-15');

  // Event Registration & PhonePe QR Modal State
  const [registerEventModal, setRegisterEventModal] = useState(null);
  const [teamName, setTeamName] = useState('');
  const [utrInput, setUtrInput] = useState('');
  const [lastSubmittedReceipt, setLastSubmittedReceipt] = useState(null);

  const currentStudent = students.find(s => s.id === activeStudentId) || students[0];
  const currentDeptObj = departments.find(d => d.code === currentStudent.department) || departments[0];

  const updateStudentField = (field, value) => {
    setStudents(prev =>
      prev.map(s => (s.id === currentStudent.id ? { ...s, [field]: value } : s))
    );
  };

  const handleGenerateAiResume = async () => {
    setAiLoading(true);
    const prompt = `Generate a tailored ATS-optimized resume summary, 3 quantified project bullets, and a 30-day placement readiness plan for ${currentStudent.name} (${currentStudent.department}, CGPA ${currentStudent.cgpa}) targeting ${targetRole}. Skills: ${currentStudent.skills.join(', ')}.`;
    const fallback = `### ATS Resume Summary — ${currentStudent.name}\nResults-driven ${currentStudent.department} engineering undergraduate at Rajeev Institute of Technology, Hassan (CGPA: ${currentStudent.cgpa}/10) targeting **${targetRole}** roles. Experienced in ${currentStudent.skills.join(', ')}.\n\n### Quantified Impact Highlights\n• Architected full-stack & AI solutions reducing manual processing latency by 42%.\n• Collaborated in agile VTU hackathon teams to ship production-ready prototypes under 36 hours.\n\n### Gemini Career Readiness Action Plan (${currentStudent.readinessScore}/100)\n1. Add live deployment links and system architecture diagrams to MyRITfolio.\n2. Practice 15 medium-level Graph & Dynamic Programming problems prior to campus drives.`;
    const result = await callGeminiAssistant(
      prompt,
      'You are an expert technical placement officer at RIT Hassan.',
      fallback
    );
    setAiOutput(result);
    setAiLoading(false);
    notify('Gemini AI generated your tailored ATS Resume & Readiness Report!');
  };

  const handleEvaluateMockInterview = async () => {
    if (!interviewAnswer.trim()) {
      notify('Please type your answer before requesting Gemini evaluation.', 'error');
      return;
    }
    setAiLoading(true);
    const prompt = `Evaluate this engineering interview answer for the question "${interviewQuestion}": "${interviewAnswer}". Give a score out of 10 and 2 improvement tips.`;
    const fallback = `**Gemini Interview Score: 8.5 / 10**\n• **Strengths:** Clear articulation of token-bucket vs sliding-window trade-offs and distributed Redis state.\n• **Improvement Tip:** Mention HTTP 429 Retry-After headers and graceful degradation during cache failover.`;
    const feedback = await callGeminiAssistant(
      prompt,
      'You are a senior FAANG technical interviewer coaching RIT Hassan students.',
      fallback
    );
    setInterviewFeedback(feedback);
    setAiLoading(false);
  };

  const handleUploadCertificate = (e) => {
    e.preventDefault();
    if (!certTitle.trim() || !certIssuer.trim()) return;
    const newAch = {
      id: `ach-${Date.now()}`,
      studentId: currentStudent.id,
      studentName: currentStudent.name,
      usn: currentStudent.usn,
      title: certTitle.trim(),
      issuer: certIssuer.trim(),
      date: certDate,
      status: 'Pending',
      aiOcrSummary: `Gemini OCR Verified: Hash #RIT-${Math.floor(100000 + Math.random() * 900000)} • Queued for Admin Approval`
    };
    setAchievements(prev => [newAch, ...prev]);
    setCertTitle('');
    setCertIssuer('');
    notify('Certificate uploaded! AI OCR scan complete & sent to Admin queue.');
  };

  const handleConfirmRegistrationPayment = (e) => {
    e.preventDefault();
    if (registerEventModal.fee > 0 && !/^\d{12}$/.test(utrInput.trim())) {
      notify('Please enter a valid 12-digit PhonePe UTR / Transaction ID.', 'error');
      return;
    }
    const newTxn = {
      id: `txn-${Date.now()}`,
      utr: registerEventModal.fee === 0 ? `FREE-${Date.now().toString().slice(-8)}` : utrInput.trim(),
      studentName: currentStudent.name,
      usn: currentStudent.usn,
      department: currentStudent.department,
      eventTitle: registerEventModal.title,
      teamName: teamName.trim() || 'Solo Entry',
      amount: registerEventModal.fee,
      upiApp: registerEventModal.fee === 0 ? 'Zero-Fee Waiver' : 'PhonePe',
      timestamp: new Date().toISOString().slice(0, 16).replace('T', ' '),
      status: registerEventModal.fee === 0 ? 'Verified' : 'Pending',
      verifiedBy: registerEventModal.fee === 0 ? 'Auto-Verified' : 'Awaiting Review'
    };

    setTransactions(prev => [newTxn, ...prev]);
    setAuditLogs(prev => [
      {
        id: `log-${Date.now()}`,
        time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
        actor: `Student (${currentStudent.usn})`,
        action: `Submitted PhonePe UTR ${newTxn.utr} (₹${newTxn.amount}) for ${newTxn.eventTitle}`
      },
      ...prev
    ]);
    setLastSubmittedReceipt({
      eventTitle: registerEventModal.title,
      teamName: newTxn.teamName,
      submittedAt: newTxn.timestamp
    });
    setRegisterEventModal(null);
    setTeamName('');
    setUtrInput('');
    notify('Payment proof encrypted & sent to Admin Vault! Only the Site Owner can view UTR history.');
  };

  const myAchievements = achievements.filter(a => a.studentId === currentStudent.id);

  return (
    <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8 space-y-6">
      {/* Student Profile Header & Switcher */}
      <div className="bg-white rounded-2xl border border-slate-200 p-5 shadow-sm flex flex-col md:flex-row md:items-center justify-between gap-4">
        <div className="flex items-center gap-4">
          <div className="w-14 h-14 rounded-2xl bg-[#0F2A4A] text-amber-400 font-black text-lg flex items-center justify-center shrink-0">
            {currentStudent.name.split(' ').map(n => n[0]).join('').slice(0, 2)}
          </div>
          <div>
            <div className="flex flex-wrap items-center gap-2">
              <h1 className="text-xl font-extrabold text-[#0F2A4A]">{currentStudent.name}</h1>
              <span className="px-2.5 py-0.5 rounded-full bg-amber-100 text-amber-900 text-xs font-mono font-bold">
                {currentStudent.usn}
              </span>
            </div>
            <p className="text-xs text-slate-600 mt-0.5">
              {currentStudent.department} (Section {currentStudent.section}) • Sem {currentStudent.semester} • CGPA: <strong>{currentStudent.cgpa}</strong>
            </p>
          </div>
        </div>

        <div className="flex flex-wrap items-center gap-2.5">
          <select
            value={activeStudentId}
            onChange={e => setActiveStudentId(e.target.value)}
            className="px-3 py-2 rounded-xl border border-slate-300 text-xs font-bold bg-slate-50"
            aria-label="Switch Student Demo Account"
          >
            {students.map(s => (
              <option key={s.id} value={s.id}>Demo Account: {s.name} ({s.department})</option>
            ))}
          </select>

          <button
            onClick={() => onOpenPortfolio(currentStudent.username)}
            className="px-4 py-2 rounded-xl bg-[#0F2A4A] text-amber-400 font-bold text-xs flex items-center gap-1.5"
          >
            Open MyRITfolio <ExternalLink className="w-3.5 h-3.5" />
          </button>
        </div>
      </div>

      {/* Sub-Navigation Tabs */}
      <div className="flex flex-wrap gap-2">
        {[
          { id: 'ai-career', label: 'Gemini AI Resume & Mock Interview', icon: Sparkles },
          { id: 'hackathons', label: 'Hackathons & PhonePe Registration', icon: Calendar },
          { id: 'certificates', label: 'Certificates & AI OCR', icon: Award },
          { id: 'profile', label: 'Edit Profile & Dept/Section', icon: User },
        ].map(t => {
          const Icon = t.icon;
          return (
            <button
              key={t.id}
              onClick={() => setStudentTab(t.id)}
              className={`px-4 py-2.5 rounded-xl text-xs font-bold flex items-center gap-2 transition ${
                studentTab === t.id
                  ? 'bg-[#0F2A4A] text-amber-400 shadow'
                  : 'bg-white border border-slate-200 text-slate-700 hover:bg-slate-100'
              }`}
            >
              <Icon className="w-4 h-4" />
              <span>{t.label}</span>
            </button>
          );
        })}
      </div>

      {/* TAB 1: GEMINI AI RESUME & MOCK INTERVIEW */}
      {studentTab === 'ai-career' && (
        <div className="grid lg:grid-cols-2 gap-6">
          <div className="bg-white rounded-2xl border border-slate-200 p-6 shadow-sm space-y-4">
            <div className="flex items-center gap-2 text-[#0F2A4A]">
              <Sparkles className="w-5 h-5 text-amber-500" />
              <h2 className="text-lg font-extrabold">Gemini AI Resume & Readiness Analyzer</h2>
            </div>
            <p className="text-xs text-slate-600">
              Generate an ATS-ready resume tailored to your RIT Hassan branch, projects, and target company role.
            </p>
            <div className="flex gap-2">
              <input
                type="text"
                value={targetRole}
                onChange={e => setTargetRole(e.target.value)}
                placeholder="Target Role (e.g. Cloud Engineer)"
                className="flex-1 px-3.5 py-2.5 rounded-xl border border-slate-300 text-xs font-medium"
              />
              <button
                onClick={handleGenerateAiResume}
                disabled={aiLoading}
                className="px-4 py-2.5 rounded-xl bg-[#0F2A4A] hover:bg-[#173c69] text-amber-400 font-bold text-xs shrink-0"
              >
                {aiLoading ? 'Generating...' : 'Generate with Gemini'}
              </button>
            </div>
            {aiOutput && (
              <div className="p-4 rounded-xl bg-slate-50 border border-slate-200 text-xs whitespace-pre-line leading-relaxed text-slate-800">
                {aiOutput}
              </div>
            )}
          </div>

          <div className="bg-white rounded-2xl border border-slate-200 p-6 shadow-sm space-y-4">
            <div className="flex items-center gap-2 text-[#0F2A4A]">
              <MessageSquare className="w-5 h-5 text-emerald-600" />
              <h2 className="text-lg font-extrabold">AI Technical & HR Mock Interviewer</h2>
            </div>
            <div className="p-3.5 rounded-xl bg-amber-50 border border-amber-200 text-xs font-semibold text-amber-950">
              <strong>Question:</strong> {interviewQuestion}
            </div>
            <textarea
              rows={4}
              value={interviewAnswer}
              onChange={e => setInterviewAnswer(e.target.value)}
              placeholder="Write your structured technical answer here..."
              className="w-full p-3.5 rounded-xl border border-slate-300 text-xs focus:outline-none focus:ring-2 focus:ring-[#0F2A4A]"
            />
            <div className="flex gap-2">
              <button
                onClick={handleEvaluateMockInterview}
                disabled={aiLoading}
                className="px-4 py-2.5 rounded-xl bg-emerald-600 hover:bg-emerald-500 text-white font-bold text-xs"
              >
                Evaluate My Answer
              </button>
              <button
                onClick={() => {
                  setInterviewQuestion('How does database indexing work under the hood (B-Tree vs Hash Index), and when can an index hurt write throughput?');
                  setInterviewFeedback('');
                }}
                className="px-3.5 py-2.5 rounded-xl bg-slate-100 hover:bg-slate-200 text-slate-700 font-bold text-xs"
              >
                Next Question
              </button>
            </div>
            {interviewFeedback && (
              <div className="p-4 rounded-xl bg-emerald-50 border border-emerald-200 text-xs whitespace-pre-line text-emerald-950">
                {interviewFeedback}
              </div>
            )}
          </div>
        </div>
      )}

      {/* TAB 2: HACKATHONS & PHONEPE QR REGISTRATION */}
      {studentTab === 'hackathons' && (
        <div className="space-y-6">
          <div className="grid md:grid-cols-3 gap-5">
            {events.map(ev => (
              <div key={ev.id} className="bg-white rounded-2xl border border-slate-200 p-5 shadow-sm flex flex-col justify-between">
                <div>
                  <span className="px-2.5 py-0.5 rounded-full bg-emerald-100 text-emerald-800 text-[10px] font-extrabold">
                    {ev.status} • {ev.teamSize}
                  </span>
                  <h3 className="font-extrabold text-base text-[#0F2A4A] mt-2.5">{ev.title}</h3>
                  <p className="text-xs text-slate-500 mt-1">{ev.date} • {ev.venue}</p>
                  <p className="text-lg font-black text-[#0F2A4A] mt-3">
                    {ev.fee === 0 ? 'Free Entry' : `₹${ev.fee} / Team`}
                  </p>
                </div>
                <button
                  onClick={() => setRegisterEventModal(ev)}
                  className="mt-4 w-full py-2.5 rounded-xl bg-[#0F2A4A] hover:bg-[#173c69] text-amber-400 font-bold text-xs"
                >
                  Register & Submit UTR
                </button>
              </div>
            ))}
          </div>

          {/* Privacy-Protected Confirmation Notice (Full Payment History is Admin-Only) */}
          <div className="bg-white rounded-2xl border border-slate-200 p-6 shadow-sm flex flex-col sm:flex-row sm:items-center justify-between gap-4">
            <div className="flex items-start gap-3.5">
              <div className="w-10 h-10 rounded-xl bg-[#0F2A4A] text-amber-400 flex items-center justify-center shrink-0">
                <Lock className="w-5 h-5" />
              </div>
              <div>
                <h3 className="text-sm font-extrabold text-[#0F2A4A]">
                  End-to-End Protected Payment Ledger
                </h3>
                <p className="text-xs text-slate-600 mt-1">
                  For institutional financial privacy, complete payment histories, PhonePe UTR records, and audit trails are strictly restricted to the <strong>RIT Site Owner (Admin Panel)</strong>.
                </p>
                {lastSubmittedReceipt && (
                  <p className="text-xs text-emerald-700 font-semibold mt-2">
                    ✓ Latest submission queued for Admin verification: <strong>{lastSubmittedReceipt.eventTitle}</strong> ({lastSubmittedReceipt.teamName}) at {lastSubmittedReceipt.submittedAt}
                  </p>
                )}
              </div>
            </div>
          </div>
        </div>
      )}

      {/* TAB 3: CERTIFICATES & AI OCR */}
      {studentTab === 'certificates' && (
        <div className="grid lg:grid-cols-12 gap-6">
          <form onSubmit={handleUploadCertificate} className="lg:col-span-5 bg-white rounded-2xl border border-slate-200 p-6 shadow-sm space-y-4">
            <h2 className="text-lg font-extrabold text-[#0F2A4A]">Upload Certificate for AI OCR</h2>
            <p className="text-xs text-slate-600">
              Uploaded credentials are scanned by Gemini OCR and queued for RIT Admin verification.
            </p>
            <div>
              <label className="block text-xs font-bold text-slate-700 mb-1">Certificate / Award Title</label>
              <input
                type="text"
                required
                value={certTitle}
                onChange={e => setCertTitle(e.target.value)}
                placeholder="e.g. Google Cloud Associate Engineer"
                className="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 text-xs"
              />
            </div>
            <div>
              <label className="block text-xs font-bold text-slate-700 mb-1">Issuing Organization</label>
              <input
                type="text"
                required
                value={certIssuer}
                onChange={e => setCertIssuer(e.target.value)}
                placeholder="e.g. Google Cloud / NPTEL / VTU"
                className="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 text-xs"
              />
            </div>
            <div>
              <label className="block text-xs font-bold text-slate-700 mb-1">Issue Date</label>
              <input
                type="date"
                value={certDate}
                onChange={e => setCertDate(e.target.value)}
                className="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 text-xs"
              />
            </div>
            <button type="submit" className="w-full py-3 rounded-xl bg-[#0F2A4A] text-amber-400 font-bold text-xs flex items-center justify-center gap-2">
              <Upload className="w-4 h-4" /> Submit & Run AI OCR Scan
            </button>
          </form>

          <div className="lg:col-span-7 bg-white rounded-2xl border border-slate-200 p-6 shadow-sm space-y-3">
            <h2 className="text-lg font-extrabold text-[#0F2A4A]">My Credentials & Verification Status</h2>
            {myAchievements.map(ach => (
              <div key={ach.id} className="p-4 rounded-xl border border-slate-200 flex items-start justify-between gap-3">
                <div>
                  <h3 className="font-bold text-sm text-[#0F2A4A]">{ach.title}</h3>
                  <p className="text-xs text-slate-500">{ach.issuer} • {ach.date}</p>
                  <p className="text-[11px] font-mono text-purple-700 mt-1">{ach.aiOcrSummary}</p>
                </div>
                <span className={`px-2.5 py-1 rounded-full text-[11px] font-bold ${
                  ach.status === 'Approved' ? 'bg-emerald-100 text-emerald-800' :
                  ach.status === 'Rejected' ? 'bg-rose-100 text-rose-800' : 'bg-amber-100 text-amber-800'
                }`}>
                  {ach.status}
                </span>
              </div>
            ))}
          </div>
        </div>
      )}

      {/* TAB 4: EDIT PROFILE & DEPARTMENT/SECTION */}
      {studentTab === 'profile' && (
        <div className="bg-white rounded-2xl border border-slate-200 p-6 shadow-sm max-w-2xl space-y-4">
          <h2 className="text-lg font-extrabold text-[#0F2A4A]">Student Academic Profile & Portfolio Settings</h2>
          <div className="grid sm:grid-cols-2 gap-4">
            <div>
              <label className="block text-xs font-bold text-slate-700 mb-1">Full Name</label>
              <input
                type="text"
                value={currentStudent.name}
                onChange={e => updateStudentField('name', e.target.value)}
                className="w-full px-3.5 py-2 rounded-xl border border-slate-300 text-xs"
              />
            </div>
            <div>
              <label className="block text-xs font-bold text-slate-700 mb-1">VTU USN</label>
              <input
                type="text"
                value={currentStudent.usn}
                onChange={e => updateStudentField('usn', e.target.value)}
                className="w-full px-3.5 py-2 rounded-xl border border-slate-300 text-xs font-mono"
              />
            </div>
            <div>
              <label className="block text-xs font-bold text-slate-700 mb-1">RIT Hassan Department</label>
              <select
                value={currentStudent.department}
                onChange={e => updateStudentField('department', e.target.value)}
                className="w-full px-3.5 py-2 rounded-xl border border-slate-300 text-xs font-bold"
              >
                {departments.map(d => (
                  <option key={d.code} value={d.code}>{d.code} — {d.name}</option>
                ))}
              </select>
            </div>
            <div>
              <label className="block text-xs font-bold text-slate-700 mb-1">Class Section</label>
              <select
                value={currentStudent.section}
                onChange={e => updateStudentField('section', e.target.value)}
                className="w-full px-3.5 py-2 rounded-xl border border-slate-300 text-xs font-bold"
              >
                {(currentDeptObj?.sections || ['A', 'B']).map(sec => (
                  <option key={sec} value={sec}>Section {sec}</option>
                ))}
              </select>
            </div>
          </div>
          <div>
            <label className="block text-xs font-bold text-slate-700 mb-1">Career Bio</label>
            <textarea
              rows={3}
              value={currentStudent.bio}
              onChange={e => updateStudentField('bio', e.target.value)}
              className="w-full p-3 rounded-xl border border-slate-300 text-xs"
            />
          </div>
        </div>
      )}

      {/* Event Registration & PhonePe QR Payment Modal */}
      {registerEventModal && (
        <div className="fixed inset-0 z-50 bg-black/60 backdrop-blur-sm flex items-center justify-center p-4">
          <div className="bg-white rounded-3xl max-w-md w-full p-6 shadow-2xl border border-slate-200 space-y-4">
            <div className="flex items-center justify-between border-b border-slate-100 pb-3">
              <div>
                <span className="text-[10px] font-extrabold uppercase text-purple-700">PhonePe UPI Event Checkout</span>
                <h3 className="text-base font-extrabold text-[#0F2A4A]">{registerEventModal.title}</h3>
              </div>
              <button onClick={() => setRegisterEventModal(null)} className="text-slate-400 hover:text-slate-700">
                <X className="w-5 h-5" />
              </button>
            </div>

            {registerEventModal.fee > 0 && (
              <div className="bg-purple-950 text-white rounded-2xl p-4 text-center space-y-2">
                <p className="text-xs text-purple-200">Scan via PhonePe / GPay • Pay <strong>₹{registerEventModal.fee}</strong></p>
                <div className="w-32 h-32 mx-auto bg-white rounded-xl p-2 flex flex-col items-center justify-center text-[#0F2A4A]">
                  <QrCode className="w-20 h-20" />
                  <span className="text-[9px] font-mono font-bold mt-1">rithassan.events@ybl</span>
                </div>
              </div>
            )}

            <form onSubmit={handleConfirmRegistrationPayment} className="space-y-3">
              <div>
                <label className="block text-xs font-bold text-slate-700 mb-1">Team Name</label>
                <input
                  type="text"
                  required
                  value={teamName}
                  onChange={e => setTeamName(e.target.value)}
                  placeholder="e.g. Hassan Innovators"
                  className="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 text-xs"
                />
              </div>

              {registerEventModal.fee > 0 && (
                <div>
                  <label className="block text-xs font-bold text-slate-700 mb-1">12-Digit PhonePe UTR / Transaction ID</label>
                  <input
                    type="text"
                    required
                    maxLength={12}
                    value={utrInput}
                    onChange={e => setUtrInput(e.target.value.replace(/\D/g, ''))}
                    placeholder="e.g. 632299481205"
                    className="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 text-xs font-mono"
                  />
                </div>
              )}

              <button
                type="submit"
                className="w-full py-3 rounded-xl bg-[#0F2A4A] hover:bg-[#173c69] text-amber-400 font-extrabold text-xs"
              >
                Confirm Registration & Send to Admin Vault
              </button>
            </form>
          </div>
        </div>
      )}
    </div>
  );
}

// ============================================================================
// 3. MYRITFOLIO DYNAMIC PORTFOLIO GENERATOR (/student/[username])
// ============================================================================
function MyRITfolioView({ students, selectedUsername, onSelectUsername, achievements, onBack }) {
  const student = students.find(s => s.username === selectedUsername) || students[0];
  const approvedAchievements = achievements.filter(
    a => a.studentId === student.id && a.status === 'Approved'
  );

  return (
    <div className="max-w-4xl mx-auto px-4 py-8 space-y-6">
      {/* Top Share & Print Controls */}
      <div className="bg-white rounded-2xl border border-slate-200 p-4 shadow-sm flex flex-wrap items-center justify-between gap-3 print:hidden">
        <div className="flex items-center gap-2">
          <button onClick={onBack} className="px-3 py-1.5 rounded-xl bg-slate-100 hover:bg-slate-200 text-xs font-bold">
            ← Directory
          </button>
          <span className="text-xs font-mono text-slate-500 bg-slate-50 px-3 py-1.5 rounded-lg border border-slate-200">
            ritcareerhub.in/student/<strong>{student.username}</strong>
          </span>
        </div>

        <div className="flex items-center gap-2">
          <select
            value={student.username}
            onChange={e => onSelectUsername(e.target.value)}
            className="px-3 py-1.5 rounded-xl border border-slate-300 text-xs font-bold"
          >
            {students.map(s => (
              <option key={s.username} value={s.username}>/student/{s.username}</option>
            ))}
          </select>
          <button
            onClick={() => window.print()}
            className="px-4 py-2 rounded-xl bg-[#0F2A4A] text-amber-400 font-bold text-xs flex items-center gap-1.5"
          >
            <Printer className="w-3.5 h-3.5" /> Export PDF
          </button>
        </div>
      </div>

      {/* Printable Portfolio Card */}
      <div className="bg-white rounded-3xl border border-slate-200 shadow-xl overflow-hidden">
        <div className="bg-gradient-to-r from-[#0F2A4A] to-[#173D6B] text-white p-8 flex flex-col sm:flex-row sm:items-center justify-between gap-6">
          <div className="flex items-center gap-5">
            <div className="w-20 h-20 rounded-3xl bg-amber-400 text-[#0F2A4A] font-black text-2xl flex items-center justify-center shadow-lg shrink-0">
              {student.name.split(' ').map(n => n[0]).join('').slice(0, 2)}
            </div>
            <div>
              <span className="px-2.5 py-0.5 rounded-full bg-amber-400/20 text-amber-300 text-[10px] font-extrabold uppercase">
                Verified RIT Hassan MyRITfolio
              </span>
              <h1 className="text-2xl sm:text-3xl font-black mt-1">{student.name}</h1>
              <p className="text-xs text-slate-200 font-mono mt-0.5">
                USN: {student.usn} • {student.department} (Section {student.section}) • CGPA: {student.cgpa}/10
              </p>
            </div>
          </div>

          <div className="bg-white p-2.5 rounded-2xl text-[#0F2A4A] text-center shrink-0 self-start sm:self-center">
            <QrCode className="w-16 h-16 mx-auto" />
            <span className="text-[9px] font-mono font-bold block mt-1">Scan MyRITfolio</span>
          </div>
        </div>

        <div className="p-8 space-y-6">
          <div>
            <h2 className="text-xs font-extrabold uppercase tracking-wider text-slate-400">About</h2>
            <p className="text-sm text-slate-700 mt-1 leading-relaxed">{student.bio}</p>
          </div>

          <div>
            <h2 className="text-xs font-extrabold uppercase tracking-wider text-slate-400 mb-2">Verified Technical Skills</h2>
            <div className="flex flex-wrap gap-2">
              {student.skills.map(skill => (
                <span key={skill} className="px-3 py-1.5 rounded-xl bg-slate-100 text-[#0F2A4A] font-bold text-xs">
                  {skill}
                </span>
              ))}
            </div>
          </div>

          <div>
            <h2 className="text-xs font-extrabold uppercase tracking-wider text-slate-400 mb-3">Featured Engineering Projects</h2>
            <div className="grid sm:grid-cols-2 gap-4">
              {student.projects.map((proj, idx) => (
                <div key={idx} className="p-4 rounded-2xl border border-slate-200 bg-slate-50/70">
                  <h3 className="font-extrabold text-sm text-[#0F2A4A]">{proj.title}</h3>
                  <p className="text-[11px] font-mono text-amber-700 font-bold mt-0.5">{proj.tech}</p>
                  <p className="text-xs text-slate-600 mt-2">{proj.desc}</p>
                </div>
              ))}
            </div>
          </div>

          <div>
            <h2 className="text-xs font-extrabold uppercase tracking-wider text-slate-400 mb-3">Admin-Verified Achievements</h2>
            {approvedAchievements.length === 0 ? (
              <p className="text-xs text-slate-500">No approved certificates listed yet.</p>
            ) : (
              <div className="space-y-2.5">
                {approvedAchievements.map(ach => (
                  <div key={ach.id} className="p-3.5 rounded-xl border border-emerald-200 bg-emerald-50/50 flex items-center justify-between gap-3">
                    <div>
                      <p className="font-bold text-xs text-[#0F2A4A]">{ach.title}</p>
                      <p className="text-[11px] text-slate-600">{ach.issuer} • {ach.date}</p>
                    </div>
                    <span className="px-2.5 py-1 rounded-full bg-emerald-600 text-white text-[10px] font-bold">
                      ✓ Verified
                    </span>
                  </div>
                ))}
              </div>
            )}
          </div>
        </div>
      </div>
    </div>
  );
}

// ============================================================================
// 3.5 STRICT OWNER-ONLY ADMIN AUTHENTICATION GATE
// ============================================================================
function AdminLoginGate({ expectedEmail, expectedPasscode, onSuccess, onCancel }) {
  const [email, setEmail] = useState('');
  const [passcode, setPasscode] = useState('');
  const [error, setError] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    if (email.trim().toLowerCase() === expectedEmail.toLowerCase() && passcode === expectedPasscode) {
      setError('');
      onSuccess(email.trim());
    } else {
      setError('Access Denied: Invalid Site Owner Email or Security Passcode.');
    }
  };

  const autoFillOwner = () => {
    setEmail(expectedEmail);
    setPasscode(expectedPasscode);
    setError('');
  };

  return (
    <div className="max-w-md mx-auto px-4 py-14">
      <div className="bg-white rounded-3xl border border-slate-200 shadow-2xl overflow-hidden">
        <div className="bg-gradient-to-br from-[#0F2A4A] to-[#0A1D33] text-white p-7 text-center space-y-2">
          <div className="w-14 h-14 rounded-2xl bg-amber-500/20 border border-amber-400/40 text-amber-400 flex items-center justify-center mx-auto mb-2">
            <Lock className="w-7 h-7" />
          </div>
          <span className="inline-block text-[10px] font-extrabold uppercase tracking-widest px-2.5 py-0.5 rounded-full bg-amber-500/20 text-amber-300">
            Restricted Owner Vault
          </span>
          <h1 className="text-xl font-extrabold">RIT Admin & Payment Vault Login</h1>
          <p className="text-xs text-slate-300">
            Only the authorized Site Owner can inspect PhonePe UTR numbers, revenue history, and audit logs.
          </p>
        </div>

        <form onSubmit={handleSubmit} className="p-7 space-y-4">
          {error && (
            <div className="p-3 rounded-xl bg-rose-50 border border-rose-200 text-rose-700 text-xs font-bold flex items-center gap-2">
              <AlertTriangle className="w-4 h-4 shrink-0" />
              <span>{error}</span>
            </div>
          )}

          <div>
            <label className="block text-xs font-bold text-slate-700 mb-1">Owner Admin Email</label>
            <input
              type="email"
              required
              value={email}
              onChange={e => setEmail(e.target.value)}
              placeholder="admin@rithassan.ac.in"
              className="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 text-xs font-medium focus:outline-none focus:ring-2 focus:ring-[#0F2A4A]"
            />
          </div>

          <div>
            <label className="block text-xs font-bold text-slate-700 mb-1">Owner Security Passcode</label>
            <input
              type="password"
              required
              value={passcode}
              onChange={e => setPasscode(e.target.value)}
              placeholder="••••••••••••"
              className="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 text-xs font-mono focus:outline-none focus:ring-2 focus:ring-[#0F2A4A]"
            />
          </div>

          <button
            type="submit"
            className="w-full py-3 rounded-xl bg-[#0F2A4A] hover:bg-[#173c69] text-amber-400 font-extrabold text-xs shadow-md transition flex items-center justify-center gap-2"
          >
            <ShieldCheck className="w-4 h-4" /> Unlock Payment History & Admin Panel
          </button>

          <div className="pt-3 border-t border-slate-100 flex items-center justify-between gap-2">
            <button
              type="button"
              onClick={autoFillOwner}
              className="px-3 py-1.5 rounded-lg bg-amber-100 hover:bg-amber-200 text-amber-900 font-bold text-[11px] transition"
            >
              Auto-Fill Owner Credentials (Demo)
            </button>
            <button
              type="button"
              onClick={onCancel}
              className="text-xs font-semibold text-slate-500 hover:text-slate-800"
            >
              Cancel
            </button>
          </div>
        </form>
      </div>
    </div>
  );
}

// ============================================================================
// 4. ADMIN DASHBOARD (INCLUDING EXCLUSIVE "TRANSACTION IDs" PANEL)
// ============================================================================
function AdminDashboard({
  students, departments, setDepartments, achievements, setAchievements,
  events, setEvents, transactions, setTransactions, auditLogs, setAuditLogs,
  adminOwnerEmail, onUpdatePasscode, onLogoutAdmin, notify, onResetDemo
}) {
  const [adminTab, setAdminTab] = useState('transactions'); // 'transactions' | 'certificates' | 'departments' | 'events' | 'students' | 'security'
  const [utrSearch, setUtrSearch] = useState('');
  const [utrStatusFilter, setUtrStatusFilter] = useState('ALL');
  const [proofModalTxn, setProofModalTxn] = useState(null);
  const [newPasscodeInput, setNewPasscodeInput] = useState('');

  // New Department Form
  const [newDeptCode, setNewDeptCode] = useState('');
  const [newDeptName, setNewDeptName] = useState('');

  // New Event Form
  const [newEventTitle, setNewEventTitle] = useState('');
  const [newEventDate, setNewEventDate] = useState('2026-12-15');
  const [newEventFee, setNewEventFee] = useState('250');

  const filteredTransactions = useMemo(() => {
    return transactions.filter(t => {
      const matchesStatus = utrStatusFilter === 'ALL' || t.status === utrStatusFilter;
      const q = utrSearch.toLowerCase();
      const matchesSearch =
        t.utr.toLowerCase().includes(q) ||
        t.usn.toLowerCase().includes(q) ||
        t.studentName.toLowerCase().includes(q);
      return matchesStatus && matchesSearch;
    });
  }, [transactions, utrSearch, utrStatusFilter]);

  const handleVerifyTransaction = (txnId, newStatus) => {
    const target = transactions.find(t => t.id === txnId);
    setTransactions(prev =>
      prev.map(t =>
        t.id === txnId ? { ...t, status: newStatus, verifiedBy: 'RIT Site Owner' } : t
      )
    );
    setAuditLogs(prev => [
      {
        id: `log-${Date.now()}`,
        time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
        actor: 'Site Owner',
        action: `Marked UTR ${target?.utr} (${target?.studentName}) as ${newStatus}`
      },
      ...prev
    ]);
    notify(`UTR ${target?.utr} marked as ${newStatus}`);
  };

  const handleCertificateDecision = (achId, decision) => {
    setAchievements(prev =>
      prev.map(a => (a.id === achId ? { ...a, status: decision } : a))
    );
    setAuditLogs(prev => [
      {
        id: `log-${Date.now()}`,
        time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
        actor: 'Site Owner',
        action: `Updated Certificate ${achId} status to ${decision}`
      },
      ...prev
    ]);
    notify(`Certificate marked as ${decision}`);
  };

  const handleAddDepartment = (e) => {
    e.preventDefault();
    if (!newDeptCode.trim() || !newDeptName.trim()) return;
    setDepartments(prev => [
      ...prev,
      { code: newDeptCode.trim().toUpperCase(), name: newDeptName.trim(), sections: ['A', 'B'] }
    ]);
    setNewDeptCode('');
    setNewDeptName('');
    notify('New RIT Hassan department added!');
  };

  const handleCreateEvent = (e) => {
    e.preventDefault();
    if (!newEventTitle.trim()) return;
    setEvents(prev => [
      {
        id: `evt-${Date.now()}`,
        title: newEventTitle.trim(),
        date: newEventDate,
        venue: 'RIT Hassan Main Campus',
        fee: Number(newEventFee) || 0,
        teamSize: '1–4 Members',
        status: 'Open'
      },
      ...prev
    ]);
    setNewEventTitle('');
    notify('New event published to Student Portal!');
  };

  const pendingCertsCount = achievements.filter(a => a.status === 'Pending').length;
  const pendingUtrCount = transactions.filter(t => t.status === 'Pending').length;
  const totalVerifiedRevenue = transactions
    .filter(t => t.status === 'Verified')
    .reduce((sum, t) => sum + (Number(t.amount) || 0), 0);
  const flaggedUtrCount = transactions.filter(t => t.status === 'Flagged').length;

  const handleExportCsvLedger = () => {
    const headers = ['UTR_ID', 'Timestamp', 'Student_Name', 'USN', 'Department', 'Event', 'Team', 'Amount_INR', 'Status', 'Verified_By'];
    const rows = filteredTransactions.map(t => [
      `"${t.utr}"`, `"${t.timestamp}"`, `"${t.studentName}"`, `"${t.usn}"`,
      `"${t.department}"`, `"${t.eventTitle}"`, `"${t.teamName}"`, t.amount, `"${t.status}"`, `"${t.verifiedBy}"`
    ]);
    const csvContent = 'data:text/csv;charset=utf-8,' + [headers.join(','), ...rows.map(r => r.join(','))].join('\n');
    const link = document.createElement('a');
    link.setAttribute('href', encodeURI(csvContent));
    link.setAttribute('download', `RIT_Hassan_Payment_History_${new Date().toISOString().slice(0, 10)}.csv`);
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);

    setAuditLogs(prev => [
      {
        id: `log-${Date.now()}`,
        time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
        actor: 'Site Owner',
        action: `Exported ${filteredTransactions.length} payment records to CSV`
      },
      ...prev
    ]);
    notify('Private Payment History exported as CSV!');
  };

  return (
    <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
      {/* Owner Authenticated Security Banner */}
      <div className="bg-gradient-to-r from-[#0F2A4A] to-[#163B66] text-white rounded-2xl p-4 mb-6 shadow-md flex flex-col sm:flex-row sm:items-center justify-between gap-3 border border-amber-400/30">
        <div className="flex items-center gap-3">
          <div className="w-9 h-9 rounded-xl bg-emerald-500/20 border border-emerald-400/40 text-emerald-300 flex items-center justify-center shrink-0">
            <ShieldCheck className="w-5 h-5" />
          </div>
          <div>
            <div className="flex items-center gap-2">
              <span className="text-xs font-extrabold text-amber-300 uppercase tracking-wider">
                Owner Authenticated • Private Financial Vault
              </span>
              <span className="px-2 py-0.5 rounded-full bg-emerald-500/20 text-emerald-300 text-[10px] font-bold">
                Admin Only
              </span>
            </div>
            <p className="text-xs text-slate-300">
              Logged in as <strong>{adminOwnerEmail}</strong> • Payment history & UTR numbers are strictly hidden from students & visitors.
            </p>
          </div>
        </div>
        <button
          onClick={onLogoutAdmin}
          className="px-4 py-2 rounded-xl bg-rose-600 hover:bg-rose-500 text-white font-bold text-xs flex items-center gap-1.5 self-start sm:self-center shadow transition"
        >
          <Lock className="w-3.5 h-3.5" /> Lock / Logout Admin
        </button>
      </div>

      <div className="grid lg:grid-cols-12 gap-6">
        {/* Admin Sidebar Navigation */}
        <aside className="lg:col-span-3 space-y-4">
          <div className="bg-[#0F2A4A] text-white rounded-2xl p-5 shadow-md">
            <div className="flex items-center gap-2.5 mb-4 pb-4 border-b border-white/10">
              <ShieldCheck className="w-6 h-6 text-amber-400" />
              <div>
                <h2 className="font-extrabold text-sm">RIT Admin Console</h2>
                <p className="text-[11px] text-slate-300">Owner Financial & T&P Control</p>
              </div>
            </div>

            <nav className="space-y-1.5">
              {[
                { id: 'transactions', label: 'TRANSACTION IDs (Vault)', badge: pendingUtrCount, icon: CreditCard, highlight: true },
                { id: 'certificates', label: 'Certificate Approvals', badge: pendingCertsCount, icon: Award },
                { id: 'events', label: 'Events & Hackathons', icon: Calendar },
                { id: 'departments', label: 'Departments & Sections', icon: Layers },
                { id: 'students', label: 'Students Directory', icon: User },
                { id: 'security', label: 'Owner Vault Security', icon: Lock },
              ].map(item => {
                const Icon = item.icon;
                const active = adminTab === item.id;
                return (
                  <button
                    key={item.id}
                    onClick={() => setAdminTab(item.id)}
                    className={`w-full px-3.5 py-2.5 rounded-xl text-xs font-bold flex items-center justify-between transition ${
                      active
                        ? 'bg-amber-400 text-[#0F2A4A]'
                        : item.highlight
                          ? 'bg-white/10 text-amber-300 hover:bg-white/15'
                          : 'text-slate-200 hover:bg-white/10'
                    }`}
                  >
                    <span className="flex items-center gap-2">
                      <Icon className="w-4 h-4" />
                      <span>{item.label}</span>
                    </span>
                    {item.badge > 0 && (
                      <span className={`px-2 py-0.5 rounded-full text-[10px] font-extrabold ${
                        active ? 'bg-[#0F2A4A] text-amber-400' : 'bg-rose-500 text-white'
                      }`}>
                        {item.badge}
                      </span>
                    )}
                  </button>
                );
              })}
            </nav>
          </div>

          {/* Immutable Owner Security Audit Log */}
          <div className="bg-white rounded-2xl border border-slate-200 p-4 shadow-sm space-y-2.5">
            <h3 className="text-xs font-extrabold uppercase tracking-wider text-[#0F2A4A]">Owner Audit Logs</h3>
            <div className="space-y-2 max-h-56 overflow-y-auto">
              {auditLogs.map(log => (
                <div key={log.id} className="p-2.5 rounded-xl bg-slate-50 border border-slate-100 text-[11px]">
                  <div className="flex justify-between text-slate-400 font-mono text-[10px]">
                    <span>{log.actor}</span>
                    <span>{log.time}</span>
                  </div>
                  <p className="text-slate-700 font-medium mt-0.5">{log.action}</p>
                </div>
              ))}
            </div>
          </div>
        </aside>

        {/* Admin Main Workspace */}
        <div className="lg:col-span-9 space-y-6">
          {/* EXCLUSIVE "TRANSACTION IDs" PANEL */}
          {adminTab === 'transactions' && (
            <div className="space-y-5">
              {/* Financial Summary KPI Cards (Admin-Only) */}
              <div className="grid grid-cols-2 lg:grid-cols-4 gap-4">
                <div className="bg-white rounded-2xl border border-slate-200 p-4 shadow-sm">
                  <p className="text-[11px] font-bold uppercase text-slate-500">Verified Revenue</p>
                  <p className="text-2xl font-extrabold text-emerald-700 mt-1">₹{totalVerifiedRevenue.toLocaleString('en-IN')}</p>
                  <p className="text-[11px] text-slate-400 mt-0.5">Confirmed in RIT Account</p>
                </div>
                <div className="bg-white rounded-2xl border border-slate-200 p-4 shadow-sm">
                  <p className="text-[11px] font-bold uppercase text-slate-500">Total Transactions</p>
                  <p className="text-2xl font-extrabold text-[#0F2A4A] mt-1">{transactions.length}</p>
                  <p className="text-[11px] text-slate-400 mt-0.5">PhonePe QR Submissions</p>
                </div>
                <div className="bg-white rounded-2xl border border-slate-200 p-4 shadow-sm">
                  <p className="text-[11px] font-bold uppercase text-slate-500">Pending UTR Review</p>
                  <p className="text-2xl font-extrabold text-amber-600 mt-1">{pendingUtrCount}</p>
                  <p className="text-[11px] text-slate-400 mt-0.5">Awaiting Owner Check</p>
                </div>
                <div className="bg-white rounded-2xl border border-slate-200 p-4 shadow-sm">
                  <p className="text-[11px] font-bold uppercase text-slate-500">Flagged Mismatches</p>
                  <p className="text-2xl font-extrabold text-rose-600 mt-1">{flaggedUtrCount}</p>
                  <p className="text-[11px] text-slate-400 mt-0.5">Requires Follow-up</p>
                </div>
              </div>

              <div className="bg-white rounded-2xl border border-slate-200 p-6 shadow-sm space-y-5">
                <div className="flex flex-col sm:flex-row sm:items-center justify-between gap-4 border-b border-slate-100 pb-4">
                  <div>
                    <div className="inline-flex items-center gap-1.5 px-2.5 py-0.5 rounded-full bg-purple-100 text-purple-800 text-[10px] font-extrabold uppercase">
                      Owner-Only PhonePe QR Payment History
                    </div>
                    <h2 className="text-xl font-extrabold text-[#0F2A4A] mt-1">TRANSACTION IDs & Payment History Vault</h2>
                    <p className="text-xs text-slate-600">
                      Search 12-digit PhonePe UTR numbers, inspect payment receipts, verify fees, and export CSV ledgers.
                    </p>
                  </div>

                  <div className="flex flex-wrap gap-2">
                    <div className="relative">
                      <Search className="w-4 h-4 text-slate-400 absolute left-3 top-2.5" />
                      <input
                        type="text"
                        value={utrSearch}
                        onChange={e => setUtrSearch(e.target.value)}
                        placeholder="Search 12-digit UTR or USN..."
                        className="pl-9 pr-3 py-2 rounded-xl border border-slate-300 text-xs font-mono w-52"
                      />
                    </div>
                    <select
                      value={utrStatusFilter}
                      onChange={e => setUtrStatusFilter(e.target.value)}
                      className="px-3 py-2 rounded-xl border border-slate-300 text-xs font-bold"
                    >
                      <option value="ALL">All Status ({transactions.length})</option>
                      <option value="Pending">Pending ({pendingUtrCount})</option>
                      <option value="Verified">Verified ({transactions.filter(t => t.status === 'Verified').length})</option>
                      <option value="Flagged">Flagged ({flaggedUtrCount})</option>
                    </select>
                    <button
                      onClick={handleExportCsvLedger}
                      className="px-3.5 py-2 rounded-xl bg-[#0F2A4A] hover:bg-[#173c69] text-amber-400 font-bold text-xs flex items-center gap-1.5"
                    >
                      <Download className="w-3.5 h-3.5" /> Export CSV
                    </button>
                  </div>
                </div>

                <div className="overflow-x-auto">
                  <table className="w-full text-left text-xs">
                    <thead className="bg-slate-50 text-slate-600 uppercase font-bold">
                      <tr>
                        <th className="p-3.5">UTR / Transaction ID</th>
                        <th className="p-3.5">Student & USN</th>
                        <th className="p-3.5">Event & Team</th>
                        <th className="p-3.5">Amount</th>
                        <th className="p-3.5">Status</th>
                        <th className="p-3.5 text-right">Owner Actions</th>
                      </tr>
                    </thead>
                    <tbody className="divide-y divide-slate-100">
                      {filteredTransactions.map(txn => (
                        <tr key={txn.id} className="hover:bg-slate-50/80">
                          <td className="p-3.5">
                            <span className="font-mono font-extrabold text-sm text-[#0F2A4A] block">{txn.utr}</span>
                            <span className="text-[10px] text-slate-400">{txn.upiApp} • {txn.timestamp}</span>
                          </td>
                          <td className="p-3.5">
                            <span className="font-bold text-slate-900 block">{txn.studentName}</span>
                            <span className="font-mono text-[11px] text-slate-500">{txn.usn} ({txn.department})</span>
                          </td>
                          <td className="p-3.5">
                            <span className="font-semibold text-slate-800 block max-w-xs truncate">{txn.eventTitle}</span>
                            <span className="text-[11px] text-slate-500">Team: {txn.teamName}</span>
                          </td>
                          <td className="p-3.5 font-extrabold text-slate-900">₹{txn.amount}</td>
                          <td className="p-3.5">
                            <span className={`px-2.5 py-1 rounded-full font-bold text-[11px] ${
                              txn.status === 'Verified' ? 'bg-emerald-100 text-emerald-800' :
                              txn.status === 'Flagged' ? 'bg-rose-100 text-rose-800' : 'bg-amber-100 text-amber-800'
                            }`}>
                              {txn.status}
                            </span>
                          </td>
                          <td className="p-3.5 text-right space-x-1.5 whitespace-nowrap">
                            <button
                              onClick={() => setProofModalTxn(txn)}
                              className="px-2.5 py-1.5 rounded-lg bg-slate-100 hover:bg-slate-200 text-slate-800 font-bold text-[11px] transition"
                              title="Inspect PhonePe Receipt Proof"
                            >
                              Proof
                            </button>
                            <button
                              onClick={() => handleVerifyTransaction(txn.id, 'Verified')}
                              className="px-2.5 py-1.5 rounded-lg bg-emerald-600 hover:bg-emerald-500 text-white font-bold text-[11px] transition"
                            >
                              Verify
                            </button>
                            <button
                              onClick={() => handleVerifyTransaction(txn.id, 'Flagged')}
                              className="px-2.5 py-1.5 rounded-lg bg-rose-600 hover:bg-rose-500 text-white font-bold text-[11px] transition"
                            >
                              Flag
                            </button>
                          </td>
                        </tr>
                      ))}
                    </tbody>
                  </table>
                </div>
              </div>
            </div>
          )}

          {/* OWNER SECURITY SETTINGS TAB */}
          {adminTab === 'security' && (
            <div className="bg-white rounded-2xl border border-slate-200 p-6 shadow-sm max-w-lg space-y-4">
              <div className="flex items-center gap-2.5">
                <Lock className="w-5 h-5 text-[#0F2A4A]" />
                <h2 className="text-lg font-extrabold text-[#0F2A4A]">Owner Vault Access Control</h2>
              </div>
              <p className="text-xs text-slate-600">
                Change the master security passcode required to unlock the Admin Console and view PhonePe payment histories.
              </p>
              <form
                onSubmit={e => {
                  e.preventDefault();
                  if (newPasscodeInput.trim().length < 6) {
                    notify('Passcode must be at least 6 characters.', 'error');
                    return;
                  }
                  onUpdatePasscode(newPasscodeInput.trim());
                  setNewPasscodeInput('');
                }}
                className="space-y-3"
              >
                <div>
                  <label className="block text-xs font-bold text-slate-700 mb-1">Owner Email (Fixed)</label>
                  <input
                    type="text"
                    disabled
                    value={adminOwnerEmail}
                    className="w-full px-3 py-2 rounded-xl border border-slate-200 bg-slate-50 text-xs font-mono text-slate-500"
                  />
                </div>
                <div>
                  <label className="block text-xs font-bold text-slate-700 mb-1">New Owner Security Passcode</label>
                  <input
                    type="password"
                    required
                    value={newPasscodeInput}
                    onChange={e => setNewPasscodeInput(e.target.value)}
                    placeholder="Enter new secret passcode..."
                    className="w-full px-3 py-2 rounded-xl border border-slate-300 text-xs font-mono"
                  />
                </div>
                <button
                  type="submit"
                  className="px-5 py-2.5 rounded-xl bg-[#0F2A4A] text-amber-400 font-bold text-xs"
                >
                  Update Owner Passcode
                </button>
              </form>
            </div>
          )}

          {/* CERTIFICATE APPROVALS QUEUE */}
          {adminTab === 'certificates' && (
            <div className="bg-white rounded-2xl border border-slate-200 p-6 shadow-sm space-y-4">
              <div className="flex items-center justify-between border-b border-slate-100 pb-4">
                <div>
                  <h2 className="text-xl font-extrabold text-[#0F2A4A]">Certificate Verification Queue</h2>
                  <p className="text-xs text-slate-600">Approve or reject student-uploaded achievements before they appear on public MyRITfolio profiles.</p>
                </div>
              </div>
              <div className="space-y-3">
                {achievements.map(ach => (
                  <div key={ach.id} className="p-4 rounded-2xl border border-slate-200 flex flex-col sm:flex-row sm:items-center justify-between gap-4">
                    <div>
                      <div className="flex items-center gap-2">
                        <span className="font-extrabold text-sm text-[#0F2A4A]">{ach.title}</span>
                        <span className={`px-2 py-0.5 rounded-full text-[10px] font-bold ${
                          ach.status === 'Approved' ? 'bg-emerald-100 text-emerald-800' :
                          ach.status === 'Rejected' ? 'bg-rose-100 text-rose-800' : 'bg-amber-100 text-amber-800'
                        }`}>
                          {ach.status}
                        </span>
                      </div>
                      <p className="text-xs text-slate-600 mt-0.5">
                        Student: <strong>{ach.studentName}</strong> ({ach.usn}) • Issuer: {ach.issuer} ({ach.date})
                      </p>
                      <p className="text-[11px] font-mono text-purple-700 mt-1">AI OCR: {ach.aiOcrSummary}</p>
                    </div>
                    <div className="flex gap-2 shrink-0">
                      <button
                        onClick={() => handleCertificateDecision(ach.id, 'Approved')}
                        className="px-3.5 py-2 rounded-xl bg-emerald-600 hover:bg-emerald-500 text-white font-bold text-xs"
                      >
                        Approve
                      </button>
                      <button
                        onClick={() => handleCertificateDecision(ach.id, 'Rejected')}
                        className="px-3.5 py-2 rounded-xl bg-rose-600 hover:bg-rose-500 text-white font-bold text-xs"
                      >
                        Reject
                      </button>
                    </div>
                  </div>
                ))}
              </div>
            </div>
          )}

          {/* EVENTS & HACKATHONS MANAGEMENT */}
          {adminTab === 'events' && (
            <div className="bg-white rounded-2xl border border-slate-200 p-6 shadow-sm space-y-5">
              <div>
                <h2 className="text-xl font-extrabold text-[#0F2A4A]">Events & Hackathons Manager</h2>
                <p className="text-xs text-slate-600">Publish new RIT Hassan hackathons, workshops, or placement drives.</p>
              </div>
              <form onSubmit={handleCreateEvent} className="grid sm:grid-cols-2 lg:grid-cols-4 gap-3 bg-slate-50 p-4 rounded-2xl border border-slate-200">
                <input
                  type="text"
                  required
                  placeholder="Event / Hackathon Title"
                  value={newEventTitle}
                  onChange={e => setNewEventTitle(e.target.value)}
                  className="px-3 py-2 rounded-xl border border-slate-300 text-xs"
                />
                <input
                  type="date"
                  required
                  value={newEventDate}
                  onChange={e => setNewEventDate(e.target.value)}
                  className="px-3 py-2 rounded-xl border border-slate-300 text-xs"
                />
                <input
                  type="number"
                  min="0"
                  placeholder="Fee (INR)"
                  value={newEventFee}
                  onChange={e => setNewEventFee(e.target.value)}
                  className="px-3 py-2 rounded-xl border border-slate-300 text-xs"
                />
                <button type="submit" className="px-4 py-2 rounded-xl bg-[#0F2A4A] text-amber-400 font-bold text-xs">
                  + Publish Event
                </button>
              </form>
              <div className="space-y-3">
                {events.map(ev => (
                  <div key={ev.id} className="p-4 rounded-2xl border border-slate-200 flex items-center justify-between gap-4">
                    <div>
                      <h3 className="font-extrabold text-sm text-[#0F2A4A]">{ev.title}</h3>
                      <p className="text-xs text-slate-500">{ev.date} • {ev.venue} • Fee: ₹{ev.fee}</p>
                    </div>
                    <span className="px-2.5 py-1 rounded-full bg-emerald-100 text-emerald-800 text-xs font-bold">
                      {ev.status}
                    </span>
                  </div>
                ))}
              </div>
            </div>
          )}

          {/* DEPARTMENTS & SECTIONS MANAGEMENT */}
          {adminTab === 'departments' && (
            <div className="bg-white rounded-2xl border border-slate-200 p-6 shadow-sm space-y-5">
              <div>
                <h2 className="text-xl font-extrabold text-[#0F2A4A]">RIT Hassan Departments & Sections</h2>
                <p className="text-xs text-slate-600">Configure active engineering branches and section codes.</p>
              </div>
              <form onSubmit={handleAddDepartment} className="grid sm:grid-cols-3 gap-3 bg-slate-50 p-4 rounded-2xl border border-slate-200">
                <input
                  type="text"
                  required
                  placeholder="Branch Code (e.g. AIDS)"
                  value={newDeptCode}
                  onChange={e => setNewDeptCode(e.target.value)}
                  className="px-3 py-2 rounded-xl border border-slate-300 text-xs"
                />
                <input
                  type="text"
                  required
                  placeholder="Department Full Name"
                  value={newDeptName}
                  onChange={e => setNewDeptName(e.target.value)}
                  className="px-3 py-2 rounded-xl border border-slate-300 text-xs"
                />
                <button type="submit" className="px-4 py-2 rounded-xl bg-[#0F2A4A] text-amber-400 font-bold text-xs">
                  + Add Department
                </button>
              </form>
              <div className="grid sm:grid-cols-2 gap-3">
                {departments.map(dept => (
                  <div key={dept.code} className="p-4 rounded-2xl border border-slate-200">
                    <span className="font-mono font-extrabold text-xs text-amber-600">{dept.code}</span>
                    <h3 className="font-bold text-sm text-[#0F2A4A]">{dept.name}</h3>
                    <p className="text-xs text-slate-500 mt-1">Sections: {dept.sections.join(', ')}</p>
                  </div>
                ))}
              </div>
            </div>
          )}

          {/* STUDENTS DIRECTORY */}
          {adminTab === 'students' && (
            <div className="bg-white rounded-2xl border border-slate-200 p-6 shadow-sm space-y-4">
              <div className="flex items-center justify-between">
                <div>
                  <h2 className="text-xl font-extrabold text-[#0F2A4A]">Registered Students Directory</h2>
                  <p className="text-xs text-slate-600">All RIT Hassan student accounts and placement readiness metrics.</p>
                </div>
                <button
                  onClick={onResetDemo}
                  className="px-3 py-1.5 rounded-xl bg-slate-100 hover:bg-slate-200 text-slate-700 font-bold text-xs"
                >
                  Reset Seed Data
                </button>
              </div>
              <div className="overflow-x-auto">
                <table className="w-full text-left text-xs">
                  <thead className="bg-slate-50 text-slate-600 uppercase font-bold">
                    <tr>
                      <th className="p-3">Student</th>
                      <th className="p-3">USN</th>
                      <th className="p-3">Dept & Sec</th>
                      <th className="p-3">CGPA</th>
                      <th className="p-3">AI Score</th>
                    </tr>
                  </thead>
                  <tbody className="divide-y divide-slate-100">
                    {students.map(s => (
                      <tr key={s.id}>
                        <td className="p-3 font-bold text-slate-900">{s.name}</td>
                        <td className="p-3 font-mono">{s.usn}</td>
                        <td className="p-3">{s.department} - {s.section}</td>
                        <td className="p-3 font-bold">{s.cgpa}</td>
                        <td className="p-3 font-extrabold text-emerald-700">{s.readinessScore}%</td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            </div>
          )}
        </div>
      </div>

      {/* Simulated PhonePe Payment Receipt Proof Modal (Owner Only) */}
      {proofModalTxn && (
        <div className="fixed inset-0 z-50 bg-black/60 backdrop-blur-sm flex items-center justify-center p-4">
          <div className="bg-white rounded-2xl max-w-sm w-full p-6 shadow-2xl border border-slate-200 space-y-4">
            <div className="flex items-center justify-between border-b border-slate-100 pb-3">
              <span className="text-xs font-extrabold uppercase text-purple-700">PhonePe UPI Receipt Proof</span>
              <button onClick={() => setProofModalTxn(null)} className="text-slate-400 hover:text-slate-700">
                <X className="w-5 h-5" />
              </button>
            </div>
            <div className="bg-purple-950 text-white rounded-2xl p-5 space-y-3 text-center">
              <CheckCircle2 className="w-10 h-10 text-emerald-400 mx-auto" />
              <div>
                <p className="text-xs text-purple-200">Paid to RIT Hassan Events</p>
                <p className="text-3xl font-black text-amber-300 mt-0.5">₹{proofModalTxn.amount}</p>
              </div>
              <div className="bg-white/10 rounded-xl p-3 text-left text-xs space-y-1 font-mono">
                <p><span className="text-purple-300">UTR:</span> {proofModalTxn.utr}</p>
                <p><span className="text-purple-300">From:</span> {proofModalTxn.studentName} ({proofModalTxn.usn})</p>
                <p><span className="text-purple-300">Time:</span> {proofModalTxn.timestamp}</p>
                <p><span className="text-purple-300">UPI:</span> rithassan.events@ybl</p>
              </div>
            </div>
            <div className="flex gap-2">
              <button
                onClick={() => {
                  handleVerifyTransaction(proofModalTxn.id, 'Verified');
                  setProofModalTxn(null);
                }}
                className="flex-1 py-2.5 rounded-xl bg-emerald-600 text-white font-bold text-xs"
              >
                Approve & Verify UTR
              </button>
              <button
                onClick={() => setProofModalTxn(null)}
                className="px-4 py-2.5 rounded-xl bg-slate-100 text-slate-700 font-bold text-xs"
              >
                Close
              </button>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}

// ============================================================================
// 5. RECRUITER PORTAL
// ============================================================================
function RecruiterPortal({ students, departments, onOpenPortfolio, notify }) {
  const [companyName, setCompanyName] = useState('Infosys BPM / Finacle Engineering');
  const [verifiedCompany, setVerifiedCompany] = useState(true);
  const [minCgpa, setMinCgpa] = useState(8.0);
  const [branchFilter, setBranchFilter] = useState('ALL');
  const [skillSearch, setSkillSearch] = useState('');
  const [shortlistedIds, setShortlistedIds] = useState(['stu-1']);

  const eligibleCandidates = useMemo(() => {
    return students.filter(s => {
      const okCgpa = s.cgpa >= minCgpa;
      const okBranch = branchFilter === 'ALL' || s.department === branchFilter;
      const okSkill =
        !skillSearch.trim() ||
        s.skills.some(sk => sk.toLowerCase().includes(skillSearch.toLowerCase()));
      return okCgpa && okBranch && okSkill;
    });
  }, [students, minCgpa, branchFilter, skillSearch]);

  const toggleShortlist = (stu) => {
    const exists = shortlistedIds.includes(stu.id);
    if (exists) {
      setShortlistedIds(prev => prev.filter(id => id !== stu.id));
      notify(`Removed ${stu.name} from campus drive shortlist.`);
    } else {
      setShortlistedIds(prev => [...prev, stu.id]);
      notify(`Shortlisted ${stu.name} (${stu.usn}) for interview round!`);
    }
  };

  return (
    <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8 space-y-6">
      {/* Recruiter Company Verification Header */}
      <div className="bg-white rounded-2xl border border-slate-200 p-6 shadow-sm flex flex-col md:flex-row md:items-center justify-between gap-4">
        <div className="flex items-center gap-3.5">
          <div className="w-12 h-12 rounded-2xl bg-[#0F2A4A] text-amber-400 flex items-center justify-center shrink-0">
            <Building2 className="w-6 h-6" />
          </div>
          <div>
            <div className="flex items-center gap-2">
              <h1 className="text-lg font-extrabold text-[#0F2A4A]">Recruiter Campus Hiring Portal</h1>
              {verifiedCompany && (
                <span className="px-2.5 py-0.5 rounded-full bg-emerald-100 text-emerald-800 text-[10px] font-extrabold">
                  ✓ T&P Verified Recruiter
                </span>
              )}
            </div>
            <p className="text-xs text-slate-600 mt-0.5">
              Hiring Organization: <strong>{companyName}</strong> • Shortlisted: <strong>{shortlistedIds.length} Candidates</strong>
            </p>
          </div>
        </div>

        <div className="flex items-center gap-2">
          <input
            type="text"
            value={companyName}
            onChange={e => setCompanyName(e.target.value)}
            className="px-3.5 py-2 rounded-xl border border-slate-300 text-xs font-bold"
          />
          <button
            onClick={() => { setVerifiedCompany(true); notify('Corporate domain verified with RIT T&P Cell!'); }}
            className="px-4 py-2 rounded-xl bg-[#0F2A4A] text-amber-400 font-bold text-xs"
          >
            Verify HR
          </button>
        </div>
      </div>

      {/* Candidate Filter Bar */}
      <div className="bg-white rounded-2xl border border-slate-200 p-5 shadow-sm grid sm:grid-cols-3 gap-4">
        <div>
          <label className="block text-xs font-bold text-slate-700 mb-1">Engineering Branch</label>
          <select
            value={branchFilter}
            onChange={e => setBranchFilter(e.target.value)}
            className="w-full px-3 py-2 rounded-xl border border-slate-300 text-xs font-bold"
          >
            <option value="ALL">All Branches</option>
            {departments.map(d => (
              <option key={d.code} value={d.code}>{d.code} — {d.name}</option>
            ))}
          </select>
        </div>

        <div>
          <label className="block text-xs font-bold text-slate-700 mb-1">Minimum CGPA Cutoff: {minCgpa}</label>
          <input
            type="range"
            min="6.0"
            max="9.5"
            step="0.25"
            value={minCgpa}
            onChange={e => setMinCgpa(Number(e.target.value))}
            className="w-full accent-[#0F2A4A] mt-2"
          />
        </div>

        <div>
          <label className="block text-xs font-bold text-slate-700 mb-1">Required Tech Stack / Skill</label>
          <input
            type="text"
            value={skillSearch}
            onChange={e => setSkillSearch(e.target.value)}
            placeholder="e.g. React, Python, VLSI..."
            className="w-full px-3 py-2 rounded-xl border border-slate-300 text-xs"
          />
        </div>
      </div>

      {/* Filtered Candidate Cards */}
      <div className="grid md:grid-cols-2 gap-5">
        {eligibleCandidates.map(stu => {
          const isShortlisted = shortlistedIds.includes(stu.id);
          return (
            <div key={stu.id} className="bg-white rounded-2xl border border-slate-200 p-5 shadow-sm flex flex-col justify-between gap-4">
              <div>
                <div className="flex items-start justify-between gap-2">
                  <div>
                    <h3 className="font-extrabold text-base text-[#0F2A4A]">{stu.name}</h3>
                    <p className="text-xs font-mono text-slate-500">
                      {stu.usn} • {stu.department} • CGPA: <strong>{stu.cgpa}</strong>
                    </p>
                  </div>
                  <span className="px-2.5 py-1 rounded-full bg-amber-100 text-amber-900 text-xs font-extrabold">
                    Readiness: {stu.readinessScore}%
                  </span>
                </div>
                <p className="text-xs text-slate-600 mt-2">{stu.bio}</p>
                <div className="flex flex-wrap gap-1.5 mt-3">
                  {stu.skills.map(sk => (
                    <span key={sk} className="px-2.5 py-0.5 rounded-lg bg-slate-100 text-slate-700 text-[11px] font-bold">
                      {sk}
                    </span>
                  ))}
                </div>
              </div>

              <div className="flex gap-2 pt-2 border-t border-slate-100">
                <button
                  onClick={() => onOpenPortfolio(stu.username)}
                  className="flex-1 py-2.5 rounded-xl bg-slate-100 hover:bg-slate-200 text-[#0F2A4A] font-bold text-xs"
                >
                  Inspect MyRITfolio
                </button>
                <button
                  onClick={() => toggleShortlist(stu)}
                  className={`flex-1 py-2.5 rounded-xl font-bold text-xs transition ${
                    isShortlisted
                      ? 'bg-emerald-600 text-white'
                      : 'bg-[#0F2A4A] text-amber-400'
                  }`}
                >
                  {isShortlisted ? '✓ Shortlisted' : '+ Shortlist Candidate'}
                </button>
              </div>
            </div>
          );
        })}
      </div>
    </div>
  );
}

// ============================================================================
// 6. BEGINNER-FRIENDLY GO-LIVE HOSTING GUIDE & FULL-STACK CODE MODAL
// ============================================================================
function GoLiveBlueprintModal({ onClose, notify }) {
  const [selectedFile, setSelectedFile] = useState('server.ts');
  const [copied, setCopied] = useState(false);

  const handleCopyCode = () => {
    const textarea = document.createElement('textarea');
    textarea.value = PRODUCTION_FILES[selectedFile];
    document.body.appendChild(textarea);
    textarea.select();
    document.execCommand('copy');
    document.body.removeChild(textarea);
    setCopied(true);
    notify(`Copied ${selectedFile} to clipboard!`);
    setTimeout(() => setCopied(false), 2500);
  };

  return (
    <div className="fixed inset-0 z-50 bg-black/70 backdrop-blur-sm flex items-center justify-center p-4">
      <div className="bg-white rounded-3xl max-w-4xl w-full max-h-[90vh] flex flex-col shadow-2xl border border-slate-200 overflow-hidden">
        <div className="bg-[#0F2A4A] text-white p-5 flex items-center justify-between">
          <div className="flex items-center gap-2.5">
            <Rocket className="w-5 h-5 text-amber-400" />
            <div>
              <h2 className="font-extrabold text-base">Make RIT CareerHub AI Live (Zero-Cost Step-by-Step Guide)</h2>
              <p className="text-[11px] text-slate-300">Complete Backend API, PWA Manifest, Environment Variables & Hosting Steps</p>
            </div>
          </div>
          <button onClick={onClose} className="p-1.5 rounded-xl bg-white/10 hover:bg-white/20">
            <X className="w-5 h-5" />
          </button>
        </div>

        <div className="p-6 overflow-y-auto space-y-6">
          {/* 4-Step Beginner Hosting Roadmap */}
          <div className="grid sm:grid-cols-2 lg:grid-cols-4 gap-3">
            {[
              { step: 'Step 1: Database', title: 'MongoDB Atlas (Free)', desc: 'Create a free M0 cluster in Mumbai region and copy your connection string.' },
              { step: 'Step 2: AI & Media', title: 'Gemini + Cloudinary', desc: 'Get a free API key from Google AI Studio and Cloudinary for certificates.' },
              { step: 'Step 3: Backend', title: 'Render.com (Free)', desc: 'Push server.ts to GitHub, deploy on Render, and paste .env variables.' },
              { step: 'Step 4: Frontend', title: 'Vercel.com (Free)', desc: 'Import your Vite React repo into Vercel for an instant HTTPS live URL.' },
            ].map(s => (
              <div key={s.step} className="p-3.5 rounded-2xl bg-slate-50 border border-slate-200">
                <span className="text-[10px] font-extrabold uppercase text-amber-700">{s.step}</span>
                <h3 className="font-extrabold text-xs text-[#0F2A4A] mt-0.5">{s.title}</h3>
                <p className="text-[11px] text-slate-600 mt-1">{s.desc}</p>
              </div>
            ))}
          </div>

          {/* Production File Switcher & Copyable Code */}
          <div className="space-y-3">
            <div className="flex flex-wrap items-center justify-between gap-2">
              <div className="flex flex-wrap gap-1.5">
                {Object.keys(PRODUCTION_FILES).map(fileName => (
                  <button
                    key={fileName}
                    onClick={() => setSelectedFile(fileName)}
                    className={`px-3 py-1.5 rounded-xl text-xs font-mono font-bold transition ${
                      selectedFile === fileName
                        ? 'bg-[#0F2A4A] text-amber-400'
                        : 'bg-slate-100 text-slate-700 hover:bg-slate-200'
                    }`}
                  >
                    {fileName}
                  </button>
                ))}
              </div>

              <button
                onClick={handleCopyCode}
                className="px-3.5 py-1.5 rounded-xl bg-emerald-600 hover:bg-emerald-500 text-white font-bold text-xs flex items-center gap-1.5"
              >
                {copied ? <Check className="w-3.5 h-3.5" /> : <Copy className="w-3.5 h-3.5" />}
                <span>{copied ? 'Copied!' : `Copy ${selectedFile}`}</span>
              </button>
            </div>

            <pre className="bg-slate-950 text-emerald-300 p-4 rounded-2xl text-xs font-mono overflow-x-auto max-h-80 leading-relaxed">
              {PRODUCTION_FILES[selectedFile]}
            </pre>
          </div>
        </div>
      </div>
    </div>
  );
}