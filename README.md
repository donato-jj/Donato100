<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Desafío IA – Evaluador Inteligente</title>

<style>
:root {
  --bg: #0b0f1a;
  --panel: #12182b;
  --neon: #00fff7;
  --danger: #ff4b5c;
  --success: #00ff9c;
  --text: #e8ecf1;
}

* {
  box-sizing: border-box;
  font-family: "Segoe UI", Roboto, sans-serif;
}

body {
  margin: 0;
  background: radial-gradient(circle at top, #101633, var(--bg));
  color: var(--text);
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}

.game-container {
  width: 95%;
  max-width: 900px;
  background: var(--panel);
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 0 40px rgba(0,255,247,0.15);
  animation: fadeIn 1s ease;
}

h1 {
  text-align: center;
  color: var(--neon);
  margin-bottom: 10px;
}

.subtitle {
  text-align: center;
  opacity: 0.8;
  margin-bottom: 20px;
}

.progress {
  height: 8px;
  background: #1f2747;
  border-radius: 10px;
  overflow: hidden;
  margin-bottom: 20px;
}

.progress-bar {
  height: 100%;
  width: 0%;
  background: linear-gradient(90deg, var(--neon), #6affff);
  transition: width 0.4s ease;
}

.question-box {
  margin-top: 20px;
  animation: slideUp 0.6s ease;
}

.topic {
  font-size: 0.85rem;
  color: var(--neon);
  margin-bottom: 8px;
}

.question {
  font-size: 1.2rem;
  margin-bottom: 16px;
}

.answers button,
.start-btn,
.restart-btn {
  width: 100%;
  margin: 8px 0;
  padding: 12px;
  background: #1c2444;
  color: var(--text);
  border: 1px solid #2b3566;
  border-radius: 10px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.answers button:hover,
.start-btn:hover,
.restart-btn:hover {
  background: var(--neon);
  color: #000;
}

input[type="text"] {
  width: 100%;
  padding: 12px;
  border-radius: 10px;
  border: none;
  margin-bottom: 10px;
  font-size: 1rem;
}

.hidden {
  display: none;
}

.result {
  text-align: center;
  animation: fadeIn 1s ease;
}

.result h2 {
  color: var(--danger);
}

.success {
  color: var(--success);
}

@keyframes fadeIn {
  from {opacity: 0;}
  to {opacity: 1;}
}

@keyframes slideUp {
  from {transform: translateY(20px); opacity: 0;}
  to {transform: translateY(0); opacity: 1;}
}
</style>
</head>

<body>

<div class="game-container">
  <h1>🧠 Evaluador IA</h1>
  <div class="subtitle">Big Data · Biotecnología · Algoritmos · Python · IA</div>

  <div class="progress hidden">
    <div class="progress-bar" id="progressBar"></div>
  </div>

  <div id="startScreen">
    <p>
      Este desafío evalúa conocimiento real.  
      <strong>Un solo error implica descalificación inmediata.</strong>
    </p>
    <button class="start-btn" onclick="startGame()">Iniciar Evaluación</button>
  </div>

  <div id="questionScreen" class="hidden">
    <div class="question-box">
      <div class="topic" id="topic"></div>
      <div class="question" id="question"></div>
      <div class="answers" id="answers"></div>
    </div>
  </div>

  <div id="resultScreen" class="hidden result"></div>
</div>

<script>
// =====================
// BASE DE CONOCIMIENTO IA
// =====================
const knowledgeBase = [
  {
    topic: "Big Data",
    type: "mc",
    difficulty: 1,
    questionVariants: [
      "¿Cuál es la función principal de Apache Hadoop?",
      "¿Para qué se utiliza Hadoop en Big Data?"
    ],
    options: [
      "Procesar grandes volúmenes de datos de forma distribuida",
      "Crear interfaces gráficas",
      "Diseñar hardware",
      "Reemplazar bases de datos relacionales"
    ],
    correct: 0
  },
  {
    topic: "Algoritmos",
    type: "tf",
    difficulty: 1,
    questionVariants: [
      "Un algoritmo con complejidad O(n log n) es más eficiente que uno O(n²) para grandes volúmenes de datos."
    ],
    correct: true
  },
  {
    topic: "Historia de la IA",
    type: "short",
    difficulty: 2,
    questionVariants: [
      "¿Quién propuso la prueba para evaluar inteligencia en máquinas en 1950?"
    ],
    keywords: ["turing", "alan turing"]
  },
  {
    topic: "Biotecnología",
    type: "mc",
    difficulty: 2,
    questionVariants: [
      "¿Qué tecnología permite editar genes con alta precisión?"
    ],
    options: [
      "CRISPR-Cas9",
      "PCR",
      "RNA mensajero",
      "Electroforesis"
    ],
    correct: 0
  },
  {
    topic: "Python",
    type: "short",
    difficulty: 3,
    questionVariants: [
      "¿Por qué Python es tan utilizado en inteligencia artificial?"
    ],
    keywords: ["simplicidad", "librerías", "legible", "productividad"]
  },
  {
    topic: "Uso real de IA",
    type: "tf",
    difficulty: 3,
    questionVariants: [
      "La inteligencia artificial actual posee conciencia propia."
    ],
    correct: false
  }
];

// =====================
// ESTADO DEL JUEGO
// =====================
let usedQuestions = [];
let score = 0;
let currentDifficulty = 1;

// =====================
// FUNCIONES IA
// =====================
function startGame() {
  score = 0;
  currentDifficulty = 1;
  usedQuestions = [];
  document.getElementById("startScreen").classList.add("hidden");
  document.querySelector(".progress").classList.remove("hidden");
  nextQuestion();
}

function selectQuestion() {
  const candidates = knowledgeBase.filter(q =>
    q.difficulty <= currentDifficulty &&
    !usedQuestions.includes(q)
  );
  if (candidates.length === 0) return null;
  return candidates[Math.floor(Math.random() * candidates.length)];
}

function nextQuestion() {
  const q = selectQuestion();
  if (!q) {
    showSuccess();
    return;
  }
  usedQuestions.push(q);
  renderQuestion(q);
}

function renderQuestion(q) {
  document.getElementById("questionScreen").classList.remove("hidden");
  document.getElementById("topic").textContent = `Tema: ${q.topic}`;
  const text = q.questionVariants[Math.floor(Math.random() * q.questionVariants.length)];
  document.getElementById("question").textContent = text;

  const answersDiv = document.getElementById("answers");
  answersDiv.innerHTML = "";

  if (q.type === "mc") {
    q.options.forEach((opt, i) => {
      const btn = document.createElement("button");
      btn.textContent = opt;
      btn.onclick = () => evaluateAnswer(i === q.correct, q.topic);
      answersDiv.appendChild(btn);
    });
  }

  if (q.type === "tf") {
    ["Verdadero", "Falso"].forEach(val => {
      const btn = document.createElement("button");
      btn.textContent = val;
      btn.onclick = () => {
        const answer = val === "Verdadero";
        evaluateAnswer(answer === q.correct, q.topic);
      };
      answersDiv.appendChild(btn);
    });
  }

  if (q.type === "short") {
    const input = document.createElement("input");
    input.type = "text";
    input.placeholder = "Escribí tu respuesta…";
    const btn = document.createElement("button");
    btn.textContent = "Enviar respuesta";
    btn.onclick = () => {
      const value = input.value.toLowerCase();
      const ok = q.keywords.some(k => value.includes(k));
      evaluateAnswer(ok, q.topic);
    };
    answersDiv.appendChild(input);
    answersDiv.appendChild(btn);
  }

  updateProgress();
}

function evaluateAnswer(correct, topic) {
  if (!correct) {
    disqualify(topic);
    return;
  }
  score++;
  if (score % 2 === 0) currentDifficulty++;
  nextQuestion();
}

function updateProgress() {
  document.getElementById("progressBar").style.width = Math.min(score * 15, 100) + "%";
}

// =====================
// RESULTADOS
// =====================
function disqualify(topic) {
  document.getElementById("questionScreen").classList.add("hidden");
  const r = document.getElementById("resultScreen");
  r.classList.remove("hidden");
  r.innerHTML = `
    <h2>⛔ Descalificado</h2>
    <p>Respuesta incorrecta. No dominás los fundamentos de Big Data e Inteligencia Artificial.</p>
    <p><strong>Puntaje:</strong> ${score}</p>
    <p><strong>Tema fallado:</strong> ${topic}</p>
    <p><strong>Nivel estimado:</strong> ${estimateLevel()}</p>
    <button class="restart-btn" onclick="location.reload()">Reiniciar Evaluación</button>
  `;
}

function showSuccess() {
  document.getElementById("questionScreen").classList.add("hidden");
  const r = document.getElementById("resultScreen");
  r.classList.remove("hidden");
  r.innerHTML = `
    <h2 class="success">✔ Evaluación Superada</h2>
    <p>Demostraste dominio sólido de los fundamentos.</p>
    <p><strong>Puntaje total:</strong> ${score}</p>
    <p><strong>Nivel:</strong> Experto</p>
    <button class="restart-btn" onclick="location.reload()">Reiniciar</button>
  `;
}

function estimateLevel() {
  if (score <= 1) return "Básico";
  if (score <= 3) return "Intermedio";
  if (score <= 5) return "Avanzado";
  return "Experto";
}
</script>

</body>
</html>
