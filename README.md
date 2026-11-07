[index.html](https://github.com/user-attachments/files/34519411/index.html)

<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>語音時間計算器</title>
<style>
* { box-sizing: border-box; }

body {
  font-family: Arial, sans-serif;
  background: #f0f2f5;
  margin: 0;
  padding: 15px;
}

.container {
  background: white;
  padding: 22px;
  border-radius: 12px;
  max-width: 420px;
  margin: auto;
  box-shadow: 0 4px 12px #0001;
}

h2 { text-align: center; }

button {
  width: 100%;
  padding: 15px;
  font-size: 18px;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  margin-top: 12px;
}

#mic {
  background: #4CAF50;
  color: white;
}

#calculate {
  background: #1976d2;
  color: white;
}

#result {
  white-space: pre-line;
  font-size: 17px;
  margin-top: 20px;
  line-height: 1.8;
}

#voiceText {
  background: #f0f2f5;
  padding: 12px;
  border-radius: 8px;
  min-height: 45px;
}
</style>
</head>

<body>
<div class="container">

<h2>語音時間計算器</h2>

<p>請按下麥克風，一次說出上下班時間。</p>

<button id="mic" onclick="startVoice()">🎤 開始語音輸入</button>

<p>辨識內容：</p>
<div id="voiceText">尚未輸入</div>

<button id="calculate" onclick="calculate()">計算總分鐘</button>

<div id="result"></div>

</div>

<script>
let recognizedText = "";

function startVoice() {
  const SpeechRecognition =
    window.SpeechRecognition ||
    window.webkitSpeechRecognition;

  if (!SpeechRecognition) {
    alert("目前瀏覽器不支援語音辨識，請使用 Chrome。");
    return;
  }

  const recognition = new SpeechRecognition();

  recognition.lang = "zh-TW";
  recognition.continuous = false;
  recognition.interimResults = false;

  document.getElementById("voiceText").innerText =
    "正在聆聽，請說出上下班時間...";

  recognition.onresult = function(event) {
    recognizedText = event.results[0][0].transcript;
    document.getElementById("voiceText").innerText =
      recognizedText;
  };

  recognition.onerror = function() {
    document.getElementById("voiceText").innerText =
      "語音辨識失敗，請重新輸入。";
  };

  recognition.start();
}

// 中文數字轉阿拉伯數字
function chineseNumber(s) {
  const digits = {
    "零":0, "一":1, "二":2, "兩":2,
    "三":3, "四":4, "五":5,
    "六":6, "七":7, "八":8, "九":9
  };

  if (/^\d+$/.test(s)) return Number(s);

  if (s === "十") return 10;

  if (s.includes("十")) {
    const p = s.split("十");
    return (p[0] ? digits[p[0]] : 1) * 10 +
      (p[1] ? digits[p[1]] : 0);
  }

  let n = 0;
  for (const c of s) {
    if (digits[c] !== undefined) n = n * 10 + digits[c];
  }
  return n;
}

// 擷取時間
function parseTimes(text) {
  text = text.replace(/\s/g, "");

  const regex =
    /(上午|早上|中午|下午|晚上)?(\d{1,2}|[零一二兩三四五六七八九十]+)(?:點|時)(?:(\d{1,2}|[零一二兩三四五六七八九十]+)分?)?/g;

  const matches = [...text.matchAll(regex)];

  if (matches.length < 2) return null;

  return matches.slice(0, 2).map(m => {
    let hour = chineseNumber(m[2]);
    let minute = m[3] ? chineseNumber(m[3]) : 0;
    const period = m[1];

    if (period === "下午" || period === "晚上") {
      if (hour < 12) hour += 12;
    } else if (period === "中午" && hour < 11) {
      hour += 12;
    } else if ((period === "上午" || period === "早上") && hour === 12) {
      hour = 0;
    }

    return hour * 60 + minute;
  });
}

function calculate() {
  if (!recognizedText) {
    alert("請先進行語音輸入！");
    return;
  }

  const times = parseTimes(recognizedText);

  if (!times) {
    document.getElementById("result").innerText =
      "無法辨識兩個完整時間，請重新說一次。";
    return;
  }

  const [startTotal, endTotal] = times;

  let diff = endTotal - startTotal;

  if (diff < 0) diff += 1440;

  const specialRanges = [
    [600, 610, 10],
    [900, 910, 10],
    [730, 780, 50]
  ];

  let deducted = 0;

  for (const [s, e, d] of specialRanges) {
    // 計算跨日區間重疊
    for (const offset of [0, 1440]) {
      if (startTotal < e + offset &&
          endTotal + (endTotal < startTotal ? 1440 : 0) > s + offset) {
        deducted += d;
        break;
      }
    }
  }

  if (endTotal > 1030) deducted += 20;

  const finalDiff = Math.max(0, diff - deducted);

  document.getElementById("result").innerText =
    "開始時間：" +
    formatTime(startTotal) +
    "\n結束時間：" +
    formatTime(endTotal) +
    "\n原始時間：" + diff + " 分鐘" +
    "\n扣除時間：" + deducted + " 分鐘" +
    "\n計算後總時間：" + finalDiff + " 分鐘";
}

function formatTime(t) {
  return String(Math.floor(t / 60) % 24).padStart(2, "0") +
    ":" +
    String(t % 60).padStart(2, "0");
}
</script>

</body>
</html>
