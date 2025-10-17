import React, { useState, useEffect, useCallback, useMemo } from 'react';
import {
    Heart, X, Save, User, FileText, Download, Share2, MessageSquare, Edit, Loader2, ArrowRightCircle, List, ArrowLeft, Send, Users, ThumbsUp, PlusCircle
} from 'lucide-react';


// --- 0. CONFIGURATION AND CONSTANTS ---


// Palette (Dark Theme)
const COLORS = {
    black: '#0B0B12',
    purple: '#6C2BD9', // Imperial Lilac
    lilac: '#C8A2C8',  // Soft accent
    ink: '#EDEAFB',    // Light text
    card: '#111111',
    border: '#222222',
};


// Common essay prompts grouped by focus
const PROMPT_GROUPS = {
    // Group 1: Academic/Policy Focused (DAAD, Fulbright)
    ACADEMIC: [
        { value: 'academic_fit', label: 'How does your academic background fit this program?' },
        { value: 'future_goals', label: 'Describe your future career goals and how this opportunity helps.' },
        { value: 'policy_impact', label: 'What specific policy impact do you hope to achieve?' },
    ],
    // Group 2: Tech/Innovation Focused (MITACS, OpenAI)
    TECH: [
        { value: 'tech_contribution', label: 'Detail your most relevant technical/research contribution.' },
        { value: 'innovative_challenge', label: 'Tell us about an innovative challenge you overcame in tech.' },
        { value: 'leadership_exp', label: 'Detail your most relevant leadership experience.' },
    ],
    // Group 3: General/Global (Chevening)
    GLOBAL: [
        { value: 'why_good_fit', label: 'Why are you a good fit for this program?' },
        { value: 'why_org', label: 'Why did you choose this specific organization or country?' },
        { value: 'personal_challenge', label: 'Tell us about a significant personal challenge you overcame.' },
    ],
};


// Fixed list of quotes about originality, now including author
const ORIGINALITY_QUOTES = [
    { quote: "The power of a single personality will always be greater than the power of circumstances.", author: "Marcus Aurelius" },
    { quote: "Originality is simply a pair of fresh eyes.", author: "Thomas W. Higginson" },
    { quote: "Be yourself; everyone else is already taken.", author: "Oscar Wilde" },
    { quote: "The secret to creativity is knowing how to hide your sources.", author: "Albert Einstein" },
    { quote: "The chief enemy of creativity is 'good' sense.", author: "Pablo Picasso" },
    { quote: "Originality is the best form of rebellion.", author: "Mike Sacks" },
    { quote: "Don't compromise yourself. You are all you've got.", author: "Janis Joplin" },
    { quote: "To be authentic, we must be willing to be vulnerable.", author: "Brené Brown" },
    { quote: "It takes courage to grow up and become who you really are.", author: "E.E. Cummings" },
    { quote: "The best way to predict the future is to create it.", author: "Peter Drucker" },
];


// Seed Data (Initial Opportunities)
const SEED_OPPORTUNITIES = [
    { id: 1, title: 'DAAD Helmut-Schmidt Scholarship', org: 'DAAD', summary: 'Public policy master’s in Germany.', deadline: '2025-07-31T00:00:00.000Z', logoUrl: 'https://placehold.co/300x150/004B87/FFCC00/png?text=DAAD+SHIELD', promptGroup: 'ACADEMIC' },
    { id: 2, title: 'Chevening Scholarship', org: 'UK FCDO', summary: 'One-year master’s in the UK.', deadline: '2025-11-07T00:00:00.000Z', logoUrl: 'https://placehold.co/300x150/012169/FFFFFF/png?text=CHEVENING+LOGO', promptGroup: 'GLOBAL' },
    { id: 3, title: 'Fulbright Foreign Student', org: 'US State Dept', summary: 'Graduate study in the USA.', deadline: '2025-04-15T00:00:00.000Z', logoUrl: 'https://placehold.co/300x150/B31942/0A3161/png?text=FULBRIGHT+US', promptGroup: 'ACADEMIC' },
    { id: 4, title: 'MITACS Globalink Research Internship', org: 'Mitacs', summary: '12-week research in Canada.', deadline: '2025-10-01T00:00:00.000Z', logoUrl: 'https://placehold.co/300x150/008000/FFFFFF/png?text=MITACS+RESEARCH', promptGroup: 'TECH' },
    { id: 5, title: 'OpenAI Safety Scholars', org: 'OpenAI', summary: 'Scholars program on AI safety.', deadline: '2025-05-30T00:00:00.000Z', logoUrl: 'https://placehold.co/300x150/000000/FFFFFF/png?text=OPENAI+LOGO', promptGroup: 'TECH' }
];


// Seed data for Network Posts
const SEED_POSTS = [
    {
        id: 'post1', authorId: 'user-b', authorName: 'Elena V.', authorPhotoUrl: 'https://placehold.co/100x100/6C2BD9/FFFFFF?text=EV',
        timestamp: new Date(Date.now() - 86400000 * 1).toISOString(),
        content: "Just spent 6 hours refining my personal statement for the Chevening application. It's a marathon, not a sprint! Anyone else applying? Would love to connect and share tips. #Chevening #UKStudy",
        imageUrl: 'https://placehold.co/600x400/111111/EDEAFB?text=Study+Session', likes: 15,
        comments: [ { id: 'c1', authorName: 'Carlos M.', text: 'Great work Elena! I am also applying. The leadership essay is tough.' }, { id: 'c2', authorName: 'User', text: 'You got this! Keep up the great work.' } ]
    },
    {
        id: 'post2', authorId: 'user-c', authorName: 'John D.', authorPhotoUrl: 'https://placehold.co/100x100/D92B2B/FFFFFF?text=JD',
        timestamp: new Date(Date.now() - 86400000 * 2).toISOString(),
        content: "Found this amazing opportunity for a research internship in Canada! MITACS Globalink is perfect for anyone in STEM. Deadline is approaching, don't miss out! Check it out.",
        imageUrl: null, opportunityLink: 'https://www.mitacs.ca/en/programs/globalink/globalink-research-internship', likes: 42, comments: []
    },
];


// Local Storage Fallback Keys
const STORAGE_KEYS = {
    PROFILE: 'nabu:profile',
    OPPORTUNITIES: 'nabu:opportunities',
    SWIPES: 'nabu:swipes',
    DRAFTS: 'nabu:drafts',
    SHARES: 'nabu:shares',
    POSTS: 'nabu:posts',
};


// --- 1. UTILITIES AND MOCKS ---
const generateUUID = () => crypto.randomUUID().slice(0, 12);
const generateSocialId = () => crypto.randomUUID().slice(0, 8);


const debounce = (func, delay) => {
    let timeout;
    return (...args) => {
        clearTimeout(timeout);
        timeout = setTimeout(() => func(...args), delay);
    };
};


const formatDate = (isoString) => {
    if (!isoString) return 'TBA';
    try {
        const date = new Date(isoString);
        return date.toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: 'numeric' });
    } catch { return 'Invalid Date'; }
};


const timeAgo = (isoString) => {
    const date = new Date(isoString);
    const seconds = Math.floor((new Date() - date) / 1000);
    let interval = seconds / 31536000; if (interval > 1) return Math.floor(interval) + "y ago";
    interval = seconds / 2592000; if (interval > 1) return Math.floor(interval) + "mo ago";
    interval = seconds / 86400; if (interval > 1) return Math.floor(interval) + "d ago";
    interval = seconds / 3600; if (interval > 1) return Math.floor(interval) + "h ago";
    interval = seconds / 60; if (interval > 1) return Math.floor(interval) + "m ago";
    return Math.floor(seconds) + "s ago";
};


const mockEssayGenerator = (profile, opportunity, prompt) => {
    const { title, org } = opportunity;
    const { cv, ambitions, stories, name } = profile;
    const storyList = Array.isArray(stories) ? stories.join('\n- ') : (stories || 'No stories provided.');
    const ambitionsList = Array.isArray(ambitions) ? ambitions.join(', ') : (ambitions || 'No interests listed.');
    return ESSAY DRAFT (LOCAL MOCK)\n\nPrompt: "${prompt || 'General fit question'}"\n\nThe ${title} at ${org} is an ideal next step in my pursuit of my core interests in ${ambitionsList}.\n\nMy background in:\n${cv || 'No CV data provided.'}\nhas uniquely prepared me to address the core challenges of this opportunity. The personal narratives, such as:\n- ${storyList}\ndemonstrate my resilience and commitment.\n\nI am confident that I can contribute effectively.\n\n---\nThis is a suggestion. Make it yours.;
};


const callGeminiApi = async (userQuery, systemPrompt) => {
    const apiKey = "";
    const apiUrl = https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey};
    const payload = {
        contents: [{ parts: [{ text: userQuery }] }],
        systemInstruction: { parts: [{ text: systemPrompt }] },
        generationConfig: { temperature: 0.6 }
    };
    for (let i = 0, delay = 1000; i < 3; i++, delay *= 2) {
        try {
            const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
            const result = await response.json();
            const essay = result.candidates?.[0]?.content?.parts?.[0]?.text;
            if (essay) return essay;
            throw new Error(result.error?.message || "API response lacked text content.");
        } catch (error) {
            console.error(Attempt ${i + 1} failed:, error);
            if (i < 2) await new Promise(resolve => setTimeout(resolve, delay));
            else throw new Error("API failed after multiple retries.");
        }
    }
    throw new Error("Failed to call Gemini API.");
};


// --- 2. REUSABLE COMPONENTS ---


const OriginalityLockComponent = ({ onClose }) => {
    const randomQuote = useMemo(() => ORIGINALITY_QUOTES[Math.floor(Math.random() * ORIGINALITY_QUOTES.length)], []);
    return (
        <div className="fixed inset-0 z-[9999] flex items-center justify-center p-8 bg-black/95 text-center transition-opacity duration-300">
            <div className="flex flex-col items-center justify-center p-12 w-full max-w-xl bg-[var(--nabu-card)] border-4 border-[var(--nabu-purple)] rounded-3xl shadow-[0_0_50px_rgba(108,43,217,0.7)] animate-fadeIn space-y-6">
                <Edit className="w-16 h-16 text-[var(--nabu-purple)]"/>
                <h3 className="text-3xl font-bold text-[var(--nabu-purple)] font-serif">Acknowledge Originality</h3>
                <div className="bg-[#1a1a1a] p-6 rounded-xl border border-[#2a2a2a] max-w-sm">
                    <p className="text-xl font-semibold text-[var(--nabu-ink)] mb-4">This is merely a suggestion. Make it yours.</p>
                    <p className="text-sm italic text-[var(--nabu-lilac)] mt-4 font-serif">"{randomQuote.quote}"</p>
                    <p className="text-xs text-gray-500 mt-1 font-semibold">— {randomQuote.author}</p>
                </div>
                <button onClick={onClose} className="py-3 px-8 text-lg font-semibold rounded-xl bg-[var(--nabu-purple)] text-[var(--nabu-ink)] hover:bg-opacity-90 transition-opacity focus:outline-none focus:ring-4 focus:ring-[var(--nabu-purple)]">I Got It</button>
            </div>
        </div>
    );
};


const NabuLogo = ({ navigate }) => (
    <button onClick={() => navigate('/network')} className="flex items-end font-extrabold text-3xl transition-colors text-[var(--nabu-ink)] hover:text-[var(--nabu-purple)] focus:outline-none focus:ring-2 focus:ring-[var(--nabu-purple)] rounded-md p-1">
        NAB<div className="relative leading-none"><span className="text-4xl transition-colors">U</span><svg className="absolute top-5 left-[-3px] w-6 h-3 text-[var(--nabu-purple)] transition-colors" viewBox="0 0 24 10" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M1 6 C6 11, 18 11, 23 6" stroke="currentColor" strokeWidth="2" strokeLinecap="round"/></svg></div>
    </button>
);


const Navbar = ({ navigate, currentPath }) => {
    const navItems = [
        { path: '/network', label: 'Network', icon: Users },
        { path: '/profile/social', label: 'Profile', icon: User },
        { path: '/swipe', label: 'Opportunities', icon: Heart },
        { path: '/saved', label: 'Studio', icon: Save },
    ];
    const showBackButton = currentPath !== '/' && currentPath !== '#/' && currentPath !== '/network';
    const getIsActive = (path) => {
        if (path === '/network' && (currentPath === '/' || currentPath === '#/' || currentPath === '/network')) return true;
        if (path === '/swipe' && currentPath === '/swipe') return true;
        if (path === '/profile/social' && (currentPath.startsWith('#/profile') || currentPath.startsWith('/profile'))) return true;
        if (path === '/saved' && (currentPath === '/saved' || currentPath.startsWith('#/editor'))) return true;
        return false;
    };
    return (
        <nav className="fixed top-0 left-0 right-0 z-50 bg-[var(--nabu-black)] border-b border-[var(--nabu-border)] shadow-lg">
            <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 h-16 flex items-center justify-between">
                <div className="flex items-center space-x-4">
                    {showBackButton && <button onClick={() => window.history.back()} className="p-2 rounded-lg text-[var(--nabu-lilac)] hover:bg-[#1f1f2a] hover:text-[var(--nabu-purple)] focus:outline-none focus:ring-2 focus:ring-[var(--nabu-purple)]" aria-label="Go back"><ArrowLeft className="w-6 h-6" /></button>}
                    <NabuLogo navigate={navigate} />
                </div>
                <div className="flex space-x-1 sm:space-x-2">
                    {navItems.map(({ path, label, icon: Icon }) => {
                        const isActive = getIsActive(path);
                        return (
                            <button key={path} onClick={() => navigate(path)} className={flex items-center space-x-1 p-2 rounded-lg transition-all text-sm font-medium ${isActive ? 'bg-[var(--nabu-purple)] text-[var(--nabu-ink)] shadow-md' : 'text-[var(--nabu-lilac)] hover:bg-[#1f1f2a] hover:text-[var(--nabu-purple)]'} focus:outline-none focus:ring-2 focus:ring-[var(--nabu-purple)]} aria-current={isActive ? 'page' : undefined}>
                                <Icon className="w-5 h-5" /><span className="hidden sm:inline">{label}</span>
                            </button>
                        );
                    })}
                </div>
            </div>
        </nav>
    );
};


const OpportunityCard = ({ opportunity, onClick, isSaved = false, actions }) => {
    const { title, org, summary, deadline, logoUrl } = opportunity;
    const cardContent = (
        <div className="p-5 flex flex-col h-full">
            {logoUrl && <div className="mb-4 rounded-lg overflow-hidden border border-[var(--nabu-border)]"><img src={logoUrl} alt={${org} Logo} className="w-full h-24 object-cover" onError={(e) => { e.target.onerror = null; e.target.src="https://placehold.co/300x150/1a1a1a/C8A2C8?text=Institution+Logo" }}/></div>}
            <div className="flex items-center justify-between mb-2"><span className="px-3 py-1 text-xs font-semibold rounded-full bg-[var(--nabu-purple)] text-[var(--nabu-ink)]">{org}</span><span className="text-xs text-[var(--nabu-lilac)] font-mono"><span className="font-semibold text-[var(--nabu-ink)]">Deadline:</span> {formatDate(deadline)}</span></div>
            <h3 className="text-xl font-bold text-[var(--nabu-ink)] mt-1 mb-3 font-serif">{title}</h3>
            <p className="text-sm text-gray-400 flex-grow mb-4">{summary}</p>
            {actions && <div className="flex justify-between space-x-4 mt-auto pt-3">{actions}</div>}
        </div>
    );
    if (onClick) return <div onClick={onClick} onKeyDown={(e) => { if (e.key === 'Enter' || e.key === ' ') onClick(); }} role="button" tabIndex="0" aria-label={Open editor for ${title}} className={w-full text-left rounded-xl transition-all duration-300 cursor-pointer bg-[var(--nabu-card)] border border-[var(--nabu-border)] hover:border-[var(--nabu-lilac)] shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-[var(--nabu-purple)] ${isSaved ? 'h-full' : 'min-h-[200px]'}}>{cardContent}</div>;
    return <div className="w-full h-full rounded-xl bg-[var(--nabu-card)] border border-[var(--nabu-border)] shadow-xl">{cardContent}</div>;
};


const BigActionButton = ({ icon: Icon, label, onClick, colorClass, hotkey }) => (
    <button onClick={onClick} className={flex flex-col items-center justify-center w-full p-4 rounded-xl text-center font-bold text-lg transition-all duration-200 shadow-xl border border-transparent ${colorClass} focus:outline-none focus:ring-4 focus:ring-offset-2 focus:ring-offset-[var(--nabu-black)]} aria-label={label} data-hotkey={hotkey}>
        <Icon className="w-8 h-8 sm:w-10 sm:h-10 mb-1" /><span className="text-sm sm:text-base">{label}</span>{hotkey && <span className="text-xs opacity-70 mt-1 hidden sm:inline-block">({hotkey})</span>}
    </button>
);


const Toast = ({ message, type, removeToast }) => {
    useEffect(() => { const timer = setTimeout(removeToast, 3000); return () => clearTimeout(timer); }, [removeToast]);
    const colors = { success: 'bg-green-700 border-green-500', error: 'bg-red-700 border-red-500', info: 'bg-blue-700 border-blue-500' };
    return <div className={fixed bottom-4 right-4 p-4 rounded-xl shadow-2xl z-[9999] text-[var(--nabu-ink)] border-l-4 ${colors[type] || colors.info} animate-slideIn}>{message}</div>;
};


const EmptyState = ({ title, message, ctaText, ctaAction }) => (
    <div className="flex flex-col items-center justify-center p-10 text-center rounded-xl border border-dashed border-[var(--nabu-lilac)] bg-[#111111] max-w-lg mx-auto mt-12">
        <ArrowRightCircle className="w-12 h-12 text-[var(--nabu-purple)] mb-4" /><h3 className="text-xl font-semibold text-[var(--nabu-ink)] mb-2">{title}</h3><p className="text-gray-400 mb-6">{message}</p>
        {ctaAction && ctaText && <button onClick={ctaAction} className="px-6 py-2 text-sm font-medium rounded-lg bg-[var(--nabu-purple)] text-[var(--nabu-ink)] hover:bg-opacity-90 transition-opacity focus:outline-none focus:ring-2 focus:ring-[var(--nabu-purple)]">{ctaText}</button>}
    </div>
);


const ShareBanner = ({ link, mode, onClose }) => {
    const handleCopy = () => { navigator.clipboard.writeText(link).then(() => { alert('Collaboration link copied!'); }); };
    return (
        <div className="p-4 rounded-xl bg-[var(--nabu-purple)] bg-opacity-10 border border-[var(--nabu-purple)] text-[var(--nabu-ink)] mb-6">
            <div className="flex justify-between items-center">
                <Share2 className="w-5 h-5 text-[var(--nabu-purple)] mr-3 flex-shrink-0" /><div className="flex-grow"><p className="font-semibold text-sm mb-2">Share your draft: collaborate via comments or allow editing.</p><div className="flex items-center space-x-2"><input type="text" readOnly value={link} className="flex-grow p-2 text-xs rounded-lg bg-[#222] border border-[var(--nabu-purple)] text-[var(--nabu-ink)] font-mono truncate"/><button onClick={handleCopy} className="flex items-center px-3 py-2 text-xs font-medium rounded-lg bg-[var(--nabu-purple)] hover:bg-opacity-90 transition-opacity">Copy Link</button><button onClick={onClose} className="text-[var(--nabu-lilac)] hover:text-[var(--nabu-ink)] p-1 rounded-full"><X className="w-4 h-4" /></button></div><p className="mt-1 text-xs text-[var(--nabu-lilac)]">Mode: {mode === 'edit' ? 'Full Editing' : 'Comment Only'}</p></div>
            </div>
        </div>
    );
};


// --- 3. GLOBAL STATE AND PERSISTENCE (CLIENT-SIDE) ---
const initialProfile = {
    socialId: '', photoUrl: 'https://placehold.co/100x100/1a1a1a/C8A2C8?text=Photo',
    name: '', description: '', cv: '', ambitions: [], stories: ['', '', '']
};
const initialOpportunities = SEED_OPPORTUNITIES;


const App = () => {
    const getRouteFromHash = () => window.location.hash.substring(1).split('?')[0] || '/';
    const [route, setRoute] = useState(getRouteFromHash());


    const navigate = (newPath) => {
        const targetPath = newPath.startsWith('/') ? newPath : /${newPath};
        window.location.hash = targetPath;
        setRoute(targetPath);
    };


    const [profile, setProfile] = useState(initialProfile);
    const [opportunities, setOpportunities] = useState(initialOpportunities);
    const [swipes, setSwipes] = useState({});
    const [drafts, setDrafts] = useState({});
    const [shareTokens, setShareTokens] = useState({});
    const [posts, setPosts] = useState([]);
    const [toast, setT
