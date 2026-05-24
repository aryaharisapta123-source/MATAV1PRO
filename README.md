import React, { useState, useEffect, useRef } from 'react';
import { 
  Upload, 
  Image as ImageIcon, 
  Sparkles, 
  Download, 
  Camera, 
  Package, 
  ArrowRight, 
  Video, 
  Hand, 
  Smartphone, 
  ChevronLeft,
  Layout,
  Layers,
  CheckCircle2,
  Monitor,
  Loader2,
  Aperture,
  X,
  ChevronDown,
  ChevronUp,
  ZoomIn,
  Eye,
  Copy,
  FileText,
  Volume2,
  Wand2,
  PlayCircle,
  Mic,
  ExternalLink,
  Archive,
  RotateCcw,
  ShieldCheck,
  KeyRound,
  Globe
} from 'lucide-react';

const apiKey = ""; // Environment handles this
const GENERATIVE_MODEL = "gemini-2.5-flash-image-preview";
const TEXT_MODEL = "gemini-2.5-flash-preview-09-2025"; 
const TTS_MODEL = "gemini-2.5-flash-preview-tts";

// --- IDB STATE MANAGEMENT ---
const DB_NAME = 'MataStudioDB';
const STORE_NAME = 'app_state';

const openDB = () => new Promise((resolve, reject) => {
    const request = indexedDB.open(DB_NAME, 1);
    request.onupgradeneeded = (e) => {
        e.target.result.createObjectStore(STORE_NAME);
    };
    request.onsuccess = () => resolve(request.result);
    request.onerror = () => reject(request.error);
});

const saveToDB = async (key, data) => {
    try {
        const db = await openDB();
        const tx = db.transaction(STORE_NAME, 'readwrite');
        tx.objectStore(STORE_NAME).put(data, key);
        return new Promise((resolve, reject) => {
            tx.oncomplete = () => resolve(true);
            tx.onerror = () => reject(tx.error);
        });
    } catch (err) { console.error("IDB save error", err); }
};

const getFromDB = async (key) => {
    try {
        const db = await openDB();
        return new Promise((resolve, reject) => {
            const tx = db.transaction(STORE_NAME, 'readonly');
            const request = tx.objectStore(STORE_NAME).get(key);
            request.onsuccess = () => resolve(request.result);
            request.onerror = () => reject(request.error);
        });
    } catch (err) { console.error("IDB get error", err); return null; }
};

const clearDB = async (key) => {
     try {
        const db = await openDB();
        const tx = db.transaction(STORE_NAME, 'readwrite');
        tx.objectStore(STORE_NAME).delete(key);
        return new Promise((resolve, reject) => {
            tx.oncomplete = () => resolve(true);
            tx.onerror = () => reject(tx.error);
        });
    } catch (err) { console.error("IDB clear error", err); }
};

const INTERNATIONAL_LANGS = [
  { id: 'int_id', name: 'Indonesia', flag: '🇮🇩' },
  { id: 'int_en_us', name: 'English (US)', flag: '🇺🇸' },
  { id: 'int_en_uk', name: 'English (UK)', flag: '🇬🇧' },
  { id: 'int_ms', name: 'Malaysia', flag: '🇲🇾' },
  { id: 'int_sg', name: 'Singapore', flag: '🇸🇬' },
  { id: 'int_ar', name: 'Arabic', flag: '🇸🇦' },
  { id: 'int_zh', name: 'Mandarin', flag: '🇨🇳' },
  { id: 'int_cant', name: 'Cantonese', flag: '🇭🇰' },
  { id: 'int_ja', name: 'Japanese', flag: '🇯🇵' },
  { id: 'int_ko', name: 'Korean', flag: '🇰🇷' },
  { id: 'int_th', name: 'Thai', flag: '🇹🇭' },
  { id: 'int_vi', name: 'Vietnamese', flag: '🇻🇳' },
  { id: 'int_hi', name: 'Hindi', flag: '🇮🇳' },
  { id: 'int_tr', name: 'Turkish', flag: '🇹🇷' },
  { id: 'int_ru', name: 'Russian', flag: '🇷🇺' },
  { id: 'int_de', name: 'German', flag: '🇩🇪' },
  { id: 'int_fr', name: 'French', flag: '🇫🇷' },
  { id: 'int_it', name: 'Italian', flag: '🇮🇹' },
  { id: 'int_es', name: 'Spanish', flag: '🇪🇸' },
  { id: 'int_pt', name: 'Portuguese', flag: '🇵🇹' },
  { id: 'int_nl', name: 'Dutch', flag: '🇳🇱' }
];

const REGIONAL_LANGS = [
  { id: 'reg_jv', name: 'Jawa', flag: '🌾' },
  { id: 'reg_su', name: 'Sunda', flag: '🍃' },
  { id: 'reg_bt', name: 'Betawi', flag: '🧱' },
  { id: 'reg_md', name: 'Madura', flag: '🐂' },
  { id: 'reg_bl', name: 'Bali', flag: '🌺' },
  { id: 'reg_bk', name: 'Batak', flag: '⛰️' },
  { id: 'reg_mk', name: 'Minangkabau', flag: '🏘️' },
  { id: 'reg_my', name: 'Melayu', flag: '🛶' },
  { id: 'reg_ac', name: 'Aceh', flag: '🕌' },
  { id: 'reg_lp', name: 'Lampung', flag: '🐘' },
  { id: 'reg_pl', name: 'Palembang', flag: '🌉' },
  { id: 'reg_bj', name: 'Banjar', flag: '💎' },
  { id: 'reg_dy', name: 'Dayak', flag: '🌳' },
  { id: 'reg_bg', name: 'Bugis', flag: '⛵' },
  { id: 'reg_mc', name: 'Makassar', flag: '🌊' },
  { id: 'reg_tr', name: 'Toraja', flag: '🐃' },
  { id: 'reg_ss', name: 'Sasak', flag: '🏜️' },
  { id: 'reg_fl', name: 'Flores', flag: '🌅' },
  { id: 'reg_sb', name: 'Sumba', flag: '🐎' },
  { id: 'reg_ab', name: 'Ambon', flag: '🐚' },
  { id: 'reg_pp', name: 'Papua', flag: '☀️' },
  { id: 'reg_ba', name: 'Biak', flag: 'Parrot/🦜' },
  { id: 'reg_dn', name: 'Dani', flag: 'Fire/🔥' }
];

const fetchWithRetry = async (url, options) => {
  const delays = [1000, 2000, 4000, 8000, 16000];
  for (let i = 0; i <= delays.length; i++) {
    try {
      const response = await fetch(url, options);
      if (response.ok) return response;
      if (i === delays.length) return response; 
    } catch (error) {
      if (i === delays.length) throw error; 
    }
    if (i < delays.length) {
      await new Promise(res => setTimeout(res, delays[i]));
    }
  }
};

let sharedAudioCtx = null;
const playClick = () => {
  try {
    const AudioContext = window.AudioContext || window.webkitAudioContext;
    if (!AudioContext) return;
    if (!sharedAudioCtx) {
        sharedAudioCtx = new AudioContext();
    }
    if (sharedAudioCtx.state === 'suspended') {
        sharedAudioCtx.resume();
    }
    const osc = sharedAudioCtx.createOscillator();
    const gain = sharedAudioCtx.createGain();
    osc.type = 'sine';
    osc.frequency.setValueAtTime(600, sharedAudioCtx.currentTime);
    osc.frequency.exponentialRampToValueAtTime(250, sharedAudioCtx.currentTime + 0.05);
    gain.gain.setValueAtTime(0.1, sharedAudioCtx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.01, sharedAudioCtx.currentTime + 0.05);
    osc.connect(gain);
    gain.connect(sharedAudioCtx.destination);
    osc.start();
    osc.stop(sharedAudioCtx.currentTime + 0.05);
  } catch (err) {
    console.warn("Audio play error:", err);
  }
};

const safeCopyToClipboard = (text) => {
  const textArea = document.createElement("textarea");
  textArea.value = text;
  textArea.style.position = "fixed";
  textArea.style.left = "-9999px";
  textArea.style.top = "0";
  document.body.appendChild(textArea);
  textArea.focus();
  textArea.select();
  try { document.execCommand('copy'); } catch (err) { console.error('Fallback copy failed', err); }
  document.body.removeChild(textArea);
};

const processImageForDownload = (base64, ratioStr) => {
  return new Promise((resolve) => {
    const img = new Image();
    img.onload = () => {
      const canvas = document.createElement('canvas');
      let targetRatio;
      if (ratioStr === '16:9') targetRatio = 16/9;
      else if (ratioStr === '9:16') targetRatio = 9/16;
      else targetRatio = 1;

      const srcRatio = img.width / img.height;
      let renderWidth, renderHeight, offsetX, offsetY;

      if (srcRatio > targetRatio) {
        renderHeight = img.height;
        renderWidth = img.height * targetRatio;
        offsetX = (img.width - renderWidth) / 2;
        offsetY = 0;
      } else {
        renderWidth = img.width;
        renderHeight = img.width / targetRatio;
        offsetX = 0;
        if (targetRatio > 1) { offsetY = (img.height - renderHeight) / 2; } 
        else { offsetY = (img.height - renderHeight) * 0.1; }
      }

      canvas.width = Math.floor(renderWidth);
      canvas.height = Math.floor(renderHeight);
      const ctx = canvas.getContext('2d');
      ctx.drawImage(img, offsetX, offsetY, renderWidth, renderHeight, 0, 0, canvas.width, canvas.height);
      resolve(canvas.toDataURL('image/png'));
    };
    img.src = base64;
  });
};

const writeString = (view, offset, string) => {
  for (let i = 0; i < string.length; i++) {
    view.setUint8(offset + i, string.charCodeAt(i));
  }
};

const base64ToWavUrl = (base64PCM) => {
  const binaryString = window.atob(base64PCM);
  const len = binaryString.length;
  const bytes = new Uint8Array(len);
  for (let i = 0; i < len; i++) {
    bytes[i] = binaryString.charCodeAt(i);
  }

  const sampleRate = 24000;
  const numChannels = 1;
  const bitsPerSample = 16;
  const blockAlign = numChannels * (bitsPerSample / 8);
  const byteRate = sampleRate * blockAlign;
  const dataSize = bytes.length;
  
  const buffer = new ArrayBuffer(44 + dataSize);
  const view = new DataView(buffer);

  writeString(view, 0, 'RIFF');
  view.setUint32(4, 36 + dataSize, true);
  writeString(view, 8, 'WAVE');
  writeString(view, 12, 'fmt ');
  view.setUint32(16, 16, true); 
  view.setUint16(20, 1, true); 
  view.setUint16(22, numChannels, true);
  view.setUint32(24, sampleRate, true);
  view.setUint32(28, byteRate, true);
  view.setUint16(32, blockAlign, true);
  view.setUint16(34, bitsPerSample, true);
  writeString(view, 36, 'data');
  view.setUint32(40, dataSize, true);

  const pcmData = new Uint8Array(buffer, 44);
  pcmData.set(bytes);

  const blob = new Blob([buffer], { type: 'audio/wav' });
  return URL.createObjectURL(blob);
};

const IDENTITY_LOCK = "CRITICAL IDENTITY LOCK: The person MUST have the EXACT same face, facial structure, hair, skin tone, and body proportions as the person in IMAGE 1. This is the exact same person. Do NOT change their identity, ethnicity, or body type. Clone their physical appearance perfectly.";

const getModeLogic = (modeId, index, productName, productDesc, background, ugcStyle = 'Handheld') => {
    let subjectPrompt = "";
    let negPrompt = "";
    let cameraLogic = "";
    let motionCamera = "";
    let lipSyncLogic = "AMBIENT: Mouth closed or natural breathing, no talking.";

    switch (modeId) {
        case 'ugc':
            if (ugcStyle === 'Vlog') {
                cameraLogic = "Shot on iPhone 15 Pro Max front camera. Raw unedited vlog style. Wide angle lens, harsh directional sunlight or strong room light. Zero post-processing.";
                motionCamera = "Smooth handheld tracking shot, natural forward movement. Model walking and talking.";
            } else if (ugcStyle === 'Tripod') {
                cameraLogic = "Shot on iPhone 15 Pro Max mounted on a simple tripod. Raw unedited smartphone photo, single harsh directional light source creating distinct shadows, authentic everyday creator setup.";
                motionCamera = "Perfectly static camera on a tripod, no movement.";
            } else {
                cameraLogic = "HANDHELD SELFIE STYLE. Shot on iPhone 15 Pro Max front camera. Candid framing, hard flash or harsh direct sunlight, authentic raw social media snapshot.";
                motionCamera = "Slight natural handheld camera shake.";
            }
            
            subjectPrompt = `RAW AUTHENTIC UGC PHOTO (${cameraLogic.split('.')[0]}). Model facing camera naturally. Candid mid-sentence expression. Model casually interacting with ${productName} (${productDesc}) in an everyday setting. CRITICAL: Extremely realistic human skin texture, visible pores, subtle blemishes, unedited natural look. Single harsh light casting distinct, realistic shadows for 3D depth. Real-world lived-in background, no studio setup.`;
            negPrompt = "studio lighting, softbox, flat lighting, professional photography, DSLR, perfect smooth plastic skin, beauty filter, airbrushed, over-polished, cinematic lighting, 3d render, illustration.";
            lipSyncLogic = "LIP-SYNC: Model is speaking, mouth moving precisely to dialogue: '[SCRIPT_SCENE_TERSEBUT]'.";
            break;

        case 'commercial':
            cameraLogic = "Professional high-end commercial product photography. Shot on Hasselblad medium format. Single harsh directional spotlight, deep striking shadows, crisp sharp details, highly polished catalog aesthetic, 8k resolution.";
            subjectPrompt = `AWARD-WINNING COMMERCIAL SHOT. Masterpiece photography. Model looking pristine and perfectly styled, elegantly presenting ${productName} (${productDesc}). The product is heroically displayed with distinct shadows for 3D dimensionality, high contrast, and hyper-realistic textures. Clean, aesthetic background.`;
            negPrompt = "Amateur, smartphone photo, messy background, low quality, flat lighting, softbox, grainy, blurry, casual everyday lighting, distorted, bad anatomy, ugly, candid, snapshot, text, watermark.";
            motionCamera = "Smooth cinematic slider movement, slow motion, showcasing the product elegantly.";
            lipSyncLogic = "LIP-SYNC: Professional commercial delivery, smiling while speaking dialogue: '[SCRIPT_SCENE_TERSEBUT]'.";
            break;

        case 'mirror':
            cameraLogic = "Authentic mirror selfie. Shot on iPhone 15 Pro. Harsh directional room light or hard flash creating deep shadows, photorealistic.";
            subjectPrompt = `RAW MIRROR SELFIE PHOTO. Model standing in front of a mirror taking a mirror selfie with a smartphone. Model is stylishly showcasing ${productName} (${productDesc}). Trendy OOTD aesthetic. Real skin texture, unedited look. Single harsh light creating bold 3D dimensionality. Mirror reflection visible.`;
            negPrompt = "Flat lighting, soft ambient light, TV commercial look, amateur, basic white background, 3d render, illustration, not a mirror selfie, missing phone.";
            motionCamera = "Slight natural handheld camera shake, mimicking a mirror selfie recording.";
            lipSyncLogic = "AMBIENT: Casual posing in the mirror, natural breathing, no talking.";
            break;

        case 'pov':
            cameraLogic = "Authentic everyday first-person POV, shot on an iPhone 15, looking down, harsh directional lighting casting distinct hand shadows, photorealistic.";
            subjectPrompt = `RAW FIRST-PERSON POV SHOT. Authentic everyday lifestyle. Only hands visible naturally holding or unboxing ${productName} (${productDesc}). Shot from user's eye-level looking down at hands. Single harsh directional light, distinct hard shadows. NO FACES, NO BODIES. Real skin texture, highly detailed macro focus on product. ABSOLUTELY NO TEXT OR UI.`;
            negPrompt = "Face, head, hair, eyes, mouth, neck, torso, legs, feet, full body, flat lighting, over-rendered, artificial studio lighting, highly polished professional shot, 3D render, illustration, painting, text, typography, watermark, UI.";
            motionCamera = "Authentic first-person POV, slight natural smartphone camera shake, looking down. Pair of hands naturally interacting with the product.";
            break;
            
        default:
            cameraLogic = "Standard high quality camera setup.";
            subjectPrompt = `High quality photorealistic shot of model with ${productName} (${productDesc}).`;
            negPrompt = "Distorted, blurry, illustration, 3d.";
            motionCamera = "Static.";
            break;
    }

    return { subjectPrompt, negPrompt, cameraLogic, motionCamera, lipSyncLogic };
};

const LayoutWrapper = ({ children, className = "" }) => (
  <div className={`min-h-screen bg-[#121214] text-zinc-200 font-sans selection:bg-zinc-800 selection:text-emerald-400 overflow-x-hidden relative ${className}`}>
    <style>
      {`
        @keyframes scanline {
          0% { top: -5%; opacity: 0; }
          5% { opacity: 0.4; }
          95% { opacity: 0.4; }
          100% { top: 105%; opacity: 0; }
        }
        .laser-scan {
          position: absolute;
          left: 0;
          width: 100%;
          height: 1px;
          background: linear-gradient(90deg, transparent, rgba(16, 185, 129, 0.4), rgba(255, 255, 255, 0.6), rgba(16, 185, 129, 0.4), transparent);
          box-shadow: 0 0 10px 1px rgba(16, 185, 129, 0.3);
          animation: scanline 6s linear infinite;
          z-index: 0;
          pointer-events: none;
        }
        .custom-scrollbar::-webkit-scrollbar {
          width: 6px;
          height: 6px;
        }
        .custom-scrollbar::-webkit-scrollbar-track {
          background: #121214;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb {
          background: #27272a;
          border-radius: 9999px;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb:hover {
          background: #3f3f46;
        }
      `}
    </style>
    <div className="laser-scan"></div>
    
    <div className="fixed top-[-10%] left-[-5%] w-[60%] h-[30%] rounded-full bg-emerald-950/20 blur-[130px] pointer-events-none transform translate-z-0 z-0"></div>
    <div className="fixed bottom-[-10%] right-[-5%] w-[60%] h-[30%] rounded-full bg-zinc-900/40 blur-[130px] pointer-events-none transform translate-z-0 z-0"></div>
    
    <div className="relative z-10 w-full h-full flex flex-col min-h-screen">
      {children}
    </div>
  </div>
);

const Navbar = ({ onBack, onViewResult, onReset, authMode }) => (
  <div className="absolute top-0 left-0 right-0 z-50 px-6 md:px-10 py-5 flex items-center justify-between bg-[#121214]/85 backdrop-blur-md border-b border-zinc-900">
    <div className="absolute left-6 md:left-10 flex items-center gap-3.5 z-10">
      <div className="w-10 h-10 rounded-xl bg-gradient-to-tr from-emerald-600 to-teal-500 flex items-center justify-center shadow-lg shadow-emerald-950/40 border border-emerald-400/20">
        <Eye className="w-5.5 h-5.5 text-white" />
      </div>
      <div className="flex flex-col">
        <div className="flex items-center gap-2">
          <span className="font-bold tracking-tight text-lg text-white leading-none uppercase">Mata Studio</span>
          <span className="text-[9px] px-1.5 py-0.5 bg-emerald-500/10 text-emerald-400 border border-emerald-500/20 rounded font-semibold tracking-wider">V1.0</span>
        </div>
        <span className="text-[10px] text-zinc-500 tracking-wider uppercase mt-1">AI VISUAL ENGINE</span>
      </div>
    </div>

    <div className="flex items-center gap-2.5 z-10 ml-auto">
      {authMode === 'admin' && (
        <span className="px-3 py-1 bg-emerald-500/10 border border-emerald-500/20 text-emerald-400 text-[10px] font-bold uppercase tracking-widest rounded-lg flex items-center gap-1.5">
          <ShieldCheck className="w-3.5 h-3.5" /> Admin
        </span>
      )}
      
      {onViewResult && (
        <button 
          type="button"
          onClick={() => { playClick(); onViewResult(); }} 
          className="bg-emerald-500 hover:bg-emerald-400 text-neutral-950 px-4 py-2 rounded-lg flex items-center gap-2 text-xs font-bold uppercase tracking-wider transition-all duration-200 shadow-md shadow-emerald-950/20"
        >
          <Eye className="w-3.5 h-3.5" /> Hasil Visual
        </button>
      )}
      {onReset && (
        <button 
          type="button"
          onClick={() => { playClick(); onReset(); }} 
          className="bg-zinc-900 border border-zinc-800 px-4 py-2 rounded-lg flex items-center gap-2 text-xs font-semibold text-zinc-300 uppercase tracking-wider hover:bg-zinc-800 hover:text-white transition-all duration-200 active:scale-95"
        >
          <RotateCcw className="w-3.5 h-3.5" /> Reset
        </button>
      )}
      {onBack && (
        <button 
          type="button"
          onClick={() => { playClick(); onBack(); }} 
          className="bg-zinc-900 border border-zinc-800 px-4 py-2 rounded-lg flex items-center gap-2 text-xs font-semibold text-zinc-300 uppercase tracking-wider hover:bg-zinc-800 hover:text-white transition-all duration-200 active:scale-95"
        >
          <ChevronLeft className="w-3.5 h-3.5" /> Kembali
        </button>
      )}
    </div>
  </div>
);

const SectionTitle = ({ title, subtitle }) => (
  <div className="animate-in fade-in slide-in-from-bottom-3 duration-500">
    <h2 className="text-2xl md:text-3xl font-bold text-white mb-2 tracking-tight">{title}</h2>
    {subtitle && <p className="text-zinc-400 text-xs md:text-sm font-normal leading-relaxed max-w-xl">{subtitle}</p>}
  </div>
);

const SearchableLanguageDropdown = ({ 
  label, 
  selectedId, 
  onChange, 
  placeholder = "Pilih Bahasa...", 
  languages, 
  isOptional = false 
}) => {
  const [isOpen, setIsOpen] = useState(false);
  const [search, setSearch] = useState("");
  const dropdownRef = useRef(null);

  useEffect(() => {
    const handleClickOutside = (event) => {
      if (dropdownRef.current && !dropdownRef.current.contains(event.target)) {
        setIsOpen(false);
      }
    };
    document.addEventListener("mousedown", handleClickOutside);
    return () => document.removeEventListener("mousedown", handleClickOutside);
  }, []);

  const selectedLang = languages.find(l => l.id === selectedId);

  const filteredLangs = languages.filter(l => 
    l.name.toLowerCase().includes(search.toLowerCase())
  );

  const intLangs = filteredLangs.filter(l => l.id.startsWith('int_'));
  const regLangs = filteredLangs.filter(l => l.id.startsWith('reg_'));
  const customLangs = filteredLangs.filter(l => l.id.startsWith('custom_'));

  return (
    <div className="space-y-2 relative" ref={dropdownRef}>
      <label className="text-[10px] font-bold text-zinc-400 uppercase tracking-widest block">
        {label}
      </label>
      
      <button
        type="button"
        onClick={() => { playClick(); setIsOpen(!isOpen); setSearch(""); }}
        className={`w-full flex items-center justify-between bg-[#121214] border ${isOpen ? 'border-emerald-500/50' : 'border-zinc-800'} rounded-xl px-4 py-3.5 text-xs text-left text-white transition-all hover:bg-zinc-900/40`}
      >
        <span className="flex items-center gap-2.5 font-semibold">
          {selectedLang ? (
            <>
              <span className="text-sm leading-none">{selectedLang.flag}</span>
              <span>{selectedLang.name}</span>
            </>
          ) : (
            <span className="text-zinc-500 italic font-normal">{placeholder}</span>
          )}
        </span>
        <ChevronDown className={`w-4 h-4 text-zinc-500 transition-transform duration-200 ${isOpen ? 'rotate-180 text-emerald-400' : ''}`} />
      </button>

      {isOpen && (
        <div className="absolute left-0 right-0 mt-1.5 bg-[#18181b] border border-zinc-800 rounded-xl shadow-2xl z-50 p-2 flex flex-col gap-2 max-h-[300px] animate-in fade-in slide-in-from-top-2 duration-150">
          <div className="relative shrink-0">
            <input
              type="text"
              placeholder="Cari Bahasa..."
              value={search}
              onChange={(e) => setSearch(e.target.value)}
              className="w-full bg-[#121214] border border-zinc-800 rounded-lg pl-9 pr-3 py-2 text-xs text-white placeholder:text-zinc-500 focus:outline-none focus:border-emerald-500/30 transition-all font-medium"
              autoFocus
            />
            <div className="absolute left-3 top-1/2 -translate-y-1/2 text-zinc-500">
              <svg className="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path></svg>
            </div>
            {search && (
              <button 
                type="button" 
                onClick={() => setSearch("")} 
                className="absolute right-3 top-1/2 -translate-y-1/2 text-zinc-500 hover:text-white"
              >
                <X className="w-3 h-3" />
              </button>
            )}
          </div>

          <div className="flex-grow overflow-y-auto custom-scrollbar space-y-3.5 pr-0.5">
            {isOptional && (
              <button
                type="button"
                onClick={() => { playClick(); onChange(""); setIsOpen(false); }}
                className="w-full text-left p-2 rounded hover:bg-zinc-900/65 text-[10px] font-bold text-zinc-500 hover:text-white uppercase tracking-wider"
              >
                ❌ Kosongkan Pilihan
              </button>
            )}

            {intLangs.length > 0 && (
              <div>
                <span className="text-[9px] font-bold tracking-widest text-zinc-500 uppercase block px-2 mb-1.5">Kategori Bahasa Internasional</span>
                <div className="grid grid-cols-1 gap-1">
                  {intLangs.map(l => (
                    <button
                      key={l.id}
                      type="button"
                      onClick={() => { playClick(); onChange(l.id); setIsOpen(false); }}
                      className={`flex items-center gap-2.5 px-3 py-2 rounded-lg text-left text-xs transition-all ${selectedId === l.id ? 'bg-emerald-500/15 border border-emerald-500/30 text-emerald-400' : 'bg-transparent border border-transparent text-zinc-400 hover:bg-zinc-900/60 hover:text-white'}`}
                    >
                      <span className="text-base leading-none">{l.flag}</span>
                      <span className="font-semibold">{l.name}</span>
                    </button>
                  ))}
                </div>
              </div>
            )}

            {regLangs.length > 0 && (
              <div>
                <span className="text-[9px] font-bold tracking-widest text-zinc-500 uppercase block px-2 mb-1.5">Kategori Bahasa Daerah Indonesia</span>
                <div className="grid grid-cols-1 gap-1">
                  {regLangs.map(l => (
                    <button
                      key={l.id}
                      type="button"
                      onClick={() => { playClick(); onChange(l.id); setIsOpen(false); }}
                      className={`flex items-center gap-2.5 px-3 py-2 rounded-lg text-left text-xs transition-all ${selectedId === l.id ? 'bg-emerald-500/15 border border-emerald-500/30 text-emerald-400' : 'bg-transparent border border-transparent text-zinc-400 hover:bg-zinc-900/60 hover:text-white'}`}
                    >
                      <span className="text-base leading-none">{l.flag}</span>
                      <span className="font-semibold">{l.name}</span>
                    </button>
                  ))}
                </div>
              </div>
            )}

            {customLangs.length > 0 && (
              <div>
                <span className="text-[9px] font-bold tracking-widest text-zinc-500 uppercase block px-2 mb-1.5">Bahasa Custom</span>
                <div className="grid grid-cols-1 gap-1">
                  {customLangs.map(l => (
                    <button
                      key={l.id}
                      type="button"
                      onClick={() => { playClick(); onChange(l.id); setIsOpen(false); }}
                      className={`flex items-center gap-2.5 px-3 py-2 rounded-lg text-left text-xs transition-all ${selectedId === l.id ? 'bg-emerald-500/15 border border-emerald-500/30 text-emerald-400' : 'bg-transparent border border-transparent text-zinc-400 hover:bg-zinc-900/60 hover:text-white'}`}
                    >
                      <span className="text-base leading-none">{l.flag}</span>
                      <span className="font-semibold">{l.name}</span>
                    </button>
                  ))}
                </div>
              </div>
            )}

            {filteredLangs.length === 0 && (
              <p className="text-center text-[10px] text-zinc-650 py-4 italic">Bahasa tidak ditemukan.</p>
            )}
          </div>
        </div>
      )}
    </div>
  );
};

const UploadZone = ({ image, onClick, label, icon: Icon }) => (
  <div className="space-y-2 group">
    <div className="flex justify-between items-baseline px-0.5">
      <span className="text-[10px] font-bold tracking-wider text-zinc-400 uppercase">{label}</span>
      {image && <span className="text-[10px] text-emerald-400 font-bold uppercase tracking-wider flex items-center gap-1"><CheckCircle2 className="w-3 h-3"/> Ready</span>}
    </div>
    <div onClick={() => { playClick(); onClick(); }} className={`relative w-full aspect-square rounded-xl overflow-hidden cursor-pointer transition-all duration-300 border border-zinc-800 bg-zinc-900/20 hover:border-zinc-700 hover:bg-zinc-900/40`}>
      {image ? (
        <>
          <img src={image} className="w-full h-full object-cover opacity-90 group-hover:opacity-100 transition-all duration-500 scale-100 group-hover:scale-[1.02]" />
          <div className="absolute inset-0 flex items-center justify-center opacity-0 group-hover:opacity-100 transition-opacity duration-200 bg-black/50 backdrop-blur-[2px]">
            <span className="px-4 py-2 rounded-lg border border-zinc-700 bg-zinc-900/90 text-xs font-medium text-white shadow-lg">Ganti Gambar</span>
          </div>
          <div className="absolute bottom-2.5 right-2.5 md:hidden">
            <div className="bg-emerald-500 rounded-full p-1"><CheckCircle2 className="w-3.5 h-3.5 text-black" /></div>
          </div>
        </>
      ) : (
        <div className="absolute inset-0 flex flex-col items-center justify-center gap-3.5 p-4 text-center">
          <div className="w-11 h-11 rounded-xl border border-zinc-800 flex items-center justify-center bg-zinc-900 text-zinc-400 group-hover:text-emerald-400 group-hover:border-emerald-500/30 transition-all duration-300">
            <Icon className="w-5 h-5" />
          </div>
          <p className="text-[10px] text-zinc-500 group-hover:text-zinc-300 font-bold tracking-widest transition-colors">UNGGAH ASET</p>
        </div>
      )}
    </div>
  </div>
);

const ButtonCTA = ({ onClick, children, disabled }) => (
  <button 
    type="button"
    onClick={() => { playClick(); onClick(); }} 
    disabled={disabled} 
    className={`relative overflow-hidden w-full py-3.5 px-6 bg-emerald-500 text-neutral-950 font-bold text-xs tracking-widest uppercase rounded-lg hover:bg-emerald-400 active:scale-[0.98] transition-all duration-200 disabled:opacity-20 disabled:cursor-not-allowed disabled:hover:scale-100 flex items-center justify-center gap-2.5 shadow-md shadow-emerald-950/20`}
  >
    {children}
  </button>
);

const MODE_BACKGROUNDS = {
    ugc: [
        'Kamar Kosan Aesthetic (Lampu Warm White)',
        'Warkop Kekinian (Lampu Kuning Hangat)',
        'Teras Depan Rumah Cluster',
        'Kafe Aesthetic (Sore Hari)',
        'Lorong Minimarket',
        'Kabin Mobil Siang Hari'
    ],
    commercial: [
        'Studio Setup Polos (Latar Putih Bersih)',
        'Meja Marmer Elegan (Lampu Studio Cerah)',
        'Latar Pastel Estetik Minimalis',
        'Podium Display Produk Mewah',
        'Set Ruang Tamu Skandinavia Modern'
    ],
    mirror: [
        'Kamar Estetik (Cermin Full Body)',
        'Kamar Mandi Mewah (Cermin Wastafel)',
        'Fitting Room Mall (Cahaya Terang)',
        'Lift Aesthetic (Cermin Full)',
        'Studio Minimalis (Cermin Bulat Besar)'
    ],
    pov: [
        'Meja Kerja Kayu Jati (Suasana Kantor)',
        'Kasur Sprei Motif Lokal (Unboxing Paket)',
        'Di Atas Jok Motor Matic (Jalanan Komplek)',
        'Di Balik Setir Mobil (Background Jalanan)',
        'Meja Kafe Estetik (Dekat Jendela)'
    ]
};

const VOICES = [
    { id: 'Puck', name: 'Budi (Pria)' },
    { id: 'Kore', name: 'Ayu (Wanita)' },
    { id: 'Charon', name: 'Rio (Pria)' },
    { id: 'Zephyr', name: 'Indra (Pria)' },
    { id: 'Fenrir', name: 'Dedi (Pria Berat)' }
];

const TONES = [
    'Profesional', 'Antusias', 'Meyakinkan', 'Santai', 'Edukatif', 
    'Humor', 'Inspiratif', 'Serius', 'Dramatis'
];

export default function App() {
  const [isRestoring, setIsRestoring] = useState(true);
  const [view, setView] = useState('login'); 
  const [authMode, setAuthMode] = useState('logged_out'); 
  const [loginRole, setLoginRole] = useState('user'); 
  const [loginInput, setLoginInput] = useState('');
  const [loginError, setLoginError] = useState('');
  const [validTokens, setValidTokens] = useState([
    'MATA_V1_101', 'MATA_V1_102', 'MATA_V1_103', 'MATA_V1_104', 'MATA_V1_105', 
    'MATA_V1_106', 'MATA_V1_107', 'MATA_V1_108', 'MATA_V1_109', 'MATA_V1_110'
  ]); 
  const [customTokenInput, setCustomTokenInput] = useState('');

  // LANGUAGE & CUSTOMIZATION STATES
  const [primaryLang, setPrimaryLang] = useState("int_id");
  const [secondaryLang, setSecondaryLang] = useState("");
  const [tertiaryLang, setTertiaryLang] = useState("");
  
  const [customLangs, setCustomLangs] = useState([]);
  const [newCustomLang, setNewCustomLang] = useState({
    name: "",
    style: "",
    dialect: "",
    greeting: ""
  });
  const [showCustomForm, setShowCustomForm] = useState(false);
  const [savedPresets, setSavedPresets] = useState([]);
  const [presetNameInput, setPresetNameInput] = useState("");

  // App State
  const [selectedMode, setSelectedMode] = useState(null);
  const [modelImage, setModelImage] = useState(null);
  const [productImage, setProductImage] = useState(null);
  const [productName, setProductName] = useState("");
  const [productDesc, setProductDesc] = useState("");
  const [dynamicBackgrounds, setDynamicBackgrounds] = useState(MODE_BACKGROUNDS.ugc);

  const [bgType, setBgType] = useState('preset'); 
  const [customBg, setCustomBg] = useState("");

  const [isAnalyzingProduct, setIsAnalyzingProduct] = useState(false);
  const [isProductConfirmed, setIsProductConfirmed] = useState(false);

  const [config, setConfig] = useState({
    background: MODE_BACKGROUNDS.ugc[0],
    ratio: '9:16',
    scenes: 4,
    ugcStyle: 'Handheld',
    language: 'id' 
  });

  const [isGenerating, setIsGenerating] = useState(false);
  const [generatedImages, setGeneratedImages] = useState([]);
  const [sceneLoadings, setSceneLoadings] = useState({});
  const [scenePrompts, setScenePrompts] = useState({});
  const [previewImage, setPreviewImage] = useState(null);
  const [copiedStates, setCopiedStates] = useState({});

  const [isAudioStudioOpen, setIsAudioStudioOpen] = useState(false);
  const [isAudioStudioMinimized, setIsAudioStudioMinimized] = useState(false);

  const [voScript, setVoScript] = useState("");
  const [voDuration, setVoDuration] = useState("15s");
  const [voVoice, setVoVoice] = useState(VOICES[0].id);
  const [voTone, setVoTone] = useState(TONES[1]);
  const [voAudioUrl, setVoAudioUrl] = useState(null);
  const [isGeneratingVOScript, setIsGeneratingVOScript] = useState(false);
  const [isGeneratingVOAudio, setIsGeneratingVOAudio] = useState(false);
  
  const [isVideoGeneratorsOpen, setIsVideoGeneratorsOpen] = useState(false);
  const [isDownloadingZip, setIsDownloadingZip] = useState(false);

  useEffect(() => {
      const restoreState = async () => {
          const savedState = await getFromDB('mata_state');
          if (savedState) {
              if (savedState.view) setView(savedState.view);
              if (savedState.authMode) setAuthMode(savedState.authMode);
              if (savedState.validTokens) setValidTokens(savedState.validTokens);
              if (savedState.selectedMode) setSelectedMode(savedState.selectedMode);
              if (savedState.modelImage) setModelImage(savedState.modelImage);
              if (savedState.productImage) setProductImage(savedState.productImage);
              if (savedState.productName) setProductName(savedState.productName);
              if (savedState.productDesc) setProductDesc(savedState.productDesc);
              if (savedState.dynamicBackgrounds) setDynamicBackgrounds(savedState.dynamicBackgrounds);
              if (savedState.bgType) setBgType(savedState.bgType);
              if (savedState.customBg) setCustomBg(savedState.customBg);
              if (savedState.isProductConfirmed !== undefined) setIsProductConfirmed(savedState.isProductConfirmed);
              if (savedState.config) setConfig(savedState.config);
              if (savedState.generatedImages) setGeneratedImages(savedState.generatedImages);
              if (savedState.scenePrompts) setScenePrompts(savedState.scenePrompts);
              if (savedState.voScript) setVoScript(savedState.voScript);
              if (savedState.voDuration) setVoDuration(savedState.voDuration);
              if (savedState.voVoice) setVoVoice(savedState.voVoice);
              if (savedState.voTone) setVoTone(savedState.voTone);
              
              if (savedState.primaryLang) setPrimaryLang(savedState.primaryLang);
              if (savedState.secondaryLang) setSecondaryLang(savedState.secondaryLang);
              if (savedState.tertiaryLang) setTertiaryLang(savedState.tertiaryLang);
              if (savedState.customLangs) setCustomLangs(savedState.customLangs);
              if (savedState.savedPresets) setSavedPresets(savedState.savedPresets);
          }
          setIsRestoring(false);
      };
      restoreState();
  }, []);

  useEffect(() => {
      if (isRestoring) return;
      const timeoutId = setTimeout(() => {
          const safeSelectedMode = selectedMode ? { id: selectedMode.id, title: selectedMode.title, desc: selectedMode.desc } : null;
          const stateToSave = {
              view, authMode, validTokens, selectedMode: safeSelectedMode,
              modelImage, productImage, productName, productDesc,
              dynamicBackgrounds, bgType, customBg, isProductConfirmed,
              config, generatedImages, scenePrompts, voScript,
              voDuration, voVoice, voTone,
              primaryLang, secondaryLang, tertiaryLang,
              customLangs, savedPresets
          };
          saveToDB('mata_state', stateToSave);
      }, 500);
      return () => clearTimeout(timeoutId);
  }, [view, authMode, validTokens, selectedMode, modelImage, productImage, productName, productDesc, dynamicBackgrounds, bgType, customBg, isProductConfirmed, config, generatedImages, scenePrompts, voScript, voDuration, voVoice, voTone, primaryLang, secondaryLang, tertiaryLang, customLangs, savedPresets, isRestoring]);

  useEffect(() => {
    window.scrollTo({ top: 0, behavior: 'smooth' });
  }, [view]);

  useEffect(() => {
    if (view === 'mode-selection' || view === 'upload-setup') {
      const expensiveRedBg = 'Latar premium Studio Foto (Neutral Grey)';
      setBgType('custom');
      setCustomBg(expensiveRedBg);
    }
  }, [view]);

  const modelInputRef = useRef(null);
  const productInputRef = useRef(null);

  const handleLogin = () => {
    playClick();
    if (loginRole === 'admin') {
       if (loginInput === 'mata_mastering123') { 
           setAuthMode('admin');
           setLoginError('');
           setView('admin-dashboard');
       } else {
           setLoginError('Password Admin salah!');
       }
    } else {
       if (validTokens.includes(loginInput.toUpperCase())) {
           setAuthMode('user');
           setLoginError('');
           setView('mode-selection');
       } else {
           setLoginError('Token tidak valid atau sudah kadaluarsa!');
       }
    }
  };

  const handleLogout = () => {
    playClick();
    clearDB('mata_state');
    setAuthMode('logged_out');
    setLoginInput('');
    setLoginError('');
    setView('login');
  };

  const handleResetAndNew = () => {
    playClick();
    clearDB('mata_state');
    setSelectedMode(null);
    setModelImage(null);
    setProductImage(null);
    setProductName("");
    setProductDesc("");
    setIsProductConfirmed(false);
    setBgType('preset');
    setCustomBg("");
    setPrimaryLang("int_id");
    setSecondaryLang("");
    setTertiaryLang("");
    setCustomLangs([]);
    setConfig({
      background: MODE_BACKGROUNDS.ugc[0],
      ratio: '9:16',
      scenes: 4,
      ugcStyle: 'Handheld',
      language: 'id'
    });
    setGeneratedImages([]);
    setSceneLoadings({});
    setScenePrompts({});
    setPreviewImage(null);
    setCopiedStates({});
    setVoScript("");
    setVoAudioUrl(null);
    setIsAudioStudioOpen(false);
    setIsAudioStudioMinimized(false);
    setIsVideoGeneratorsOpen(false);
    setView('mode-selection');
  };

  const updateSceneData = (index, updates) => {
    setGeneratedImages(prev => {
        const newArr = [...prev];
        if (newArr[index]) {
            newArr[index] = { ...newArr[index], ...updates };
        }
        return newArr;
    });
  };

  const getPromptForScene = (sceneData) => {
    if (!sceneData) return "";
    let contextLogic = "";
    if (sceneData.voActive) {
        if (sceneData.isBRoll) {
            contextLogic = `NARRATIVE VOICE-OVER: '${sceneData.voText}'. Product remains the main focus.`;
        } else {
            contextLogic = `LIP-SYNC: Model is speaking, mouth moving precisely to dialogue: '${sceneData.voText}'.`;
        }
    } else {
        contextLogic = sceneData.isBRoll ? "AMBIENT: No human visible, purely product focused." : "AMBIENT: Mouth closed or natural breathing, no talking.";
    }
    return `PROMPT: ${sceneData.videoMotion} CAMERA: ${sceneData.videoCamera} DETAILS: Photorealistic high-fidelity video generation. Maintain strict consistency with the provided image reference. CONTEXT: ${contextLogic} ENVIRONMENT: ${sceneData.activeBackground}. NEGATIVE: distortion, morphing, bad hands, text overlays.`;
  };

  const handleCopyPrompt = (index) => {
    playClick();
    const sceneData = generatedImages[index];
    if (!sceneData) return;

    const dynamicPrompt = getPromptForScene(sceneData);

    safeCopyToClipboard(dynamicPrompt);
    setCopiedStates(prev => ({ ...prev, [index]: true }));
    setTimeout(() => {
        setCopiedStates(prev => ({ ...prev, [index]: false }));
    }, 2000);
  };

  const getCombinedLangsList = () => {
    const customs = customLangs.map(l => ({
      id: `custom_${l.name.toLowerCase().replace(/\s+/g, '_')}`,
      name: l.name,
      flag: '✨',
      isCustom: true,
      details: l
    }));
    return [...INTERNATIONAL_LANGS, ...REGIONAL_LANGS, ...customs];
  };

  const handleAddCustomLang = () => {
    if (!newCustomLang.name.trim()) return;
    setCustomLangs([...customLangs, newCustomLang]);
    setNewCustomLang({ name: "", style: "", dialect: "", greeting: "" });
    setShowCustomForm(false);
  };

  const savePreset = () => {
    if (!presetNameInput.trim()) return;
    const newPreset = {
      id: Date.now().toString(),
      name: presetNameInput,
      primaryLang,
      secondaryLang,
      tertiaryLang,
      customLangs
    };
    setSavedPresets([...savedPresets, newPreset]);
    setPresetNameInput("");
  };

  const loadPreset = (preset) => {
    setPrimaryLang(preset.primaryLang);
    setSecondaryLang(preset.secondaryLang);
    setTertiaryLang(preset.tertiaryLang);
    if (preset.customLangs) {
      setCustomLangs(preset.customLangs);
    }
  };

  const getLanguageLabel = (langId) => {
    const combined = getCombinedLangsList();
    const found = combined.find(l => l.id === langId);
    if (found) {
      return `${found.flag} ${found.name} ${found.isCustom ? '(Custom)' : ''}`;
    }
    return langId;
  };

  const analyzeProductImage = async (base64Image) => {
    setIsAnalyzingProduct(true);
    setProductName("Sedang menganalisis...");
    setProductDesc("AI sedang mengidentifikasi detail produk...");
    try {
        const imageBase64 = base64Image.split(',')[1];
        const modeTitle = selectedMode?.title || 'Komersial';
        const systemPrompt = `
        Tugas Anda adalah menganalisis gambar produk dan mengembalikan HANYA format JSON valid.
        1. "productName": Identifikasi produk, maksimal 3 kata (Bahasa Indonesia).
        2. "productDesc": Deskripsi SANGAT RINGKAS (kategori, merek, warna/ciri menonjol).
        3. "backgrounds": Array berisi 7 rekomendasi latar/background yang eksklusif & estetik untuk sesi ${modeTitle} produk tersebut di Indonesia.
        
        Contoh output:
        {
          "productName": "Sepatu Lari Nike",
          "productDesc": "Sepatu lari Nike Air Max warna putih dan biru.",
          "backgrounds": ["Trek Lari Senayan", "Jalanan Sudirman", "Studio Putih", "Teras Cafe", "Ruang Tamu Estetik", "Track Urban", "Taman Kota"]
        }
        `;

        const payload = {
            contents: [{
                parts: [
                    { text: systemPrompt },
                    { inlineData: { mimeType: "image/png", data: imageBase64 } }
                ]
            }],
            generationConfig: { 
                responseMimeType: "application/json"
            }
        };

        const response = await fetchWithRetry(`https://generativelanguage.googleapis.com/v1beta/models/${TEXT_MODEL}:generateContent?key=${apiKey}`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(payload)
        });

        if (response.ok) {
            const result = await response.json();
            const text = result.candidates?.[0]?.content?.parts?.[0]?.text;
            if (text) {
                const data = JSON.parse(text);
                setProductName(data.productName || "Produk Terdeteksi");
                setProductDesc(data.productDesc || "Detail produk.");
                const baseBgs = selectedMode ? MODE_BACKGROUNDS[selectedMode.id] : MODE_BACKGROUNDS.ugc;
                if (data.backgrounds && Array.isArray(data.backgrounds) && data.backgrounds.length > 0) {
                    const mergedBackgrounds = Array.from(new Set([...data.backgrounds, ...baseBgs]));
                    setDynamicBackgrounds(mergedBackgrounds);
                    setConfig(prev => ({...prev, background: data.backgrounds[0] || mergedBackgrounds[0]}));
                    setBgType('preset');
                } else {
                    setDynamicBackgrounds(baseBgs);
                    setConfig(prev => ({...prev, background: baseBgs[0]}));
                }
            } else {
                setProductName("Produk Manual"); setProductDesc("Isi manual");
            }
        } else {
            setProductName("Produk Manual"); setProductDesc("Isi manual");
        }
    } catch (error) {
        console.error("Product analysis error:", error);
        setProductName("Produk Manual"); setProductDesc("Gagal memuat AI. Silakan isi manual.");
        const baseBgs = selectedMode ? MODE_BACKGROUNDS[selectedMode.id] : MODE_BACKGROUNDS.ugc;
        setDynamicBackgrounds(baseBgs);
        setConfig(prev => ({...prev, background: baseBgs[0]}));
    } finally {
        setIsAnalyzingProduct(false);
    }
  };

  const handleProductUpload = (e) => {
    playClick();
    const file = e.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onloadend = () => {
        const base64 = reader.result;
        setProductImage(base64);
        setIsProductConfirmed(false);
        analyzeProductImage(base64);
      };
      reader.readAsDataURL(file);
    }
  };

  const handleModelUpload = (e) => {
    playClick();
    const file = e.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onloadend = () => setModelImage(reader.result);
      reader.readAsDataURL(file);
    }
  };

  const modes = [
    { id: 'ugc', title: 'UGC Natural', icon: Smartphone, desc: 'Gaya kasual, otentik, direkam dengan smartphone' },
    { id: 'commercial', title: 'Commercial', icon: Package, desc: 'Kualitas studio profesional, bersih dan tajam' },
    { id: 'mirror', title: 'Mirror Check', icon: Camera, desc: 'Gaya mirror selfie, estetika OOTD di depan kaca' },
    { id: 'pov', title: 'POV Hand Review', icon: Hand, desc: 'Sudut pandang orang pertama (First-person view)' },
  ];

  const ratios = [
    { label: 'Portrait', sub: '9:16', value: '9:16' },
    { label: 'Square', sub: '1:1', value: '1:1' },
    { label: 'Landscape', sub: '16:9', value: '16:9' }
  ];

  const sceneCounts = [4, 8, 12];

  const getLanguageSystemInstructions = () => {
    const combined = getCombinedLangsList();
    const primaryDetail = combined.find(l => l.id === primaryLang);
    const secondaryDetail = combined.find(l => l.id === secondaryLang);
    const tertiaryDetail = combined.find(l => l.id === tertiaryLang);

    let instructions = `Bahasa Utama: ${primaryDetail ? primaryDetail.name : 'Indonesia'}.`;
    if (primaryDetail?.isCustom) {
       instructions += ` Gaya Bahasa: ${primaryDetail.details.style}, Karakter Dialek: ${primaryDetail.details.dialect}, Kata Sapaan: ${primaryDetail.details.greeting}.`;
    }

    if (secondaryDetail) {
       instructions += ` Bahasa Kedua (gunakan secara selang-seling atau pendukung): ${secondaryDetail.name}.`;
       if (secondaryDetail.isCustom) {
          instructions += ` Gaya Bahasa: ${secondaryDetail.details.style}, Karakter Dialek: ${secondaryDetail.details.dialect}, Kata Sapaan: ${secondaryDetail.details.greeting}.`;
       }
    }

    if (tertiaryDetail) {
       instructions += ` Bahasa Ketiga (gunakan untuk aksen tambahan): ${tertiaryDetail.name}.`;
       if (tertiaryDetail.isCustom) {
          instructions += ` Gaya Bahasa: ${tertiaryDetail.details.style}, Karakter Dialek: ${tertiaryDetail.details.dialect}, Kata Sapaan: ${tertiaryDetail.details.greeting}.`;
       }
    }

    return instructions;
  };

  const generateScripts = async () => {
      let toneInstruction = "Tone: Profesional, jernih, persuasif.";
      let narrativeArc = "Follow a logical 6-act narrative arc: context -> problem -> discovery -> interaction -> transformation -> resolution.";

      if (selectedMode?.id === 'ugc') {
          toneInstruction = "Tone: Sangat natural, ngobrol santai seperti UGC TikTok/Reels sungguhan, tidak terkesan membaca script, gunakan bahasa sehari-hari yang sangat membumi.";
          narrativeArc = "Follow a highly organic, natural TikTok/Reels style narrative: casual hook -> relatable problem -> introducing product as an everyday find -> showing real usage -> genuine reaction -> soft recommendation.";
      } else if (selectedMode?.id === 'commercial') {
          toneInstruction = "Tone: Elegan, profesional, sangat meyakinkan seperti iklan TV premium.";
      } else if (selectedMode?.id === 'mirror') {
          toneInstruction = "Tone: Stylish, trendy, fokus pada penampilan (OOTD) dan kepercayaan diri (Mirror Selfie style).";
      } else if (selectedMode?.id === 'pov') {
          toneInstruction = "Tone: Review langsung, fokus pada detail produk, praktis, dan interaktif.";
      }

      const langContext = getLanguageSystemInstructions();

      const prompt = `Generate a JSON object with a single key 'scripts' containing an array of precisely ${config.scenes} strings for a storyboard about ${productName} (${productDesc}). ${narrativeArc} IMPORTANT: ${langContext} ${toneInstruction} EXACT DURATION: Each script string MUST take exactly 6 to 8 seconds to speak. To achieve this, write STRICTLY between 18 and 25 words per string. Do not make it too short (< 15 words) or too long (> 25 words).`;

      const payload = {
        contents: [{ parts: [{ text: prompt }] }],
        generationConfig: { responseMimeType: "application/json" }
      };

      try {
          const response = await fetchWithRetry(`https://generativelanguage.googleapis.com/v1beta/models/${TEXT_MODEL}:generateContent?key=${apiKey}`, {
              method: 'POST',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify(payload)
          });
          if (response.ok) {
              const result = await response.json();
              const text = result.candidates?.[0]?.content?.parts?.[0]?.text;
              const data = JSON.parse(text);
              if (data.scripts && data.scripts.length > 0) {
                  return data.scripts;
              }
          }
      } catch (err) {
          console.error("Script gen error", err);
      }
      
      return Array.from({ length: config.scenes }).map((_, i) => `Scene ${i+1}: Narasi default untuk ${productName}.`);
  };

  const generateVisual = async () => {
    playClick(); 
    setIsGenerating(true);
    setGeneratedImages(new Array(config.scenes).fill(null)); 
    setSceneLoadings({});
    setScenePrompts({});
    setCopiedStates({});
    setView('result');

    try {
      const activeBackground = bgType === 'custom' ? (customBg || 'Studio Foto Polos') : config.background;
      const scriptsArray = await generateScripts();

      const generateSingleScene = async (index) => {
        try {
          const base64Model = modelImage ? modelImage.split(',')[1] : null;
          const scriptText = scriptsArray[index] || `Scene ${index+1}`;
          const isBRoll = (index % 3 === 2) && (index !== config.scenes - 1);
          
          let role = isBRoll ? "B-ROLL DETAIL" : `ACT ${index + 1}`;
          if (index === 0) role = "HOOK";
          if (index === config.scenes - 1) role = "CTA";

          const modeParams = getModeLogic(selectedMode?.id, index, productName, productDesc, activeBackground, config.ugcStyle);
          
          let finalSubjectPrompt = modeParams.subjectPrompt;
          let finalNegPrompt = modeParams.negPrompt;
          let diffusionPrompt = "";
          let videoMotion = modeParams.motionCamera;
          let videoCamera = modeParams.cameraLogic;

          if (isBRoll) {
              if (selectedMode?.id === 'ugc') {
                  finalSubjectPrompt = `RAW LIFESTYLE B-ROLL PHOTO. Casual close-up of ${productName} (${productDesc}) held casually by hands or resting on a surface. Shot on iPhone 15 Pro Max. Background: ${activeBackground}. Single harsh directional light (like direct sunlight) creating strong distinct shadows. CRITICAL: Real human skin texture on hands, visible pores. NO FACES. Unedited, raw, candid.`;
                  finalNegPrompt = "Human face, eyes, mouth, floating product, overly stylized studio lighting, flat lighting, artificial look, 3d render, DSLR, softbox, professional macro, smooth plastic skin.";
                  videoMotion = "Slight natural handheld camera shake, showcasing the product.";
                  videoCamera = "Smartphone camera, close up.";
              } else if (selectedMode?.id === 'commercial') {
                  finalSubjectPrompt = `PREMIUM STUDIO B-ROLL. Masterpiece macro close-up of ${productName} (${productDesc}). Focus on premium texture and material details. Background: ${activeBackground}. Dramatic commercial lighting with a single harsh spotlight, distinct shadows, elegant reflections, 8k resolution. NO HUMAN FACES.`;
                  finalNegPrompt = "Human face, eyes, mouth, messy background, low quality, flat lighting, grainy, amateur, everyday lighting, blurry.";
                  videoMotion = "Smooth macro pan/slide over the product surface.";
                  videoCamera = "Macro Lens, sharp focus, high-end commercial style.";
              } else if (selectedMode?.id === 'mirror') {
                  finalSubjectPrompt = `RAW MIRROR SELFIE B-ROLL. Close-up of ${productName} (${productDesc}) reflected in a mirror. Held casually. Background: ${activeBackground}. Harsh directional room light creating distinct shadows, highly detailed textures. NO FACES.`;
                  finalNegPrompt = "Human face, flat lighting, lack of shadows, overexposed, clean commercial look, 3d render.";
                  videoMotion = "Slight handheld shake, focusing on the product in the mirror.";
                  videoCamera = "Smartphone camera, mirror reflection close up.";
              }
          }

          let globalNeg = ", text, words, letters, typography, watermark, signature, logo, brand name, subtitles, captions, text overlays, UI elements, illustration, painting, cartoon, anime, 3d render, CGI, unreal engine, deformed, bad anatomy, distorted, different face, morphed body, flat lighting, soft ambient light, lack of shadows, AI generated look, synthetic, plastic, generic AI face, overly smooth, midjourney style, washed out";
          if (['ugc', 'pov', 'mirror'].includes(selectedMode?.id)) {
              globalNeg += ", flawless perfect lighting, professional photography, softbox, studio flash, plastic skin";
          }
          finalNegPrompt += globalNeg;
          
          let rolePrefix = "PHOTOGRAPHY MASTERCLASS TASK:";
          let styling = "Hyper-realistic color processing. Single harsh directional light source creating deep, hard shadows and bold 3D dimensionality. ABSOLUTELY NO TEXT OR WATERMARKS.";
          let realismLaws = "MANDATORY: Must look like a real, actual photograph. Ultra-detailed, lifelike. NOT AI GENERATED. Include natural photographic imperfections and high-contrast depth. STRICTLY NO TEXT, NO UI, NO WATERMARKS ALLOWED IN THE IMAGE.";

          if (['ugc', 'pov', 'mirror'].includes(selectedMode?.id)) {
              rolePrefix = "RAW SMARTPHONE PHOTO TASK:";
              styling = "Authentic raw aesthetic. Single harsh directional light (like direct sunlight or hard smartphone flash) casting distinct, hard shadows for realistic depth. Shot on iPhone. ABSOLUTELY NO TEXT OR WATERMARKS.";
              realismLaws = "AUTHENTIC UGC REALISM: Must look EXACTLY like an unedited iPhone photo posted directly to social media. ZERO AI VIBE. CRITICAL: Highly detailed real human skin texture, visible pores, natural imperfections, distinct harsh lighting contrast. NO soft studio lights, NO beauty filters. STRICTLY NO TEXT, NO UI, NO WATERMARKS ALLOWED IN THE IMAGE.";
          } else if (selectedMode?.id === 'commercial') {
              rolePrefix = "HIGH-END COMMERCIAL PHOTOGRAPHY TASK:";
              styling = "Premium catalog aesthetic, razor-sharp details. Single harsh spotlight creating dramatic, deep shadows and striking 3D dimensionality. ABSOLUTELY NO TEXT OR WATERMARKS.";
          }

          const base64ProductRaw = productImage ? productImage.split(',')[1] : null;
          const hasModelActive = base64Model && !isBRoll && selectedMode?.id !== 'pov';
          
          let refInfo = "";
          let payloadParts = [];
          
          if (hasModelActive) {
              refInfo = "[CRITICAL] REFERENCE IMAGES: Image 1 = Base Model (Clone this exact face, hair, and body shape). Image 2 = Product Reference. ";
              payloadParts.push({ inlineData: { mimeType: "image/png", data: base64Model } });
              if (base64ProductRaw) payloadParts.push({ inlineData: { mimeType: "image/png", data: base64ProductRaw } });
          } else if (base64ProductRaw) {
              refInfo = "[CRITICAL] REFERENCE IMAGE: Image 1 = The EXACT Product. ";
              payloadParts.push({ inlineData: { mimeType: "image/png", data: base64ProductRaw } });
          }

          const currentIdentityLock = hasModelActive ? IDENTITY_LOCK : "";
          finalNegPrompt += ", different face, altered identity, unrecognizable person, random face";

          diffusionPrompt = `${refInfo}${rolePrefix} Scene ${index + 1}. ${scriptText}. ${styling} ${modeParams.cameraLogic}. ${finalSubjectPrompt} ENVIRONMENT: ${activeBackground}. ${realismLaws} ${currentIdentityLock} NEGATIVE: ${finalNegPrompt} Technical: ${config.ratio} format.`;

          payloadParts.push({ text: diffusionPrompt });

          const payload = {
            contents: [{ parts: payloadParts }],
            generationConfig: { responseModalities: ["IMAGE"], aspectRatio: config.ratio }
          };

          const response = await fetchWithRetry(`https://generativelanguage.googleapis.com/v1beta/models/${GENERATIVE_MODEL}:generateContent?key=${apiKey}`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(payload)
          });

          if (!response || !response.ok) {
              const errorText = response ? await response.text() : 'No response';
              throw new Error(`Generation failed: ${errorText}`);
          }

          const result = await response.json();
          const base64Output = result.candidates?.[0]?.content?.parts?.find(p => p.inlineData)?.inlineData?.data;

          if (base64Output) {
            const imgSrc = `data:image/png;base64,${base64Output}`;
            const defaultVoActive = !isBRoll && (selectedMode?.id !== 'pov');
            
            const scenarioData = { 
                src: imgSrc, 
                role, 
                action: finalSubjectPrompt, 
                scriptHint: scriptText,
                videoMotion,
                videoCamera,
                activeBackground,
                isBRoll,
                voActive: defaultVoActive,
                voText: scriptText
            };
            
            setGeneratedImages(prev => {
               const newArr = [...prev];
               const updated = new Array(config.scenes).fill(null).map((_, i) => {
                  if (prev[i]) return prev[i];
                  if (i === index) return scenarioData;
                  return null;
               });
               return updated.map((item, i) => item || prev[i]);
            });
          }
        } catch (err) {
          console.error(`Scene ${index} failed:`, err);
        } 
      };

      for (let i = 0; i < config.scenes; i++) {
        await generateSingleScene(i);
      }

    } catch (err) {
      console.error(err);
    } finally {
      setIsGenerating(false);
    }
  };

  const regenerateSingleScene = async (index) => {
    playClick();
    const revisionInstruction = scenePrompts[index] || "";
    const originalScenario = generatedImages[index]; 
    const activeBackground = bgType === 'custom' ? (customBg || 'Studio Foto Polos') : config.background;
    
    setSceneLoadings(prev => ({ ...prev, [index]: true }));

    try {
      const base64Model = modelImage ? modelImage.split(',')[1] : null;
      const base64ProductRaw = productImage ? productImage.split(',')[1] : null;

      const hasModelActive = base64Model && !originalScenario.isBRoll && selectedMode?.id !== 'pov';
      const currentIdentityLock = hasModelActive ? IDENTITY_LOCK : "";

      let revisionNegPrompt = "different face, altered identity, unrecognizable person, random face, morphed body, text, words, letters, typography, watermark, signature, logo, brand name, subtitles, captions, text overlays, UI elements, illustration, painting, cartoon, 3d render, CGI, unreal engine, deformed, distorted, flat lighting, soft ambient light, lack of shadows, AI generated look, synthetic, plastic, generic AI face, overly smooth, midjourney style";
      if (['ugc', 'pov', 'mirror'].includes(selectedMode?.id)) {
          revisionNegPrompt += ", flawless perfect lighting, cinematic movie shot, professional photography, DSLR, 8k, high res, softbox, studio flash, plastic skin, beauty filter";
      }

      let rolePrefix = "PHOTOGRAPHY MASTERCLASS TASK:";
      let realismLaws = "MANDATORY: Must look like a real, actual photograph. Ultra-detailed, lifelike. NOT AI GENERATED. Include natural photographic imperfections and high-contrast depth with single harsh directional lighting. STRICTLY NO TEXT, NO UI, NO WATERMARKS ALLOWED IN THE IMAGE.";
      if (['ugc', 'pov', 'mirror'].includes(selectedMode?.id)) {
          rolePrefix = "RAW SMARTPHONE PHOTO TASK:";
          realismLaws = "AUTHENTIC UGC REALISM: Must look EXACTLY like an unedited iPhone photo. ZERO AI VIBE. CRITICAL: Highly detailed real human skin texture, visible pores. Single harsh directional light (like direct sunlight or hard flash) casting distinct, hard shadows for realistic depth. STRICTLY NO TEXT, NO UI, NO WATERMARKS ALLOWED IN THE IMAGE.";
      } else if (selectedMode?.id === 'commercial') {
          rolePrefix = "HIGH-END COMMERCIAL PHOTOGRAPHY TASK:";
          realismLaws = "MANDATORY: Premium catalog aesthetic. Single harsh spotlight creating dramatic, deep shadows and striking 3D dimensionality. STRICTLY NO TEXT, NO UI, NO WATERMARKS ALLOWED IN THE IMAGE.";
      }

      let refInfo = "";
      let payloadParts = [];
      
      if (hasModelActive) {
          refInfo = "[CRITICAL] REFERENCE IMAGES: Image 1 = Base Model (Clone this exact face, hair, and body shape). Image 2 = Product Reference. ";
          payloadParts.push({ inlineData: { mimeType: "image/png", data: base64Model } });
          if (base64ProductRaw) payloadParts.push({ inlineData: { mimeType: "image/png", data: base64ProductRaw } });
      } else if (base64ProductRaw) {
          refInfo = "[CRITICAL] REFERENCE IMAGE: Image 1 = The EXACT Product. ";
          payloadParts.push({ inlineData: { mimeType: "image/png", data: base64ProductRaw } });
      }

      const systemPrompt = `
      ${refInfo}
      ${rolePrefix} - REVISION MODE.
      Target: Regenerate Scene ${index + 1} (${originalScenario.role}).
      Product: ${productName} (${productDesc})
      User Instruction: ${revisionInstruction || "Improve lighting and composition."}
      Keep original action: ${originalScenario.action}. Background: ${activeBackground}.
      ${realismLaws}
      ${currentIdentityLock}
      CRITICAL NEGATIVE: ${revisionNegPrompt}.
      `;

      payloadParts.push({ text: systemPrompt });

      const payload = {
        contents: [{ parts: payloadParts }],
        generationConfig: { responseModalities: ["IMAGE"], aspectRatio: config.ratio }
      };

      const response = await fetchWithRetry(`https://generativelanguage.googleapis.com/v1beta/models/${GENERATIVE_MODEL}:generateContent?key=${apiKey}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });

      if (!response || !response.ok) {
          const errorText = response ? await response.text() : 'No response';
          throw new Error(`Generation failed: ${errorText}`);
      }

      const result = await response.json();
      const base64Output = result.candidates?.[0]?.content?.parts?.find(p => p.inlineData)?.inlineData?.data;

      if (base64Output) {
        const imgSrc = `data:image/png;base64,${base64Output}`;
        setGeneratedImages(prev => {
          const newArr = [...prev];
          newArr[index] = { ...newArr[index], src: imgSrc }; 
          return newArr;
        });
        setScenePrompts(prev => ({ ...prev, [index]: "" }));
      }
    } catch (err) {
      console.error("Scene regeneration error:", err);
    } finally {
      setSceneLoadings(prev => ({ ...prev, [index]: false }));
    }
  };

  const downloadImage = async (imgSrc, index, e) => {
    if (e) {
      e.preventDefault();
      e.stopPropagation();
    }
    playClick(); 
    if (!imgSrc) return;
    
    const croppedSrc = await processImageForDownload(imgSrc, config.ratio);
    
    try {
      const res = await fetch(croppedSrc);
      const blob = await res.blob();
      const blobUrl = window.URL.createObjectURL(blob);
      
      const link = document.createElement('a');
      link.href = blobUrl;
      link.download = `mata-studio-scene-${index + 1}-${Date.now()}.png`;
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
      
      setTimeout(() => window.URL.revokeObjectURL(blobUrl), 1000);
    } catch (err) {
      console.error("Download failed, using fallback:", err);
      const link = document.createElement('a');
      link.href = croppedSrc;
      link.download = `mata-studio-scene-${index + 1}-${Date.now()}.png`;
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    }
  };

  const handleDownloadAllZip = async () => {
      playClick();
      setIsDownloadingZip(true);
      try {
          let JSZip = window.JSZip;
          if (!JSZip) {
              await new Promise((resolve, reject) => {
                  const script = document.createElement('script');
                  script.src = 'https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js';
                  script.onload = () => resolve();
                  script.onerror = reject;
                  document.head.appendChild(script);
              });
              JSZip = window.JSZip;
          }

          const zip = new JSZip();

          for (let i = 0; i < generatedImages.length; i++) {
              const sceneData = generatedImages[i];
              if (!sceneData || !sceneData.src) continue;

              const folderName = `Scene_0${i + 1}`;
              const folder = zip.folder(folderName);

              const croppedBase64DataUrl = await processImageForDownload(sceneData.src, config.ratio);
              const base64Data = croppedBase64DataUrl.split(',')[1];
              folder.file(`${folderName}_Image.png`, base64Data, { base64: true });

              const promptText = getPromptForScene(sceneData);
              folder.file(`${folderName}_Prompt.txt`, promptText);
          }

          const content = await zip.generateAsync({ type: 'blob' });
          const blobUrl = window.URL.createObjectURL(content);
          const link = document.createElement('a');
          link.href = blobUrl;
          link.download = `mata_studio_semua_scene_${Date.now()}.zip`;
          document.body.appendChild(link);
          link.click();
          document.body.removeChild(link);
          setTimeout(() => window.URL.revokeObjectURL(blobUrl), 1000);
      } catch (err) {
          console.error("Failed to generate ZIP", err);
      } finally {
          setIsDownloadingZip(false);
      }
  };

  const handleGenerateVOScript = async () => {
      playClick();
      setIsGeneratingVOScript(true);
      
      const langContext = getLanguageSystemInstructions();
      const prompt = `Buatkan naskah master voiceover gaya ${selectedMode?.title || 'Komersial'} untuk ${productName || 'Produk'}. Durasi: ${voDuration}. ${langContext} Tuliskan naskahnya saja langsung tanpa tambahan judul, markdown, atau penjelasan.`;
      
      const payload = {
        contents: [{ parts: [{ text: prompt }] }],
      };

      try {
          const response = await fetchWithRetry(`https://generativelanguage.googleapis.com/v1beta/models/${TEXT_MODEL}:generateContent?key=${apiKey}`, {
              method: 'POST',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify(payload)
          });
          if (response && response.ok) {
              const result = await response.json();
              const text = result.candidates?.[0]?.content?.parts?.[0]?.text;
              if (text) setVoScript(text.trim());
          }
      } catch (err) {
          console.error("Master VO Script gen error", err);
      } finally {
          setIsGeneratingVOScript(false);
      }
  };

  const handleGenerateVOAudio = async () => {
      if (!voScript) return;
      playClick();
      setIsGeneratingVOAudio(true);
      setVoAudioUrl(null);

      const payload = {
        contents: [{
            parts: [{ text: `Bacakan dengan nada ${voTone}: ${voScript}` }]
        }],
        generationConfig: {
            responseModalities: ["AUDIO"],
            speechConfig: {
                voiceConfig: {
                    prebuiltVoiceConfig: {
                        voiceName: voVoice
                    }
                }
            }
        }
      };

      try {
          const response = await fetchWithRetry(`https://generativelanguage.googleapis.com/v1beta/models/${TTS_MODEL}:generateContent?key=${apiKey}`, {
              method: 'POST',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify(payload)
          });

          if (response && response.ok) {
              const result = await response.json();
              const base64PCM = result.candidates?.[0]?.content?.parts?.[0]?.inlineData?.data;
              if (base64PCM) {
                  const url = base64ToWavUrl(base64PCM);
                  setVoAudioUrl(url);
              }
          }
      } catch (err) {
          console.error("Audio generation failed:", err);
      } finally {
          setIsGeneratingVOAudio(false);
      }
  };

  const renderContent = () => {
    if (isRestoring) {
      return (
        <LayoutWrapper>
          <div className="flex-grow flex items-center justify-center min-h-screen">
            <Loader2 className="w-8 h-8 animate-spin text-emerald-500" />
            <span className="ml-3.5 text-zinc-400 font-bold tracking-widest uppercase text-xs">Memulihkan Sesi...</span>
          </div>
        </LayoutWrapper>
      );
    }

    switch (view) {
      case 'login':
        return (
          <LayoutWrapper>
            <div className="flex-grow flex flex-col items-center justify-center text-center relative w-full h-full min-h-screen px-4 overflow-hidden">
               <div className="relative z-10 flex flex-col items-center p-8 md:p-10 rounded-2xl bg-[#18181b]/60 backdrop-blur-md border border-zinc-800 shadow-xl max-w-sm w-full animate-in fade-in zoom-in-95 duration-500">
                  <div className="w-14 h-14 bg-gradient-to-br from-emerald-600 to-teal-500 rounded-xl flex items-center justify-center border border-emerald-400/20 shadow-lg mb-6">
                      <Eye className="w-6.5 h-6.5 text-white" />
                  </div>
                  
                  <h1 className="text-xl font-bold tracking-tight text-white uppercase mb-6">
                     V1 PRO MATA STUDIO AI
                  </h1>
                  
                  <div className="w-full flex bg-[#121214] rounded-lg p-1 border border-zinc-800 mb-5">
                     <button 
                       onClick={() => { playClick(); setLoginRole('user'); setLoginError(''); setLoginInput(''); }} 
                       className={`flex-1 py-2 rounded-md text-[10px] font-bold uppercase tracking-wider transition-all duration-200 flex items-center justify-center gap-1.5 ${loginRole === 'user' ? 'bg-zinc-800 text-white shadow-sm' : 'text-zinc-500 hover:text-zinc-300'}`}
                     >
                       <KeyRound className="w-3.5 h-3.5" /> User Token
                     </button>
                     <button 
                       onClick={() => { playClick(); setLoginRole('admin'); setLoginError(''); setLoginInput(''); }} 
                       className={`flex-1 py-2 rounded-md text-[10px] font-bold uppercase tracking-wider transition-all duration-200 flex items-center justify-center gap-1.5 ${loginRole === 'admin' ? 'bg-zinc-800 text-white shadow-sm' : 'text-zinc-500 hover:text-zinc-300'}`}
                     >
                       <ShieldCheck className="w-3.5 h-3.5" /> Admin
                     </button>
                  </div>

                  <div className="w-full space-y-4">
                     <div className="relative">
                         <input 
                           type={loginRole === 'admin' ? 'password' : 'text'}
                           placeholder={loginRole === 'admin' ? "Masukkan Password Admin" : "Masukkan Token Akses"}
                           value={loginInput}
                           onChange={(e) => setLoginInput(e.target.value)}
                           onKeyDown={(e) => e.key === 'Enter' && handleLogin()}
                           className="w-full bg-[#121214] border border-zinc-800 rounded-lg p-3.5 text-center text-xs text-white placeholder:text-zinc-650 focus:outline-none focus:border-zinc-700 transition-all uppercase tracking-widest font-mono"
                         />
                     </div>

                     {loginError && <p className="text-amber-500 text-[10px] font-bold uppercase tracking-wider">{loginError}</p>}
                     
                     <button 
                       onClick={handleLogin}
                       className="w-full py-3 bg-emerald-500 hover:bg-emerald-400 text-neutral-950 rounded-lg font-bold text-xs tracking-widest uppercase transition-all duration-200 flex items-center justify-center gap-1.5 mt-1"
                     >
                       Masuk ke Studio <ArrowRight className="w-4 h-4" />
                     </button>
                  </div>
               </div>
            </div>
          </LayoutWrapper>
        );

      case 'admin-dashboard':
        return (
          <LayoutWrapper>
            <Navbar onBack={handleLogout} authMode={authMode} />
            <div className="flex-grow flex flex-col px-6 pt-32 pb-12 md:px-12 md:pt-40 lg:px-24">
              <div className="max-w-4xl mx-auto w-full">
                <SectionTitle title="Admin Dashboard" subtitle="Kelola Token Akses agar pengguna dapat menggunakan MATA STUDIO.AI." />
                
                <div className="mt-8 bg-[#18181b]/60 border border-zinc-800 rounded-2xl p-6 md:p-8 backdrop-blur-md space-y-6 animate-in fade-in slide-in-from-bottom-5">
                   <div className="flex flex-col md:flex-row md:items-center justify-between gap-4">
                      <div>
                          <h3 className="text-base font-bold text-white tracking-tight">Token Akses Aktif</h3>
                          <p className="text-xs text-zinc-500 font-normal mt-1">Daftar token yang dapat digunakan user untuk login.</p>
                      </div>
                      <div className="flex items-center gap-2">
                          <input 
                              type="text" 
                              placeholder="Ketik token..." 
                              value={customTokenInput}
                              onChange={(e) => setCustomTokenInput(e.target.value.toUpperCase())}
                              onKeyDown={(e) => {
                                  if (e.key === 'Enter') {
                                      playClick();
                                      if(customTokenInput.trim() !== '' && !validTokens.includes(customTokenInput.trim())) {
                                          setValidTokens([...validTokens, customTokenInput.trim()]);
                                          setCustomTokenInput('');
                                      }
                                  }
                              }}
                              className="bg-[#121214] border border-zinc-800 rounded-lg px-4 py-2.5 text-xs text-white placeholder:text-zinc-655 focus:outline-none focus:border-zinc-700 transition-all font-mono uppercase tracking-widest w-44 md:w-auto"
                          />
                          <button 
                            onClick={() => {
                              playClick();
                              if(customTokenInput.trim() !== '' && !validTokens.includes(customTokenInput.trim())) {
                                  setValidTokens([...validTokens, customTokenInput.trim()]);
                                  setCustomTokenInput('');
                              }
                            }} 
                            className="px-5 py-2.5 bg-zinc-900 border border-zinc-800 hover:bg-zinc-800 text-white rounded-lg text-xs font-bold uppercase tracking-wider flex items-center justify-center gap-1.5 transition-all whitespace-nowrap"
                          >
                             <Sparkles className="w-4 h-4 text-emerald-400" /> Tambah
                          </button>
                      </div>
                   </div>
                   
                   <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-3.5">
                      {validTokens.map((token, idx) => (
                         <div key={idx} className="bg-[#121214] border border-zinc-800/80 p-4 rounded-xl flex justify-between items-center hover:border-zinc-700 transition-colors">
                            <span className="font-mono text-zinc-300 tracking-wider text-sm font-semibold">{token}</span>
                            <div className="flex items-center gap-1.5">
                                <button 
                                    onClick={() => { playClick(); safeCopyToClipboard(token); }} 
                                    className="w-7 h-7 rounded-lg bg-zinc-900 border border-zinc-850 flex items-center justify-center text-zinc-400 hover:text-white hover:bg-zinc-800 transition-all"
                                    title="Copy Token"
                                >
                                    <Copy className="w-3.5 h-3.5"/>
                                </button>
                                <button 
                                    onClick={() => { 
                                        playClick(); 
                                        setValidTokens(validTokens.filter(t => t !== token)); 
                                    }} 
                                    className="w-7 h-7 rounded-lg bg-zinc-900/40 border border-zinc-850 flex items-center justify-center text-zinc-500 hover:text-amber-500 hover:bg-zinc-800/60 transition-all"
                                    title="Hapus Token"
                                >
                                    <X className="w-3.5 h-3.5"/>
                                </button>
                            </div>
                         </div>
                      ))}
                      {validTokens.length === 0 && <p className="text-zinc-500 text-xs italic">Belum ada token yang dibuat.</p>}
                   </div>

                   <div className="pt-6 border-t border-zinc-800 flex justify-end">
                      <button 
                        onClick={() => { playClick(); setView('mode-selection'); }} 
                        className="w-full md:w-auto px-6 py-3 bg-emerald-500 hover:bg-emerald-400 text-neutral-950 rounded-lg font-bold text-xs tracking-widest uppercase transition-all flex items-center justify-center gap-2"
                      >
                         Masuk ke Studio (Admin Test) <ArrowRight className="w-4 h-4" />
                      </button>
                   </div>
                </div>
              </div>
            </div>
          </LayoutWrapper>
        );

      case 'mode-selection':
        return (
          <LayoutWrapper>
            <Navbar onBack={handleLogout} onReset={handleResetAndNew} authMode={authMode} />
            <div className="flex-grow flex flex-col justify-center px-6 pt-32 pb-12 md:px-12 md:pt-40 lg:px-24">
              <div className="max-w-6xl mx-auto w-full">
                <div className="flex flex-col md:flex-row md:items-end justify-between mb-10 gap-4 border-b border-zinc-900 pb-6">
                  <SectionTitle title="Pilih Mode Visual" subtitle="Tentukan gaya estetika kampanye visual Anda." />
                  <div className="hidden md:flex items-center gap-2 text-zinc-500">
                    <span className="text-[10px] font-bold tracking-widest uppercase text-emerald-400">01</span>
                    <div className="w-12 h-px bg-zinc-800"></div>
                    <span className="text-[10px] font-bold tracking-widest uppercase opacity-40">04</span>
                  </div>
                </div>
                <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 animate-in fade-in slide-in-from-bottom-5 duration-500">
                  {modes.map((mode, idx) => {
                    const CurrentIcon = mode.icon;
                    return (
                      <button 
                        key={mode.id} 
                        type="button"
                        onClick={() => { 
                          playClick(); 
                          setSelectedMode(mode); 
                          setDynamicBackgrounds(MODE_BACKGROUNDS[mode.id]);
                          setConfig(prev => ({
                              ...prev, 
                              background: MODE_BACKGROUNDS[mode.id][0]
                          }));
                          setView('customize-language'); 
                        }} 
                        className="group relative flex flex-col justify-between h-64 p-6 rounded-2xl bg-[#18181b]/60 border border-zinc-800 hover:border-zinc-700 transition-all duration-300 overflow-hidden text-left hover:bg-[#18181b]"
                      >
                        <div className="relative z-10 flex flex-col h-full justify-between">
                          <div className="flex justify-between items-start w-full">
                            <div className="w-11 h-11 rounded-xl border border-zinc-800 bg-[#121214] flex items-center justify-center text-zinc-400 group-hover:text-emerald-400 group-hover:border-zinc-700 transition-all duration-300">
                              <CurrentIcon className="w-5 h-5" />
                            </div>
                            <span className="text-[10px] font-mono text-zinc-600 group-hover:text-zinc-400 transition-colors">0{idx + 1}</span>
                          </div>
                          <div className="space-y-3">
                            <div>
                              <h3 className="text-base font-bold text-white mb-1.5 tracking-tight group-hover:text-emerald-400 transition-colors">{mode.title}</h3>
                              <p className="text-xs text-zinc-400 font-normal leading-relaxed group-hover:text-zinc-300 transition-colors line-clamp-2">{mode.desc}</p>
                            </div>
                            <div className="flex items-center justify-between pt-4 border-t border-zinc-800 group-hover:border-zinc-700">
                              <span className="text-[9px] font-bold tracking-widest uppercase text-zinc-500 group-hover:text-emerald-400 transition-colors">Pilih Mode</span>
                              <div className="w-7 h-7 rounded-lg border border-zinc-800 flex items-center justify-center bg-zinc-900 group-hover:bg-emerald-500 group-hover:text-neutral-950 group-hover:border-transparent transition-all duration-300 text-zinc-400">
                                 <ArrowRight className="w-3.5 h-3.5 -rotate-45 group-hover:rotate-0 transition-transform duration-300" />
                              </div>
                            </div>
                          </div>
                        </div>
                      </button>
                    );
                  })}
                </div>
              </div>
            </div>
          </LayoutWrapper>
        );

      case 'customize-language':
        return (
          <LayoutWrapper>
            <Navbar onBack={() => setView('mode-selection')} onReset={handleResetAndNew} authMode={authMode} />
            <div className="flex-grow flex flex-col px-6 pt-32 pb-12 md:px-12 md:pt-40 lg:px-24">
               <div className="max-w-5xl mx-auto w-full flex flex-col gap-8">
                  <SectionTitle 
                    title="🌍 PILIH BAHASA" 
                    subtitle={`Konfigurasi bahasa utama, logat daerah, atau bahasa kustom menggunakan sistem filter Dropdown Searchable untuk gaya kampanye ${selectedMode?.title}.`} 
                  />
                  
                  <div className="grid grid-cols-1 lg:grid-cols-12 gap-8 animate-in fade-in slide-in-from-bottom-5 duration-500">
                    <div className="lg:col-span-7 space-y-6">
                      
                      {/* Searchable Language Dropdown Components */}
                      <div className="bg-[#18181b]/60 border border-zinc-850 rounded-2xl p-6 space-y-5 shadow-xl">
                        <SearchableLanguageDropdown 
                          label="✓ Bahasa Utama" 
                          selectedId={primaryLang} 
                          onChange={(val) => { if(val) setPrimaryLang(val); }} 
                          placeholder="Pilih Bahasa Utama..." 
                          languages={getCombinedLangsList()} 
                          isOptional={false}
                        />

                        <SearchableLanguageDropdown 
                          label="✓ Bahasa Kedua (Opsional)" 
                          selectedId={secondaryLang} 
                          onChange={(val) => setSecondaryLang(val)} 
                          placeholder="Pilih Bahasa Kedua..." 
                          languages={getCombinedLangsList().filter(l => l.id !== primaryLang)} 
                          isOptional={true}
                        />

                        <SearchableLanguageDropdown 
                          label="✓ Bahasa Ketiga (Opsional)" 
                          selectedId={tertiaryLang} 
                          onChange={(val) => setTertiaryLang(val)} 
                          placeholder="Pilih Bahasa Ketiga..." 
                          languages={getCombinedLangsList().filter(l => l.id !== primaryLang && l.id !== secondaryLang)} 
                          isOptional={true}
                        />
                      </div>

                      {/* Custom Language Form Container */}
                      <div className="bg-[#18181b]/40 border border-zinc-850 rounded-2xl p-5 space-y-4">
                        <button
                          type="button"
                          onClick={() => { playClick(); setShowCustomForm(!showCustomForm); }}
                          className="w-full flex items-center justify-between text-xs font-bold uppercase tracking-wider text-zinc-300 hover:text-white transition-colors"
                        >
                          <span className="flex items-center gap-2">✨ Tambah Bahasa Custom</span>
                          <span className="text-emerald-400 font-extrabold text-base">{showCustomForm ? '—' : '+'}</span>
                        </button>

                        {showCustomForm && (
                          <div className="pt-2 space-y-3 animate-in fade-in duration-200">
                            <div className="grid grid-cols-1 sm:grid-cols-2 gap-3.5">
                              <div className="space-y-1.5">
                                <label className="text-[10px] text-zinc-550 uppercase tracking-widest font-bold">Nama Bahasa</label>
                                <input 
                                  type="text"
                                  placeholder="Cth: Sunda Lembut"
                                  value={newCustomLang.name}
                                  onChange={(e) => setNewCustomLang({...newCustomLang, name: e.target.value})}
                                  className="w-full bg-[#121214] border border-zinc-800 rounded-lg p-2.5 text-xs text-white placeholder:text-zinc-650 focus:outline-none focus:border-zinc-700"
                                />
                              </div>
                              <div className="space-y-1.5">
                                <label className="text-[10px] text-zinc-550 uppercase tracking-widest font-bold">Gaya Bahasa</label>
                                <input 
                                  type="text"
                                  placeholder="Cth: Kasual & Halus"
                                  value={newCustomLang.style}
                                  onChange={(e) => setNewCustomLang({...newCustomLang, style: e.target.value})}
                                  className="w-full bg-[#121214] border border-zinc-800 rounded-lg p-2.5 text-xs text-white placeholder:text-zinc-655 focus:outline-none focus:border-zinc-700"
                                />
                              </div>
                            </div>

                            <div className="grid grid-cols-1 sm:grid-cols-2 gap-3.5">
                              <div className="space-y-1.5">
                                <label className="text-[10px] text-zinc-550 uppercase tracking-widest font-bold">Karakter Dialek</label>
                                <input 
                                  type="text"
                                  placeholder="Cth: Ayunan lembut di akhir kalimat"
                                  value={newCustomLang.dialect}
                                  onChange={(e) => setNewCustomLang({...newCustomLang, dialect: e.target.value})}
                                  className="w-full bg-[#121214] border border-zinc-800 rounded-lg p-2.5 text-xs text-white placeholder:text-zinc-655 focus:outline-none focus:border-zinc-700"
                                />
                              </div>
                              <div className="space-y-1.5">
                                <label className="text-[10px] text-zinc-550 uppercase tracking-widest font-bold">Kata Sapaan</label>
                                <input 
                                  type="text"
                                  placeholder="Cth: Sampurasun, Akang, Teteh"
                                  value={newCustomLang.greeting}
                                  onChange={(e) => setNewCustomLang({...newCustomLang, greeting: e.target.value})}
                                  className="w-full bg-[#121214] border border-zinc-800 rounded-lg p-2.5 text-xs text-white placeholder:text-zinc-655 focus:outline-none focus:border-zinc-700"
                                />
                              </div>
                            </div>

                            <button
                              type="button"
                              onClick={() => { playClick(); handleAddCustomLang(); }}
                              disabled={!newCustomLang.name}
                              className="w-full py-2.5 bg-emerald-500 hover:bg-emerald-400 text-neutral-950 rounded-lg text-xs font-bold uppercase tracking-widest transition-all disabled:opacity-20 disabled:cursor-not-allowed"
                            >
                              Tambah ke Koleksi Bahasa
                            </button>
                          </div>
                        )}
                      </div>
                    </div>

                    {/* Left Hand Presets Management Panel */}
                    <div className="lg:col-span-5 space-y-6">
                      <div className="bg-[#18181b]/60 border border-zinc-800 rounded-2xl p-6 space-y-5 shadow-xl">
                        
                        <div className="space-y-3">
                          <span className="text-[10px] font-bold uppercase tracking-wider text-zinc-400 block">Preset Konfigurasi</span>
                          
                          <div className="flex gap-2">
                            <input 
                              type="text"
                              placeholder="Nama Preset..."
                              value={presetNameInput}
                              onChange={(e) => setPresetNameInput(e.target.value)}
                              className="flex-grow bg-[#121214] border border-zinc-800 rounded-lg px-3 py-2 text-xs text-white placeholder:text-zinc-655 focus:outline-none focus:border-zinc-700"
                            />
                            <button 
                              type="button"
                              onClick={() => { playClick(); savePreset(); }}
                              disabled={!presetNameInput.trim()}
                              className="px-4 py-2 bg-zinc-900 border border-zinc-800 text-[10px] font-bold uppercase text-emerald-400 rounded-lg hover:bg-zinc-800 transition-colors disabled:opacity-20 whitespace-nowrap"
                            >
                              Simpan
                            </button>
                          </div>

                          {savedPresets.length > 0 && (
                            <div className="space-y-1.5 pt-2">
                              <label className="text-[9px] text-zinc-550 font-bold uppercase">Muat Preset Terdaftar</label>
                              <div className="grid grid-cols-1 gap-2 max-h-[100px] overflow-y-auto custom-scrollbar pr-0.5">
                                {savedPresets.map((preset) => (
                                  <button
                                    key={preset.id}
                                    type="button"
                                    onClick={() => { playClick(); loadPreset(preset); }}
                                    className="px-3 py-2 text-left bg-zinc-900/60 border border-zinc-850 hover:border-zinc-750 rounded-lg text-[10px] font-semibold text-zinc-300 flex items-center justify-between"
                                  >
                                    <span className="truncate">{preset.name}</span>
                                    <span className="text-emerald-400 font-bold ml-1">✓</span>
                                  </button>
                                ))}
                              </div>
                            </div>
                          )}
                        </div>

                        <div className="pt-4 border-t border-zinc-800">
                          <ButtonCTA onClick={() => { playClick(); setView('upload-setup'); }}>
                            Simpan & Lanjutkan
                          </ButtonCTA>
                        </div>
                      </div>
                    </div>
                  </div>
               </div>
            </div>
          </LayoutWrapper>
        );

      case 'upload-setup':
        const isUploadComplete = (selectedMode?.id === 'pov' ? true : modelImage) && productImage;
        
        return (
          <LayoutWrapper>
            <Navbar onBack={() => setView('customize-language')} onReset={handleResetAndNew} authMode={authMode} />
            <div className="flex-grow flex flex-col px-6 pt-32 pb-12 md:px-12 md:pt-40 lg:px-24">
               <div className="max-w-2xl mx-auto w-full flex flex-col gap-8">
                  <SectionTitle title="Aset Visual" subtitle={`Siapkan aset gambar untuk gaya ${selectedMode?.title}.`} />
                  
                  <div className={`grid gap-5 ${selectedMode?.id === 'pov' ? 'grid-cols-1 max-w-xs mx-auto w-full' : 'grid-cols-2'}`}>
                    {selectedMode?.id !== 'pov' && (
                      <UploadZone 
                        label="01. Base Model (Wajah/Tubuh)" 
                        image={modelImage} 
                        onClick={() => modelInputRef.current?.click()} 
                        icon={Camera} 
                      />
                    )}
                    <UploadZone 
                      label={selectedMode?.id === 'pov' ? "01. Gambar Produk" : "02. Gambar Produk"} 
                      image={productImage} 
                      onClick={() => productInputRef.current?.click()} 
                      icon={Package} 
                    />
                  </div>
                  
                  <input type="file" ref={modelInputRef} onChange={handleModelUpload} className="hidden" accept="image/*" />
                  <input type="file" ref={productInputRef} onChange={handleProductUpload} className="hidden" accept="image/*" />

                  <div className="bg-[#18181b]/60 border border-zinc-800 rounded-xl p-5 space-y-4 animate-in fade-in slide-in-from-bottom-3">
                      <div className="flex items-center justify-between">
                          <label className="text-[10px] font-bold text-zinc-400 uppercase tracking-widest flex items-center gap-2">
                             <FileText className="w-4 h-4 text-emerald-400" /> Detail Produk
                          </label>
                          {isAnalyzingProduct && (
                              <span className="text-[10px] text-emerald-400 flex items-center gap-1.5 animate-pulse font-medium">
                                  <Sparkles className="w-3.5 h-3.5" /> AI Menganalisis...
                              </span>
                          )}
                      </div>
                      
                      <div className="space-y-2.5">
                          <input 
                              type="text"
                              value={productName}
                              onChange={(e) => setProductName(e.target.value)}
                              placeholder="Nama Produk (Misal: Kopi Susu Aren)"
                              className="w-full bg-[#121214] border border-zinc-800 rounded-lg p-3 text-xs text-white placeholder:text-zinc-650 focus:outline-none focus:border-zinc-700 transition-all font-medium"
                          />
                          <textarea 
                              value={productDesc}
                              onChange={(e) => setProductDesc(e.target.value)}
                              placeholder="Unggah gambar produk agar AI dapat mendeteksi deskripsinya secara otomatis, atau ketik manual..."
                              className="w-full h-20 bg-[#121214] border border-zinc-800 rounded-lg p-3 text-xs text-zinc-300 placeholder:text-zinc-650 focus:outline-none focus:border-zinc-700 transition-all resize-none leading-relaxed"
                          />
                      </div>

                      <div className="flex justify-end pt-1">
                          <button 
                              type="button"
                              onClick={() => { playClick(); setIsProductConfirmed(true); }}
                              disabled={!productName || isProductConfirmed || isAnalyzingProduct}
                              className={`
                                  px-5 py-2 rounded-lg text-[10px] font-bold uppercase tracking-wider transition-all flex items-center gap-1.5
                                  ${isProductConfirmed 
                                      ? 'bg-emerald-500/10 text-emerald-400 border border-emerald-500/20 cursor-default' 
                                      : 'bg-emerald-500 hover:bg-emerald-400 text-neutral-950'}
                                  disabled:opacity-20 disabled:cursor-not-allowed
                              `}
                          >
                              {isAnalyzingProduct ? (
                                  <><Loader2 className="w-3.5 h-3.5 animate-spin" /> Menganalisis</>
                              ) : isProductConfirmed ? (
                                  <><CheckCircle2 className="w-3.5 h-3.5" /> Terkonfirmasi</>
                              ) : (
                                  "Konfirmasi Info"
                              )}
                          </button>
                      </div>
                  </div>

                  <div className="pt-2 pb-6">
                     <ButtonCTA 
                        disabled={!isUploadComplete || !isProductConfirmed || isAnalyzingProduct} 
                        onClick={() => setView('config')}
                     >
                        Lanjutkan
                     </ButtonCTA>
                     {!isProductConfirmed && isUploadComplete && !isAnalyzingProduct && (
                         <p className="text-center text-[10px] text-amber-500 mt-2.5 animate-pulse font-medium">Mohon konfirmasi info produk terlebih dahulu</p>
                     )}
                  </div>
               </div>
            </div>
          </LayoutWrapper>
        );

      case 'config':
        return (
          <LayoutWrapper>
            <Navbar 
              onBack={() => setView('upload-setup')} 
              onViewResult={generatedImages.some(img => img !== null) ? () => setView('result') : null}
              onReset={handleResetAndNew}
              authMode={authMode}
            />
            <div className="flex-grow flex flex-col px-6 pt-32 pb-12 md:px-12 md:pt-40 lg:px-24">
              <div className="max-w-5xl mx-auto w-full grid grid-cols-1 lg:grid-cols-12 gap-8 lg:gap-12">
                <div className="lg:col-span-7 space-y-6 md:space-y-8">
                  <SectionTitle title="Konfigurasi Studio" subtitle="Sesuaikan pengaturan latar, bahasa/adat, dan detail output visual." />
                  
                  {/* BAHASA & GAYA DIALEK ADAT SUMMARY */}
                  <div className="space-y-4 pt-4 border-t border-zinc-900/40">
                    <label className="text-[10px] font-bold text-zinc-400 uppercase tracking-widest flex items-center gap-2">
                      <Globe className="w-3.5 h-3.5 text-emerald-400" /> Bahasa Terpilih
                    </label>

                    <div className="flex flex-col gap-2 bg-[#18181b]/40 border border-zinc-850 rounded-xl p-4">
                      <div className="flex items-center justify-between border-b border-zinc-850/80 pb-2">
                        <span className="text-[10px] text-zinc-550 font-bold uppercase">Bahasa Utama (Aktif)</span>
                        <span className="text-xs font-bold text-emerald-400">{getLanguageLabel(primaryLang)}</span>
                      </div>
                      {secondaryLang && (
                        <div className="flex items-center justify-between border-b border-zinc-850/80 pb-2">
                          <span className="text-[10px] text-zinc-550 font-bold uppercase">Bahasa Kedua (Aktif)</span>
                          <span className="text-xs font-bold text-zinc-300">{getLanguageLabel(secondaryLang)}</span>
                        </div>
                      )}
                      {tertiaryLang && (
                        <div className="flex items-center justify-between pb-1">
                          <span className="text-[10px] text-zinc-550 font-bold uppercase">Bahasa Ketiga (Aktif)</span>
                          <span className="text-xs font-bold text-zinc-300">{getLanguageLabel(tertiaryLang)}</span>
                        </div>
                      )}
                    </div>
                  </div>

                  <div className="space-y-4 pt-6 border-t border-zinc-800">
                    <label className="text-[10px] font-bold text-zinc-400 uppercase tracking-widest flex items-center gap-2">
                      <Monitor className="w-3.5 h-3.5 text-emerald-400" /> Latar / Background (Rekomendasi AI)
                    </label>
                    <div className="relative group">
                      <select 
                        value={bgType === 'custom' ? 'CUSTOM' : config.background} 
                        onChange={(e) => { 
                          playClick(); 
                          if (e.target.value === 'CUSTOM') {
                            setBgType('custom');
                          } else {
                            setBgType('preset');
                            setConfig({...config, background: e.target.value}); 
                          }
                        }} 
                        className="w-full appearance-none bg-[#18181b]/60 border border-zinc-800 rounded-xl px-5 py-4 text-xs text-white focus:outline-none focus:border-zinc-700 transition-all cursor-pointer font-semibold hover:bg-[#18181b]"
                      >
                        {dynamicBackgrounds.map(bg => (
                          <option key={bg} value={bg} className="bg-[#121214] text-zinc-300 py-1.5">{bg}</option>
                        ))}
                        <option value="CUSTOM" className="bg-zinc-800 text-emerald-400 font-bold">✨ Latar Kustom (Ketik Manual)...</option>
                      </select>
                      <div className="absolute right-5 top-1/2 -translate-y-1/2 pointer-events-none text-zinc-500">
                        <ChevronDown className="w-4 h-4" />
                      </div>
                    </div>
                    
                    {bgType === 'custom' && (
                      <div className="mt-2.5 animate-in fade-in">
                        <input 
                          type="text"
                          placeholder="Contoh: 'Garasi rumah dengan lampu neon RGB' atau 'Kantin kampus siang hari'"
                          value={customBg}
                          onChange={(e) => setCustomBg(e.target.value)}
                          className="w-full bg-[#121214] border border-zinc-800 rounded-lg p-3.5 text-xs text-white placeholder:text-zinc-650 focus:outline-none focus:border-zinc-700 transition-all"
                          autoFocus
                        />
                      </div>
                    )}

                    {selectedMode?.id === 'ugc' && (
                      <div className="pt-5 border-t border-zinc-800 space-y-3 animate-in fade-in">
                        <label className="text-[10px] font-bold text-zinc-400 uppercase tracking-widest flex items-center gap-2">
                          <Video className="w-3.5 h-3.5 text-emerald-400" /> Gaya Kamera UGC
                        </label>
                        <p className="text-xs text-zinc-500">Pilih pergerakan dan komposisi kamera.</p>
                        <div className="grid grid-cols-1 md:grid-cols-3 gap-2.5">
                          {[
                            { id: 'Handheld', label: '📱 Handheld', desc: 'Gaya selfie, natural shake' },
                            { id: 'Tripod', label: '📸 Tripod', desc: 'Kamera statis & stabil' },
                            { id: 'Vlog', label: '🚶‍♂️ Vlog', desc: 'Dinamis, wide angle' }
                          ].map(style => (
                            <button
                              key={style.id}
                              type="button"
                              onClick={() => { playClick(); setConfig({...config, ugcStyle: style.id}); }}
                              className={`flex flex-col items-start p-3.5 rounded-xl border transition-all duration-200 text-left ${config.ugcStyle === style.id ? 'bg-[#18181b] border-emerald-500/30' : 'bg-transparent border-zinc-800 hover:border-zinc-700'}`}
                            >
                              <span className={`text-xs font-bold mb-1 ${config.ugcStyle === style.id ? 'text-emerald-400' : 'text-zinc-300'}`}>{style.label}</span>
                              <span className={`text-[10px] ${config.ugcStyle === style.id ? 'text-zinc-400' : 'text-zinc-500'}`}>{style.desc}</span>
                            </button>
                          ))}
                        </div>
                      </div>
                    )}

                  </div>
                </div>
                <div className="lg:col-span-5 space-y-5 lg:pt-14">
                  <div className="p-5 md:p-6 rounded-2xl border border-zinc-800 bg-[#18181b]/60 backdrop-blur-md space-y-6 shadow-xl">
                    <div className="space-y-3">
                      <label className="text-[10px] font-bold text-zinc-400 uppercase tracking-widest flex items-center gap-2">
                        <Layout className="w-3.5 h-3.5 text-emerald-400" /> Format Output (Rasio)
                      </label>
                      <div className="flex flex-col gap-1.5">
                        {ratios.map(r => (
                          <button 
                            key={r.value} 
                            type="button"
                            onClick={() => { playClick(); setConfig({...config, ratio: r.value}); }} 
                            className={`flex items-center justify-between px-4 py-3 rounded-lg border transition-all duration-200 ${config.ratio === r.value ? 'bg-[#121214] border-zinc-700 text-emerald-400' : 'bg-transparent border-zinc-800 text-zinc-400 hover:bg-zinc-900/50 hover:text-zinc-200'}`}
                          >
                            <span className="text-[11px] font-semibold uppercase tracking-wider">{r.label}</span>
                            <span className="text-[10px] font-mono opacity-50">{r.sub}</span>
                          </button>
                        ))}
                      </div>
                    </div>
                    <div className="space-y-3">
                      <label className="text-[10px] font-bold text-zinc-400 uppercase tracking-widest flex items-center gap-2">
                        <Layers className="w-3.5 h-3.5 text-emerald-400" /> Jumlah Scene
                      </label>
                      <div className="grid grid-cols-3 gap-2">
                        {sceneCounts.map(count => (
                          <button 
                            key={count} 
                            type="button"
                            onClick={() => { playClick(); setConfig({...config, scenes: count}); }} 
                            className={`py-2 rounded-lg text-xs font-bold border transition-all duration-200 ${config.scenes === count ? 'bg-emerald-500 text-neutral-950 border-transparent font-black' : 'bg-transparent border-zinc-800 text-zinc-400 hover:border-zinc-700 hover:text-zinc-200'}`}
                          >
                            {count}
                          </button>
                        ))}
                      </div>
                    </div>
                    <div className="pt-2">
                      <ButtonCTA onClick={generateVisual}>Mulai Generasi</ButtonCTA>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </LayoutWrapper>
        );

      case 'result':
        return (
          <LayoutWrapper className="h-screen flex flex-col overflow-hidden bg-[#121214]">
            <div className="h-16 border-b border-zinc-900 flex items-center justify-between px-6 bg-[#121214]/85 backdrop-blur-md z-30 shrink-0">
              <div className="flex items-center gap-4 overflow-hidden">
                <div className="flex flex-col min-w-0">
                  <span className="text-[9px] text-zinc-550 font-bold uppercase tracking-wider">Project Studio</span>
                  <span className="text-xs font-bold text-white truncate">{selectedMode?.title} / {productName}</span>
                </div>
                <div className="hidden md:block h-6 w-px bg-zinc-800"></div>
                <div className="hidden md:flex items-center gap-3 text-xs text-zinc-400">
                  <span className="px-2.5 py-0.5 rounded border border-zinc-800 bg-zinc-900/45 text-[10px] font-medium max-w-[180px] truncate">
                    {bgType === 'custom' ? (customBg || 'Kustom') : config.background}
                  </span>
                  <span className="px-2.5 py-0.5 rounded border border-zinc-800 bg-zinc-900/45 text-[10px] font-medium uppercase">{getLanguageLabel(primaryLang)}</span>
                  <span className="px-2.5 py-0.5 rounded border border-zinc-800 bg-zinc-900/45 text-[10px] font-medium">{config.ratio}</span>
                </div>
              </div>
              <div className="flex items-center gap-2.5">
                {!isGenerating && (
                   <>
                    <button 
                      onClick={() => { playClick(); setIsAudioStudioOpen(true); setIsAudioStudioMinimized(false); }} 
                      className="relative flex items-center gap-1.5 px-4 py-2 bg-emerald-500 text-neutral-950 rounded-lg text-[11px] font-bold tracking-wider uppercase hover:bg-emerald-400 transition-all duration-200 shadow-md shadow-emerald-950/25"
                    >
                      <Volume2 className="w-3.5 h-3.5" /> 
                      <span>Audio Studio</span>
                    </button>

                    <button type="button" onClick={() => { playClick(); setView('config'); }} className="flex items-center gap-1 text-[11px] font-bold tracking-wider uppercase text-zinc-400 hover:text-white transition-colors">
                       <ChevronLeft className="w-4 h-4" />
                       <span className="hidden md:inline">Kembali</span>
                    </button>
                    <button onClick={handleResetAndNew} className="px-4 py-2 bg-[#18181b] border border-zinc-800 hover:border-zinc-700 text-white text-[11px] font-bold tracking-wider uppercase rounded-lg transition-all duration-200">Buat Baru</button>
                   </>
                )}
              </div>
            </div>

            <div className="flex-grow relative overflow-y-auto custom-scrollbar z-10">
              <div className="max-w-[1600px] mx-auto p-6 md:p-8 pb-32">
                
                <div className={`grid gap-5 ${config.scenes === 4 ? 'grid-cols-1 md:grid-cols-2 max-w-4xl mx-auto' : 'grid-cols-1 sm:grid-cols-2 lg:grid-cols-4'}`}>
                  {Array.from({ length: config.scenes }).map((_, index) => {
                    const sceneData = generatedImages[index]; 
                    const image = sceneData?.src;
                    const isLoadingThis = sceneLoadings[index] || (isGenerating && !image);
                    
                    return (
                      <div key={index} className="flex flex-col gap-3 group">
                        <div className={`relative rounded-xl overflow-hidden bg-[#18181b] border border-zinc-800 transition-all duration-300 ${config.ratio === '9:16' ? 'aspect-[9/16]' : config.ratio === '16:9' ? 'aspect-video' : 'aspect-square'} ${image ? 'hover:border-zinc-700 shadow-sm' : ''}`}>
                          {image ? (
                            <>
                               {config.ratio === '16:9' ? (
                                  <img src={image} alt={`Output ${index}`} className={`w-full h-full object-cover object-center transition-all duration-500 ${isLoadingThis ? 'scale-105 blur-sm opacity-55' : 'group-hover:scale-[1.02]'}`} />
                               ) : (
                                  <img src={image} alt={`Output ${index}`} className={`w-full h-full object-cover object-top transition-all duration-500 ${isLoadingThis ? 'scale-105 blur-sm opacity-55' : 'group-hover:scale-[1.02]'}`} />
                               )}
                              
                              {isLoadingThis ? (
                                 <GeneratingState index={index} />
                              ) : (
                                  <div className="absolute inset-0 bg-black/50 opacity-0 group-hover:opacity-100 transition-opacity duration-200 flex items-center justify-center gap-3 z-20">
                                      <button 
                                        type="button"
                                        onClick={() => setPreviewImage(image)} 
                                        className="w-10 h-10 bg-zinc-900 border border-zinc-800 rounded-lg flex items-center justify-center hover:bg-zinc-800 hover:text-white transition-all text-zinc-350 shadow-lg" title="Pratinjau Layar Penuh"
                                      >
                                        <ZoomIn className="w-4.5 h-4.5" />
                                      </button>
                                  </div>
                              )}
                              
                              {sceneData?.role && (
                                  <div className="absolute top-3 left-3 px-2.5 py-0.5 bg-[#121214]/85 backdrop-blur-md border border-zinc-800 rounded z-20">
                                      <span className="text-[9px] font-bold text-zinc-300 uppercase tracking-wider">{sceneData.role}</span>
                                  </div>
                              )}
                            </>
                          ) : (
                            <div className="absolute inset-0">
                              {isLoadingThis ? (
                                 <GeneratingState index={index} />
                              ) : (
                                 <div className="w-full h-full flex flex-col items-center justify-center bg-zinc-900/20"></div>
                              )}
                            </div>
                          )}
                        </div>

                        {image && !isGenerating && (
                          <div className="bg-[#18181b]/70 backdrop-blur-md border border-zinc-800 rounded-xl p-4 space-y-3 shadow-lg">
                             <div className="flex items-center justify-between pb-2.5 border-b border-zinc-850">
                                <span className="text-[10px] font-bold uppercase tracking-wider text-zinc-400">Scene 0{index + 1}</span>
                                <div className="flex items-center gap-1.5">
                                    <button 
                                      type="button" 
                                      onClick={() => regenerateSingleScene(index)} 
                                      disabled={isLoadingThis}
                                      className="flex items-center gap-1.5 px-2.5 py-1.5 bg-zinc-900 border border-zinc-800 rounded text-[10px] font-bold uppercase hover:bg-zinc-800 hover:text-white transition-all disabled:opacity-20"
                                    >
                                      {isLoadingThis ? <Loader2 className="w-3 h-3 animate-spin" /> : <RotateCcw className="w-3 h-3" />} Ulangi
                                    </button>
                                    <button 
                                      type="button" 
                                      onClick={(e) => downloadImage(image, index, e)} 
                                      className="flex items-center gap-1.5 px-2.5 py-1.5 bg-[#121214] border border-zinc-800 rounded text-[10px] font-bold uppercase text-emerald-400 hover:bg-zinc-800 hover:text-emerald-300 transition-all"
                                    >
                                      <Download className="w-3 h-3" /> Unduh
                                    </button>
                                </div>
                             </div>

                             <div className="space-y-3.5 pt-1">
                                 <div className="bg-[#121214] border border-zinc-800 rounded-lg p-3 space-y-2.5">
                                    <div className="flex items-center gap-1.5">
                                      <Sparkles className="w-3.5 h-3.5 text-emerald-400" />
                                      <span className="text-[10px] font-bold uppercase tracking-wider text-zinc-350">Revisi Visual AI</span>
                                    </div>
                                    <div className="relative flex gap-1.5">
                                      <input 
                                        type="text" 
                                        disabled={isLoadingThis} 
                                        placeholder="Cth: Buat pencahayaan lebih terang..." 
                                        className="flex-grow bg-zinc-900 border border-zinc-800 rounded pl-2.5 pr-2.5 py-2 text-xs text-white placeholder:text-zinc-600 focus:outline-none focus:border-zinc-700 transition-all disabled:opacity-20" 
                                        value={scenePrompts[index] || ""} 
                                        onChange={(e) => setScenePrompts(prev => ({...prev, [index]: e.target.value}))} 
                                        onKeyDown={(e) => e.key === 'Enter' && regenerateSingleScene(index)} 
                                      />
                                      <button 
                                        type="button" 
                                        onClick={() => regenerateSingleScene(index)} 
                                        disabled={isLoadingThis || !scenePrompts[index]} 
                                        className="px-3 rounded text-[10px] font-bold uppercase bg-emerald-500 text-neutral-950 hover:bg-emerald-400 transition-all disabled:opacity-20 disabled:cursor-not-allowed flex items-center justify-center shrink-0"
                                      >
                                        {isLoadingThis ? <Loader2 className="w-3.5 h-3.5 animate-spin" /> : "Kirim"}
                                      </button>
                                    </div>
                                 </div>

                                 <div className="bg-[#121214] p-3 rounded-lg border border-zinc-800 space-y-2">
                                     <div className="flex items-center justify-between">
                                         <span className={`text-[9px] font-bold uppercase tracking-wider flex items-center gap-1 ${sceneData.voActive ? 'text-emerald-400' : 'text-zinc-550'}`}>
                                             <Mic className="w-3 h-3"/> Voice Over {sceneData.voActive ? 'Aktif' : 'Nonaktif'}
                                         </span>
                                         <label className="relative inline-flex items-center cursor-pointer">
                                           <input 
                                               type="checkbox" 
                                               className="sr-only peer" 
                                               checked={sceneData.voActive || false} 
                                               onChange={(e) => updateSceneData(index, { voActive: e.target.checked })} 
                                               disabled={isLoadingThis} 
                                           />
                                           <div className="w-6.5 h-3.5 bg-zinc-800 peer-focus:outline-none rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:left-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-2.5 after:w-2.5 after:transition-all peer-checked:bg-emerald-500"></div>
                                         </label>
                                     </div>
                                     
                                     {sceneData.voActive ? (
                                         <textarea 
                                             value={sceneData.voText || ""} 
                                             onChange={(e) => updateSceneData(index, { voText: e.target.value })}
                                             disabled={isLoadingThis}
                                             className="w-full bg-zinc-900 border border-zinc-800 rounded-md p-2 text-xs text-zinc-300 placeholder:text-zinc-650 focus:outline-none focus:border-zinc-700 transition-all resize-none h-14 custom-scrollbar"
                                             placeholder="Ketik naskah Voice Over..."
                                         />
                                     ) : (
                                         <div className="flex items-center justify-center py-1.5 bg-zinc-950/20 rounded border border-zinc-850 border-dashed">
                                             <p className="text-[10px] text-zinc-600 font-medium italic">Ambience / BGM Only</p>
                                         </div>
                                     )}
                                 </div>
                             </div>

                             <button 
                                  type="button" 
                                  onClick={() => handleCopyPrompt(index)} 
                                  className={`w-full py-2 border text-[10px] font-bold uppercase tracking-wider rounded transition-all flex items-center justify-center gap-1.5 group/copy ${copiedStates[index] ? 'bg-emerald-500/10 border-emerald-500/30 text-emerald-400' : 'bg-zinc-900 border-zinc-800 text-zinc-350 hover:border-zinc-700 hover:text-white'}`}
                              >
                                  {copiedStates[index] ? (
                                      <>
                                          <CheckCircle2 className="w-3 h-3" /> Tersalin!
                                      </>
                                  ) : (
                                      <>
                                          <Copy className="w-3 h-3 group-hover/copy:scale-105 transition-transform" /> Salin Prompt Video AI
                                      </>
                                  )}
                             </button>
                          </div>
                        )}
                      </div>
                    );
                  })}
                </div>

                {/* External Video Generator Links */}
                {!isGenerating && generatedImages.some(img => img !== null) && (
                   <div className="mt-16 pt-8 border-t border-zinc-900 flex flex-col items-center gap-4.5 animate-in fade-in duration-500">
                       
                       <button 
                           onClick={handleDownloadAllZip}
                           disabled={isDownloadingZip}
                           className="px-6 py-3 bg-emerald-500 hover:bg-emerald-400 text-neutral-950 rounded-lg text-xs font-bold tracking-widest uppercase transition-all flex items-center gap-2 disabled:opacity-20 disabled:cursor-not-allowed"
                       >
                           {isDownloadingZip ? (
                               <><Loader2 className="w-4 h-4 animate-spin" /> Memproses...</>
                           ) : (
                               <><Archive className="w-4 h-4" /> Download Semua Adegan (.ZIP)</>
                           )}
                       </button>

                       <button 
                           onClick={() => { playClick(); setIsVideoGeneratorsOpen(!isVideoGeneratorsOpen); }}
                           className="px-6 py-3 bg-zinc-900 hover:bg-zinc-800 border border-zinc-800 rounded-lg text-xs font-bold tracking-widest uppercase text-zinc-300 transition-all flex items-center gap-2 hover:border-zinc-700"
                       >
                           <Video className="w-4 h-4 text-emerald-400" />
                           Akses Video Generator
                           {isVideoGeneratorsOpen ? <ChevronUp className="w-3.5 h-3.5" /> : <ChevronDown className="w-3.5 h-3.5" />}
                       </button>

                       {isVideoGeneratorsOpen && (
                           <div className="flex flex-col sm:flex-row items-center gap-3 w-full justify-center animate-in zoom-in-95 duration-200">
                               <a 
                                  href="https://www.meta.ai/" 
                                  target="_blank" 
                                  rel="noopener noreferrer" 
                                  onClick={playClick} 
                                  className="flex items-center justify-center gap-1.5 w-full sm:w-auto px-5 py-3 bg-[#18181b] border border-zinc-800 rounded-lg text-[10px] md:text-xs font-bold uppercase tracking-wider text-white hover:border-zinc-700"
                               >
                                   <ExternalLink className="w-3.5 h-3.5" /> Meta AI
                               </a>
                               <a 
                                  href="https://grok.com/" 
                                  target="_blank" 
                                  rel="noopener noreferrer" 
                                  onClick={playClick} 
                                  className="flex items-center justify-center gap-1.5 w-full sm:w-auto px-5 py-3 bg-[#18181b] border border-zinc-800 rounded-lg text-[10px] md:text-xs font-bold uppercase tracking-wider text-white hover:border-zinc-700"
                               >
                                   <ExternalLink className="w-3.5 h-3.5" /> Grok AI
                               </a>
                               <a 
                                  href="https://labs.google/fx/tools/flow" 
                                  target="_blank" 
                                  rel="noopener noreferrer" 
                                  onClick={playClick} 
                                  className="flex items-center justify-center gap-1.5 w-full sm:w-auto px-5 py-3 bg-[#18181b] border border-zinc-800 rounded-lg text-[10px] md:text-xs font-bold uppercase tracking-wider text-white hover:border-zinc-700"
                               >
                                   <ExternalLink className="w-3.5 h-3.5" /> Flow AI
                               </a>
                           </div>
                       )}
                   </div>
                )}
              </div>
            </div>

            {previewImage && (
              <div className="fixed inset-0 z-[100] bg-[#0c0c0e]/95 flex items-center justify-center p-4 md:p-12 animate-in fade-in duration-150" onClick={() => setPreviewImage(null)}>
                <div 
                  className={`
                    relative overflow-hidden rounded-lg shadow-2xl transition-all flex items-center justify-center bg-[#121214]
                    ${config.ratio === '9:16' ? 'aspect-[9/16] h-full max-h-[85vh] w-auto' : ''}
                    ${config.ratio === '16:9' ? 'aspect-[16/9] w-full max-w-[85vw] h-auto' : ''}
                    ${config.ratio === '1:1' ? 'aspect-square h-full max-h-[85vh] w-auto' : ''}
                    border border-zinc-800
                  `}
                  onClick={(e) => e.stopPropagation()}
                >
                   {config.ratio === '16:9' ? (
                      <img src={previewImage} alt="Full Preview" className="w-full h-full object-cover object-center" />
                   ) : (
                      <img src={previewImage} alt="Full Preview" className="w-full h-full object-cover object-top" />
                   )}
                  
                  <button type="button" onClick={() => setPreviewImage(null)} className="absolute top-4 right-4 p-1.5 bg-zinc-900 hover:bg-zinc-800 rounded-lg text-white border border-zinc-800 z-20 shadow-md">
                    <X className="w-4 h-4" />
                  </button>
                </div>
              </div>
            )}
          </LayoutWrapper>
        );

      default:
        return null;
    }
  };

  const GeneratingState = ({ index }) => (
    <div className="absolute inset-0 flex flex-col items-center justify-center bg-[#0c0c0e]/90 backdrop-blur-md z-20 transition-all duration-500">
        <div className="absolute w-64 h-64 bg-emerald-500/5 rounded-full blur-3xl animate-pulse pointer-events-none" />
        <div className="relative z-10 flex flex-col items-center gap-6">
          <div className="relative w-12 h-12">
             <div className="absolute inset-0 rounded-full border-[1.5px] border-zinc-800" />
             <div className="absolute inset-0 rounded-full border-t-[1.5px] border-emerald-500 animate-[spin_1s_linear_infinite]" />
             <div className="absolute inset-0 flex items-center justify-center">
               <div className="w-1.5 h-1.5 bg-emerald-400 rounded-full animate-ping" />
             </div>
          </div>
          <div className="text-center space-y-2">
            <p className="text-[10px] font-semibold tracking-[0.3em] text-emerald-400 uppercase animate-pulse">Rendering Image</p>
            <div className="flex items-center justify-center gap-3 opacity-60">
               <div className="w-4 h-px bg-zinc-800" />
               <p className="text-[9px] font-medium tracking-widest text-zinc-400 uppercase font-mono">Scene {String(index + 1).padStart(2, '0')}</p>
               <div className="w-4 h-px bg-zinc-800" />
            </div>
          </div>
        </div>
    </div>
  );

  return (
    <React.Fragment>
      {renderContent()}

      {/* --- AUDIO STUDIO POPUP (FLOATING WINDOW) --- */}
      {isAudioStudioOpen && (
        <div className={`fixed z-[100] right-4 bottom-4 md:right-8 md:bottom-8 bg-[#18181b]/95 backdrop-blur-md border border-zinc-800 shadow-2xl transition-all duration-300 flex flex-col overflow-hidden origin-bottom-right ${isAudioStudioMinimized ? 'w-[180px] h-[44px] rounded-xl opacity-90' : 'w-[calc(100vw-32px)] md:w-[650px] h-[calc(100vh-100px)] md:h-[480px] rounded-2xl opacity-100 scale-100'}`}>
           <div 
             className="flex items-center justify-between px-4 py-3 bg-[#121214] border-b border-zinc-900 cursor-pointer hover:bg-zinc-900 transition-colors shrink-0"
             onClick={() => setIsAudioStudioMinimized(!isAudioStudioMinimized)}
           >
              <div className="flex items-center gap-2">
                <div className="w-5 h-5 rounded bg-zinc-900 flex items-center justify-center border border-zinc-800 text-emerald-400">
                   <Volume2 className="w-3.5 h-3.5" />
                </div>
                <span className="text-[10px] font-bold tracking-wider uppercase text-zinc-300">Audio Studio</span>
              </div>
              <div className="flex items-center gap-1.5">
                <button 
                  type="button"
                  onClick={(e) => { e.stopPropagation(); setIsAudioStudioMinimized(!isAudioStudioMinimized); }}
                  className="p-1 rounded text-zinc-500 hover:text-white transition-colors"
                >
                  {isAudioStudioMinimized ? <ChevronUp className="w-3.5 h-3.5" /> : <ChevronDown className="w-3.5 h-3.5" />}
                </button>
                <button 
                  type="button"
                  onClick={(e) => { e.stopPropagation(); setIsAudioStudioOpen(false); setIsAudioStudioMinimized(false); }}
                  className="p-1 rounded text-zinc-500 hover:text-white transition-colors"
                >
                  <X className="w-3.5 h-3.5" />
                </button>
              </div>
           </div>

           {!isAudioStudioMinimized && (
              <div className="flex-grow overflow-y-auto custom-scrollbar p-5 flex flex-col gap-4">
                  <div className="flex items-center justify-between border-b border-zinc-850 pb-3">
                     <div>
                        <h2 className="text-xs font-bold text-white tracking-wide mb-0.5">Naskah & Suara AI</h2>
                        <p className="text-[10px] text-zinc-500">Atur naskah Voice Over dan tentukan karakter suara AI Anda.</p>
                     </div>
                  </div>
                  
                  <div className="grid grid-cols-1 md:grid-cols-5 gap-4.5 flex-grow">
                      <div className="md:col-span-3 flex flex-col gap-2.5 h-full">
                          <div className="flex items-center justify-between">
                              <h3 className="text-[10px] font-bold uppercase tracking-wider text-zinc-400 flex items-center gap-1">
                                  <FileText className="w-3 h-3 text-emerald-400"/> Naskah Voice Over
                              </h3>
                              <div className="flex items-center gap-1 bg-[#121214] p-0.5 rounded border border-zinc-800">
                                 {['15s', '30s', '45s'].map(dur => (
                                   <button 
                                      key={dur} 
                                      onClick={() => { playClick(); setVoDuration(dur); }}
                                      className={`px-2 py-0.5 text-[9px] font-bold rounded transition-all ${voDuration === dur ? 'bg-zinc-800 text-emerald-400 shadow-sm' : 'text-zinc-500 hover:text-zinc-350'}`}
                                   >
                                      {dur}
                                   </button>
                                 ))}
                              </div>
                          </div>

                          <textarea
                              value={voScript}
                              onChange={(e) => setVoScript(e.target.value)}
                              placeholder="Ketik naskah Voice Over Anda di sini, atau tekan tombol AI Auto-Script di bawah..."
                              className="w-full flex-grow min-h-[100px] bg-[#121214] border border-zinc-800 rounded-lg p-3 text-xs text-white placeholder:text-zinc-600 focus:outline-none focus:border-zinc-700 transition-all resize-none custom-scrollbar leading-relaxed"
                          />

                          <button 
                             onClick={handleGenerateVOScript}
                             disabled={isGeneratingVOScript}
                             className="w-full py-2 bg-zinc-900 border border-zinc-800 hover:bg-zinc-800 text-zinc-300 hover:text-white rounded-lg text-[10px] font-bold uppercase tracking-wider flex items-center justify-center gap-1.5 transition-all disabled:opacity-20"
                          >
                             {isGeneratingVOScript ? <><Loader2 className="w-3 h-3 animate-spin"/> Menyusun...</> : <><Wand2 className="w-3 h-3 text-emerald-400"/> Auto-Script ({voDuration})</>}
                          </button>
                      </div>

                      <div className="md:col-span-2 flex flex-col gap-3.5 border-t md:border-t-0 md:border-l border-zinc-900 pt-3 md:pt-0 md:pl-4">
                          <h3 className="text-[10px] font-bold uppercase tracking-wider text-zinc-400 flex items-center gap-1">
                              <Mic className="w-3 h-3 text-emerald-400"/> Pengaturan Karakter
                          </h3>

                          <div className="space-y-3">
                              <div>
                                  <label className="text-[9px] font-bold text-zinc-500 uppercase tracking-wider block mb-1">Karakter Suara</label>
                                  <div className="relative">
                                      <select 
                                         value={voVoice} 
                                         onChange={(e) => { playClick(); setVoVoice(e.target.value); }}
                                         className="w-full appearance-none bg-[#121214] border border-zinc-800 rounded-lg px-3 py-2 text-[10px] text-white focus:outline-none focus:border-zinc-700 cursor-pointer font-medium"
                                      >
                                         {VOICES.map(v => <option key={v.id} value={v.id} className="bg-[#121214] text-white">{v.name}</option>)}
                                      </select>
                                      <ChevronDown className="w-3.5 h-3.5 absolute right-3 top-1/2 -translate-y-1/2 text-zinc-500 pointer-events-none" />
                                  </div>
                              </div>

                              <div>
                                  <label className="text-[9px] font-bold text-zinc-500 uppercase tracking-wider block mb-1">Gaya Bicara (Emosi)</label>
                                  <div className="relative">
                                      <select 
                                         value={voTone} 
                                         onChange={(e) => { playClick(); setVoTone(e.target.value); }}
                                         className="w-full appearance-none bg-[#121214] border border-zinc-800 rounded-lg px-3 py-2 text-[10px] text-white focus:outline-none focus:border-zinc-700 cursor-pointer font-medium"
                                      >
                                         {TONES.map(t => <option key={t} value={t} className="bg-[#121214] text-white">{t}</option>)}
                                      </select>
                                      <ChevronDown className="w-3.5 h-3.5 absolute right-3 top-1/2 -translate-y-1/2 text-zinc-500 pointer-events-none" />
                                  </div>
                              </div>
                          </div>

                          <div className="flex-grow flex flex-col justify-end gap-2.5 mt-2">
                               {voAudioUrl && (
                                   <div className="bg-[#121214] border border-zinc-800 rounded-lg p-2.5 flex flex-col gap-2">
                                       <audio controls src={voAudioUrl} className="w-full h-7 opacity-90" />
                                       <a 
                                          href={voAudioUrl} 
                                          download={`VO_${productName || 'Audio'}_${voTone}.wav`}
                                          className="w-full py-1.5 bg-zinc-900 text-zinc-300 rounded text-[9px] font-bold uppercase hover:bg-zinc-850 hover:text-white transition-colors border border-zinc-800 text-center"
                                       >
                                          Unduh Audio (WAV)
                                       </a>
                                   </div>
                               )}

                               <button 
                                  onClick={handleGenerateVOAudio} 
                                  disabled={isGeneratingVOAudio || !voScript.trim()}
                                  className="w-full py-2.5 bg-emerald-500 hover:bg-emerald-400 text-neutral-950 font-bold text-[10px] tracking-wider uppercase rounded-lg transition-all duration-200 disabled:opacity-20 disabled:cursor-not-allowed flex items-center justify-center gap-1.5"
                               >
                                  {isGeneratingVOAudio ? (
                                      <><Loader2 className="w-3.5 h-3.5 animate-spin"/> Memproses...</>
                                  ) : (
                                      <><PlayCircle className="w-3.5 h-3.5"/> Buat Audio VO</>
                                  )}
                               </button>
                          </div>
                      </div>
                  </div>
              </div>
           )}
        </div>
      )}
    </React.Fragment>
  );
}
