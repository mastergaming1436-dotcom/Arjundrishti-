import React, { useState, useEffect } from "react";
import confetti from "canvas-confetti";

// Paper 1 Demo (20 Questions only)
const TESTS = {
  "Paper 1 (28 Oct 2023 Shift-1)": {
    duration: 120,
    totalQuestions: 20,
    negative: 0.25,
    questions: [
      {
        id: 1,
        text: "शिवाजी की मृत्यु के बाद मराठा राज्य में प्रभावी शक्ति किसके पास थी और कौन-सा शहर राजधानी बना?",
        options: [
          { key: "a", label: "सतारा" },
          { key: "b", label: "नागपुर" },
          { key: "c", label: "नासिक" },
          { key: "d", label: "पुणे" },
        ],
        answer: "d",
      },
      {
        id: 2,
        text: "महावीर स्वामी को 'जिन' कहा गया, 'जिन' का अर्थ है?",
        options: [
          { key: "a", label: "महान" },
          { key: "b", label: "आत्मा" },
          { key: "c", label: "भगवान" },
          { key: "d", label: "अहिंसक" },
        ],
        answer: "a",
      },
      // 👉 बाकी 18 सवाल इसी format में डालते जाएँ
    ],
  },
};

export default function UPSSSPETTestApp() {
  const [selectedTest, setSelectedTest] = useState("Paper 1 (28 Oct 2023 Shift-1)");
  const [timeLeft, setTimeLeft] = useState(TESTS[selectedTest].duration * 60);
  const [submitted, setSubmitted] = useState(false);
  const [started, setStarted] = useState(false);
  const [attemptHistory, setAttemptHistory] = useState([]);
  const [answers, setAnswers] = useState({});
  const [questionTimes, setQuestionTimes] = useState({});
  const [currentQ, setCurrentQ] = useState(0);
  const [lastTimestamp, setLastTimestamp] = useState(Date.now());

  const { duration, totalQuestions, negative, questions } = TESTS[selectedTest];

  // Timer
  useEffect(() => {
    if (!started || submitted) return;
    if (timeLeft <= 0) {
      handleSubmit();
      return;
    }
    const timer = setInterval(() => {
      setTimeLeft((t) => (t > 0 ? t - 1 : 0));
    }, 1000);
    return () => clearInterval(timer);
  }, [started, submitted, timeLeft]);

  // Auto save time per question
  useEffect(() => {
    if (!started || submitted) return;
    const now = Date.now();
    const diff = Math.floor((now - lastTimestamp) / 1000);
    setQuestionTimes((prev) => ({
      ...prev,
      [questions[currentQ].id]: (prev[questions[currentQ].id] || 0) + diff,
    }));
    setLastTimestamp(now);
  }, [currentQ]);

  const handleSelect = (qid, key) => {
    if (answers[qid]) return;
    setAnswers((a) => ({ ...a, [qid]: key }));
    if (key === questions.find((q) => q.id === qid).answer) confetti();
    // Auto-next after 2s
    setTimeout(() => {
      if (currentQ < questions.length - 1) setCurrentQ((c) => c + 1);
    }, 2000);
  };

  const handleSubmit = () => {
    const correctCount = questions.filter((q) => answers[q.id] === q.answer).length;
    const wrongCount = Object.keys(answers).length - correctCount;
    const attempted = Object.keys(answers).length;
    const marks = ((correctCount - wrongCount * negative) / questions.length) * 100;

    setAttemptHistory((prev) => [
      ...prev,
      { date: new Date().toLocaleString(), marks: marks.toFixed(2) },
    ]);
    setSubmitted(true);
  };

  const correctCount = questions.filter((q) => answers[q.id] === q.answer).length;
  const wrongCount = Object.keys(answers).length - correctCount;
  const attempted = Object.keys(answers).length;
  const marks = ((correctCount - wrongCount * negative) / questions.length) * 100;

  // ---- Front Page ----
  if (!started) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-[url('/bowarrow.svg')] bg-cover bg-fixed p-4">
        <div className="bg-white rounded-2xl shadow-xl p-6 max-w-md w-full relative">
          <div className="absolute top-2 right-2 text-xs text-gray-600">
            Attempts: {attemptHistory.length}
            {attemptHistory.length > 0 && (
              <div>Last: {attemptHistory[attemptHistory.length - 1].marks} Marks</div>
            )}
          </div>
          <h1 className="text-xl font-bold mb-4 text-center">अर्जुन दृष्टि 🏹</h1>
          <p className="mb-1">Paper: {selectedTest}</p>
          <p className="mb-1">Duration: {duration} Minutes</p>
          <p className="mb-1">Total Questions: {totalQuestions}</p>
          <p className="mb-1">Total Marks: 100</p>
          <p className="mb-4">Negative Marking: 1/4 (-0.25)</p>
          <button
            onClick={() => setStarted(true)}
            className="px-6 py-3 bg-blue-600 text-white rounded-xl shadow hover:opacity-90 w-full"
          >
            Start Test
          </button>
        </div>
      </div>
    );
  }

  // ---- Result Page ----
  if (submitted) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-[url('/bowarrow.svg')] bg-cover bg-fixed p-4">
        <div className="bg-white rounded-2xl shadow-xl p-6 max-w-lg w-full text-center">
          <h2 className="text-xl font-bold mb-4">Result</h2>
          <p>Attempted: {attempted} / {questions.length}</p>
          <p className="text-green-600">Correct: {correctCount}</p>
          <p className="text-red-600">Wrong: {wrongCount}</p>
          <p className="text-blue-600 font-semibold">Marks: {marks.toFixed(2)} / 100</p>
          {marks === 100 && <p>🌟 Special Congratulations 🎊 Perfect Score!</p>}
          {marks >= 80 && marks < 100 && <p>🎉 Congratulations! Great Job 🎉</p>}
          {marks < 80 && <p>🙏 और मेहनत करें, अगली बार और बेहतर!</p>}
        </div>
      </div>
    );
  }

  // ---- Questions Page ----
  const q = questions[currentQ];
  const chosen = answers[q.id];

  return (
    <div className="min-h-screen bg-[url('/background.jpg')] bg-cover bg-fixed p-4">
      {/* Header */}
      <div className="flex justify-between items-center mb-4">
        <div className="font-bold">{selectedTest}</div>
        <div>⏱ {Math.floor(timeLeft / 60)}:{("0" + (timeLeft % 60)).slice(-2)}</div>
      </div>

      {/* Navigator */}
      <div className="flex flex-wrap gap-2 mb-4">
        {questions.map((qq, idx) => {
          let color = "bg-red-400";
          if (answers[qq.id]) color = "bg-green-500";
          if (idx === currentQ) color = "bg-blue-500";
          return (
            <div
              key={qq.id}
              className={`w-7 h-7 rounded-full flex items-center justify-center text-white text-xs cursor-pointer ${color}`}
              onClick={() => setCurrentQ(idx)}
            >
              {qq.id}
            </div>
          );
        })}
      </div>

      {/* Question */}
      <div className="bg-white rounded-2xl shadow p-4">
        <h2 className="font-semibold mb-3">
          {q.id}. {q.text}
        </h2>
        <div className="grid grid-cols-1 sm:grid-cols-2 gap-3">
          {q.options.map((opt) => {
            let state = "border-gray-300 hover:border-gray-400 bg-gray-50";
            if (chosen) {
              if (opt.key === q.answer) state = "border-green-600 bg-green-50";
              if (chosen === opt.key && chosen !== q.answer)
                state = "border-red-600 bg-red-50";
            }
            return (
              <div
                key={opt.key}
                onClick={() => handleSelect(q.id, opt.key)}
                className={`border rounded-xl p-3 cursor-pointer select-none transition ${state}`}
              >
                <div className="flex items-center gap-2">
                  <span className="inline-flex items-center justify-center w-7 h-7 rounded-lg bg-white border text-sm font-medium">
                    {opt.key.toUpperCase()}
                  </span>
                  <span className="text-sm font-medium">{opt.label}</span>
                </div>
              </div>
            );
          })}
        </div>
      </div>

      {/* Controls */}
      <div className="flex justify-between mt-4">
        <button
          disabled={currentQ === 0}
          onClick={() => setCurrentQ((c) => c - 1)}
          className="px-4 py-2 bg-gray-300 rounded-lg disabled:opacity-50"
        >
          Back
        </button>
        <button
          onClick={handleSubmit}
          className="px-4 py-2 bg-blue-600 text-white rounded-lg"
        >
          Submit
        </button>
        <button
          disabled={currentQ === questions.length - 1}
          onClick={() => setCurrentQ((c) => c + 1)}
          className="px-4 py-2 bg-gray-300 rounded-lg disabled:opacity-50"
        >
          Next
        </button>
      </div>
    </div>
  );
}
