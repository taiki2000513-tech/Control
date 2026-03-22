import { initializeApp } from "https://www.gstatic.com/firebasejs/10.6.0/firebase-app.js";
import { getDatabase, ref, onValue, set } from "https://www.gstatic.com/firebasejs/10.6.0/firebase-database.js";

const firebaseConfig = {
  apiKey: "<API_KEY>",
  authDomain: "<PROJECT_ID>.firebaseapp.com",
  databaseURL: "https://<PROJECT_ID>.firebaseio.com",
  projectId: "<PROJECT_ID>"
};

const app = initializeApp(firebaseConfig);
const db = getDatabase(app);

// ランダムにユーザーIDを生成（本番はログインで管理可）
const userId = "user_" + Math.floor(Math.random() * 1000);
const userRef = ref(db, "metronomes/" + userId);

// 初期状態
set(userRef, { bpm: 120, playing: false });

// Web Audio API 初期化
const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
let intervalId = null;

// メトロノーム関数
function playClick() {
  const osc = audioCtx.createOscillator();
  const gain = audioCtx.createGain();
  osc.type = 'square';
  osc.frequency.setValueAtTime(1000, audioCtx.currentTime);
  gain.gain.setValueAtTime(0.2, audioCtx.currentTime);
  osc.connect(gain);
  gain.connect(audioCtx.destination);
  osc.start();
  osc.stop(audioCtx.currentTime + 0.05);
}

function startMetronome(bpm) {
  playClick();
  intervalId = setInterval(() => playClick(), 60000 / bpm);
}

function stopMetronome() {
  clearInterval(intervalId);
  intervalId = null;
}

// UIイベント
const bpmInput = document.getElementById('bpm');
const startStopBtn = document.getElementById('startStopBtn');

bpmInput.addEventListener('change', () => {
  const bpm = parseInt(bpmInput.value);
  set(userRef, { bpm, playing: intervalId !== null });
  if (intervalId) {
    stopMetronome();
    startMetronome(bpm);
  }
});

startStopBtn.addEventListener('click', () => {
  onValue(userRef, (snapshot) => {
    const data = snapshot.val();
    if (!data.playing) {
      startMetronome(data.bpm);
      set(userRef, { bpm: data.bpm, playing: true });
    } else {
      stopMetronome();
      set(userRef, { bpm: data.bpm, playing: false });
    }
  }, { onlyOnce: true });
});
