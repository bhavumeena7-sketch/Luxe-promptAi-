<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Photo Prompt Generator | Posh • Saree • Couple</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <!-- React and Babel -->
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <style>
        body { background: #0b0b0b; color: white; }
        .btn-premium { background: #ff0055; transition: all 0.3s; }
        .btn-premium:hover { background: #e6004d; transform: scale(1.02); }
        .card-dark { background: #151515; border: 1px solid #222; }
        .gradient-text { background: linear-gradient(to right, #ff0055, #ff6b6b); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        textarea { background: #1a1a1a !important; color: #ccc !important; border: 1px solid #333 !important; }
    </style>
</head>
<body>
    <div id="root"></div>

    <!-- Firebase SDK -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, onAuthStateChanged, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, onSnapshot, query, deleteDoc, doc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = JSON.parse(__firebase_config);
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'photo-generator-v1';

        window.FB = { 
            db, auth, appId,
            collection, addDoc, onSnapshot, query, deleteDoc, doc,
            signInAnonymously, onAuthStateChanged, signInWithCustomToken
        };
    </script>

    <script type="text/babel">
        const { useState, useEffect } = React;

        function App() {
            const [user, setUser] = useState(null);
            const [basePrompts, setBasePrompts] = useState([]);
            const [output, setOutput] = useState('');
            const [loading, setLoading] = useState(true);
            const [selection, setSelection] = useState({ category: 'Woman Saree', mood: 'Elegant' });
            const [newBase, setNewBase] = useState('');

            // Rule 3: Auth
            useEffect(() => {
                const initAuth = async () => {
                    if (!window.FB) { setTimeout(initAuth, 100); return; }
                    const { auth, signInAnonymously, onAuthStateChanged, signInWithCustomToken } = window.FB;
                    try {
                        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                            await signInWithCustomToken(auth, __initial_auth_token);
                        } else {
                            await signInAnonymously(auth);
                        }
                    } catch (e) { console.error(e); }
                    onAuthStateChanged(auth, setUser);
                };
                initAuth();
            }, []);

            // Rule 1 & 3: Fetch Prompts
            useEffect(() => {
                if (!user || !window.FB?.db) return;
                const { db, collection, onSnapshot, query, appId } = window.FB;
                const q = query(collection(db, 'artifacts', appId, 'public', 'data', 'base_prompts'));
                
                const unsub = onSnapshot(q, (snap) => {
                    const data = snap.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                    setBasePrompts(data.length > 0 ? data : [
                        { text: "Indian woman wearing luxury silk saree", category: "Woman Saree" },
                        { text: "Stylish Indian man in professional black suit", category: "Man Posh" },
                        { text: "Modern stylish girl fashion portrait", category: "Girl Stylish" },
                        { text: "Elegant Indian couple photoshoot in luxury setting", category: "Couple Luxury" }
                    ]);
                    setLoading(false);
                    setTimeout(() => lucide.createIcons(), 100);
                });
                return () => unsub();
            }, [user]);

            const generatePrompt = () => {
                const base = basePrompts.find(p => p.category === selection.category)?.text || basePrompts[0].text;
                const lights = ["soft studio lighting", "natural sunlight", "cinematic volumetric lighting"];
                const styles = "ultra realistic, 8K resolution, high detail, professional photography, masterpiece";
                
                const results = lights.map(l => `${base}, ${selection.mood} mood, ${l}, 85mm DSLR, ${styles}`);
                setOutput(results.join("\n\n---\n\n"));
            };

            const copyToClipboard = () => {
                if(!output) return;
                const textArea = document.createElement("textarea");
                textArea.value = output;
                document.body.appendChild(textArea);
                textArea.select();
                document.execCommand('copy');
                document.body.removeChild(textArea);
                alert("Prompts Copied! ✅");
            };

            const addBasePrompt = async () => {
                if(!newBase || !window.FB) return;
                const { db, collection, addDoc, appId } = window.FB;
                try {
                    await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'base_prompts'), {
                        text: newBase,
                        category: selection.category,
                        createdAt: Date.now()
                    });
                    setNewBase('');
                    alert("Base Prompt Added to Cloud! ☁️");
                } catch (e) { alert("Save failed"); }
            };

            const deleteBase = async (id) => {
                if(!confirm("Delete this base?")) return;
                const { db, doc, deleteDoc, appId } = window.FB;
                await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'base_prompts', id));
            };

            return (
                <div className="min-h-screen flex flex-col">
                    <header className="bg-[#111] py-8 text-center border-b border-[#222]">
                        <h1 className="text-3xl font-bold gradient-text">🔥 AI Photo Prompt Generator</h1>
                        <p className="text-gray-400 mt-2">Man • Woman • Girl • Couple • Posh • Saree • Luxury</p>
                    </header>

                    <main className="flex-grow max-w-4xl w-full mx-auto p-6 space-y-6">
                        {/* Prompt Generator Card */}
                        <div className="card-dark p-6 rounded-xl space-y-4">
                            <h2 className="text-xl font-semibold flex items-center gap-2">
                                <i data-lucide="sparkles" className="text-pink-500 w-5 h-5"></i>
                                Prompt Generator
                            </h2>
                            
                            <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                                <div className="space-y-1">
                                    <label className="text-xs text-gray-500 uppercase font-bold">Category</label>
                                    <select 
                                        className="w-full bg-[#222] border-none text-white p-3 rounded-lg outline-none"
                                        value={selection.category}
                                        onChange={e => setSelection({...selection, category: e.target.value})}
                                    >
                                        <option>Woman Saree</option>
                                        <option>Man Posh</option>
                                        <option>Girl Stylish</option>
                                        <option>Couple Luxury</option>
                                    </select>
                                </div>
                                <div className="space-y-1">
                                    <label className="text-xs text-gray-500 uppercase font-bold">Mood / Style</label>
                                    <select 
                                        className="w-full bg-[#222] border-none text-white p-3 rounded-lg outline-none"
                                        value={selection.mood}
                                        onChange={e => setSelection({...selection, mood: e.target.value})}
                                    >
                                        <option>Elegant</option>
                                        <option>Luxury</option>
                                        <option>Cinematic</option>
                                        <option>Romantic</option>
                                        <option>Edgy</option>
                                    </select>
                                </div>
                            </div>

                            <button 
                                onClick={generatePrompt}
                                className="w-full btn-premium py-4 rounded-lg font-bold text-lg shadow-lg"
                            >
                                Generate Pro Prompts
                            </button>

                            <textarea 
                                readOnly
                                value={output}
                                placeholder="Your AI prompt variations will appear here..."
                                className="w-full h-64 p-4 rounded-lg outline-none resize-none font-mono text-sm"
                            />

                            <button 
                                onClick={copyToClipboard}
                                className="w-full bg-white text-black py-3 rounded-lg font-bold flex items-center justify-center gap-2 hover:bg-gray-200"
                            >
                                <i data-lucide="copy" className="w-4 h-4"></i>
                                Copy All Prompts
                            </button>
                        </div>

                        {/* Admin Panel (Always Visible for You) */}
                        <div className="card-dark p-6 rounded-xl border-red-900/30 bg-red-900/5">
                            <h2 className="text-xl font-bold text-red-500 mb-4 flex items-center gap-2">
                                <i data-lucide="settings" className="w-5 h-5"></i>
                                Admin: Cloud Base Prompts
                            </h2>
                            <textarea 
                                placeholder="Add a new base description for the selected category..."
                                className="w-full h-24 p-3 mb-4 rounded-lg"
                                value={newBase}
                                onChange={e => setNewBase(e.target.value)}
                            />
                            <button 
                                onClick={addBasePrompt}
                                className="w-full bg-red-600 hover:bg-red-700 py-3 rounded-lg font-bold"
                            >
                                Save Base to Cloud
                            </button>

                            <div className="mt-6 space-y-2 max-h-60 overflow-y-auto pr-2">
                                {basePrompts.map(p => (
                                    <div key={p.id} className="flex justify-between items-start gap-4 p-3 bg-black/40 rounded border border-white/5 text-xs">
                                        <div className="flex-grow">
                                            <span className="text-pink-500 font-bold">[{p.category}]</span> {p.text}
                                        </div>
                                        {p.id && (
                                            <button onClick={() => deleteBase(p.id)} className="text-red-400 hover:text-red-200">
                                                <i data-lucide="trash-2" className="w-4 h-4"></i>
                                            </button>
                                        )}
                                    </div>
                                ))}
                            </div>
                        </div>
                    </main>

                    <footer className="bg-[#111] py-6 text-center text-gray-500 text-sm border-t border-[#222]">
                        © 2026 AI Prompt Generator | SaaS Cloud Version
                    </footer>
                </div>
            );
        }

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
        setTimeout(() => lucide.createIcons(), 500);
    </script>
</body>
</html># Luxe-promptAi-
