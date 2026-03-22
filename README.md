# Control
アプリせんよう
my-app/
├─ public/
│   └─ index.html
├─ src/
│   ├─ components/
│   │   └─ PlayerCard.tsx
│   ├─ hooks/
│   │   └─ use-game.ts
│   ├─ App.tsx
│   ├─ main.tsx
│   └─ index.css
├─ package.json
├─ tsconfig.json
├─ tailwind.config.js
└─ vite.config.ts
{
  "name": "my-app",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "framer-motion": "^10.12.16",
    "lucide-react": "^0.277.0",
    "clsx": "^2.0.0",
    "tailwind-merge": "^1.14.0"
  },
  "devDependencies": {
    "typescript": "^5.2.2",
    "vite": "^5.0.0",
    "@vitejs/plugin-react": "^4.0.0",
    "tailwindcss": "^3.3.3",
    "postcss": "^8.4.0",
    "autoprefixer": "^10.4.0"
  },
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  }
}
{
  "compilerOptions": {
    "target": "ESNext",
    "useDefineForClassFields": true,
    "lib": ["DOM", "DOM.Iterable", "ESNext"],
    "allowJs": false,
    "skipLibCheck": true,
    "esModuleInterop": false,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "module": "ESNext",
    "moduleResolution": "Node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx"
  },
  "include": ["src"]
}
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./index.html", "./src/**/*.{ts,tsx}"],
  theme: {
    extend: {
      colors: {
        primary: "#00ffff",
        secondary: "#ff00ff",
        success: "#22c55e",
        destructive: "#ef4444",
        card: "#1f1f1f"
      },
      animation: {
        glow: "glow 1.5s infinite alternate"
      },
      keyframes: {
        glow: {
          "0%": { boxShadow: "0 0 10px rgba(0,255,255,0.2)" },
          "100%": { boxShadow: "0 0 20px rgba(0,255,255,0.6)" }
        }
      }
    }
  },
  plugins: []
}
export interface UserState {
  name: string;
  level: number;
  role: 'player' | 'spectator';
  playing: boolean;
  bpm: number;
  shasei: boolean;
  patience: number;
  stopCount: number;
}
import React from 'react';
import { motion } from 'framer-motion';
import { Activity, PauseCircle, Zap, ShieldAlert } from 'lucide-react';
import { UserState } from '@/hooks/use-game';
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

function getPatienceBarColor(patience: number): string {
  if (patience <= 20) return 'bg-blue-500 shadow-[0_0_6px_rgba(59,130,246,0.6)]';
  if (patience <= 60) return 'bg-green-500 shadow-[0_0_6px_rgba(34,197,94,0.6)]';
  if (patience === 80) return 'bg-orange-500 shadow-[0_0_6px_rgba(249,115,22,0.6)]';
  return 'bg-red-500 shadow-[0_0_6px_rgba(239,68,68,0.7)]';
}

function getPatienceLabelColor(patience: number): string {
  if (patience <= 20) return 'text-blue-400';
  if (patience <= 60) return 'text-green-400';
  if (patience === 80) return 'text-orange-400';
  return 'text-red-400';
}

interface PlayerCardProps {
  id: string;
  user: UserState;
  isMyTurn: boolean;
  isMe: boolean;
}

export function PlayerCard({ id, user, isMyTurn, isMe }: PlayerCardProps) {
  const isSpectator = user.role === 'spectator';

  if (isSpectator) {
    return (
      <div className={cn(
        "bg-black/30 border rounded-lg px-4 py-3 flex items-center justify-between",
        isMe ? "border-secondary" : "border-white/5"
      )}>
        <span className="font-medium text-muted-foreground">{user.name}</span>
        <span className="text-xs uppercase tracking-widest text-secondary font-bold bg-secondary/10 px-2 py-1 rounded">観戦者</span>
      </div>
    );
  }
  return (
    <motion.div
      layout
      initial={{ opacity: 0, scale: 0.9 }}
      animate={{ opacity: 1, scale: 1 }}
      className={cn(
        "relative rounded-2xl overflow-hidden transition-all duration-500",
        "bg-card/80 border backdrop-blur-md",
        isMe ? "border-primary/50 shadow-[0_0_20px_rgba(0,255,255,0.15)]" : "border-white/10",
        isMyTurn && "animate-glow border-primary"
      )}
    >
      {/* Turn Indicator Bar */}
      <div className={cn(
        "absolute left-0 top-0 bottom-0 w-1.5 transition-all duration-500",
        isMyTurn ? "bg-primary shadow-[0_0_10px_currentColor]" : "bg-transparent"
      )} />

      <div className="p-4 md:p-5 flex flex-col h-full pl-6">
        
        {/* Header */}
        <div className="flex justify-between items-start mb-4">
          <div>
            <div className="flex items-center gap-2">
              <span className="font-bold text-lg leading-none truncate max-w-[120px]" title={user.name}>
                {user.name}
              </span>
              {isMe && (
                <span className="text-[10px] uppercase font-bold bg-primary/20 text-primary px-1.5 py-0.5 rounded">あなた</span>
              )}
            </div>
            <div className="text-xs text-muted-foreground mt-1 flex items-center gap-1 font-display">
              LVL {user.level}
            </div>
          </div>

          <div className={cn(
            "flex items-center gap-1.5 px-2.5 py-1 rounded-full text-xs font-bold uppercase tracking-wider border",
            user.playing 
              ? "bg-success/10 text-success border-success/30" 
              : "bg-destructive/10 text-destructive border-destructive/30"
          )}>
            {user.playing ? <Activity className="w-3 h-3 animate-pulse" /> : <PauseCircle className="w-3 h-3" />}
            {user.playing ? '再生中' : '停止中'}
          </div>
        </div>

        {/* Big BPM Display */}
        <div className="flex-grow flex items-center justify-center py-2">
          <div className="text-center relative">
            {user.shasei ? (
              <motion.div
                key="shasei"
                initial={{ scale: 0.5, opacity: 0 }}
                animate={{ scale: 1, opacity: 1 }}
                className="text-4xl md:text-5xl font-black tracking-tighter text-pink-400 drop-shadow-[0_0_12px_rgba(236,72,153,0.8)]"
              >
                射精
              </motion.div>
            ) : (
              <motion.div 
                key={user.bpm}
                initial={{ scale: 1.5, opacity: 0 }}
                animate={{ scale: 1, opacity: 1 }}
                className="text-5xl md:text-6xl font-display font-black text-transparent bg-clip-text bg-gradient-to-br from-white to-white/60 tracking-tighter"
              >
                {user.bpm}
              </motion.div>
            )}
            {!user.shasei && <span className="text-xs text-muted-foreground font-display tracking-widest absolute -bottom-4 left-1/2 -translate-x-1/2">BPM</span>}
          </div>
        </div>

        {/* Stats Footer */}
        <div className="mt-6 space-y-3">
          {/* Patience Gauge */}
          <div className="space-y-1">
            <div className="flex justify-between text-[10px] uppercase font-bold text-muted-foreground">
              <span className="flex items-center gap-1"><Zap className="w-3 h-3" /> 我慢</span>
              <span className={`font-display font-bold ${getPatienceLabelColor(user.patience)}`}>{user.patience}%</span>
            </div>
            <div className="h-1.5 w-full bg-black/50 rounded-full overflow-hidden">
              <motion.div 
                className={`h-full rounded-full transition-colors duration-500 ${getPatienceBarColor(user.patience)}`}
                initial={{ width: 0 }}
                animate={{ width: `${user.patience}%` }}
                transition={{ type: "spring", stiffness: 100 }}
              />
            </div>
          </div>

          {/* Stop Count */}
          <div className="flex justify-between items-center bg-black/20 rounded-lg px-3 py-2 border border-white/5">
            <span className="text-[10px] font-bold uppercase text-muted-foreground flex items-center gap-1">
              <ShieldAlert className="w-3 h-3" /> 寸止め回数
            </span>
            <span className="font-display font-bold text-destructive text-sm">{user.stopCount}</span>
          </div>
        </div>
      </div>
    </motion.div>
  );
}
import React from 'react';
import { PlayerCard } from './components/PlayerCard';
import { UserState } from './hooks/use-game';

export default function App() {
  const dummyUser: UserState = {
    name: "太貴",
    level: 5,
    role: "player",
    playing: true,
    bpm: 120,
    shasei: false,
    patience: 45,
    stopCount: 2
  };

  return (
    <div className="min-h-screen bg-gray-900 flex items-center justify-center p-4">
      <PlayerCard id="1" user={dummyUser} isMyTurn={true} isMe={true} />
    </div>
  );
}
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import './index.css';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
@tailwind base;
@tailwind components;
@tailwind utilities;

body {
  font-family: 'Inter', sans-serif;
}

<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>PlayerCard Test</title>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
