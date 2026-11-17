
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>My Personal Assistant</title>

<style>
* { box-sizing: border-box; }

body {
  margin: 0;
  min-height: 100vh;
  background: radial-gradient(circle, #12334b, #020811);
  color: #00eaff;
  font-family: Arial, sans-serif;
  text-align: center;
  padding: 25px 15px;
}

h1 {
  margin-top: 25px;
  letter-spacing: 3px;
  font-size: 27px;
}

.subtitle { color: #a9dce8; }

#circle {
  width: 155px;
  height: 155px;
  border: 3px solid #00eaff;
  border-radius: 50%;
  margin: 35px auto;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 65px;
  box-shadow: 0 0 25px #00eaff;
}

#circle.listening {
  box-shadow: 0 0 45px #00eaff;
  background: #073047;
}

button {
  padding: 14px 20px;
  margin: 8px;
  border: 1px solid #00eaff;
  border-radius: 12px;
  background: #10283b;
  color: white;
  font-size: 16px;
}

button:active { background: #07516b; }

input {
  width: 90%;
  max-width: 400px;
  padding: 15px;
  margin-top: 20px;
  border: 1px solid #00eaff;
  border-radius: 10px;
  background: #10283b;
  color: white;
  font-size: 16px;
}

#reply {
  color: white;
  background: #10283b;
  border: 1px solid #176078;
  border-radius: 12px;
  padding: 18px;
  margin: 25px auto;
  max-width: 450px;
  min-height: 60px;
  line-height: 1.5;
}

#status {
  color: #a9dce8;
  font-size: 14px;
}
</style>
</head>

<body>

<h1>MY AI ASSISTANT</h1>
<p class="subtitle">Your Personal Voice Assistant</p>

<div id="circle">🤖</div>

<p id="status">Assistant is ready.</p>

<button id="micButton" onclick="startListening()">
🎤 Speak
</button>

<button onclick="stopListening()">
⏹ Stop
</button>

<br>

<input
  id="message"
  type="text"
  placeholder="Type your message..."
  onkeydown="if(event.key === 'Enter') sendMessage()"
>

<br>

<button onclick="sendMessage()">Send</button>

<p id="reply">Hello! How can I help you?</p>

<script>

const replyBox = document.getElementById("reply");
const statusBox = document.getElementById("status");
const circle = document.getElementById("circle");
const micButton = document.getElementById("micButton");
const messageBox = document.getElementById("message");

let recognition = null;
let listening = false;

function speak(text) {
  if (!("speechSynthesis" in window)) {
    statusBox.innerText = "Speech output is not supported.";
    return;
  }

  speechSynthesis.cancel();

  const speech = new SpeechSynthesisUtterance(text);
  speech.lang = "en-IN";
  speech.rate = 1;
  speech.pitch = 1;

  speechSynthesis.speak(speech);
}

function showReply(text) {
  replyBox.innerText = text;
}

function respond(text) {
  const q = text.toLowerCase().trim();
  let answer = "";

  if (!q) return;

  if (q.includes("hello") || q.includes("hi")) {
    answer = "Hello! Nice to meet you. How can I help?";
  }
  else if (q.includes("your name")) {
    answer = "I am your personal assistant.";
  }
  else if (q.includes("time")) {
    answer = "The current time is " +
      new Date().toLocaleTimeString("en-IN");
  }
  else if (q.includes("date") || q.includes("today")) {
    answer = "Today's date is " +
      new Date().toLocaleDateString("en-IN");
  }
  else if (q.includes("how are you")) {
    answer = "I am working properly. Thank you for asking.";
  }
  else if (q.includes("who created you")) {
    answer = "You are creating me!";
  }
  else if (q.includes("thank")) {
    answer = "You are welcome!";
  }
  else if (q.includes("bye")) {
    answer = "Goodbye! Have a great day.";
  }
  else if (q.includes("help")) {
    answer = "You can ask me the time, date, my name, or say hello.";
  }
  else {
    answer = "I do not know the answer yet. My real AI connection has not been added.";
  }

  showReply(answer);
  speak(answer);
}

function sendMessage() {
  const text = messageBox.value.trim();

  if (!text) {
    showReply("Please type something first.");
    return;
  }

  respond(text);
  messageBox.value = "";
}

function startListening() {
  const SpeechRecognition =
    window.SpeechRecognition ||
    window.webkitSpeechRecognition;

  if (!SpeechRecognition) {
    showReply("Voice recognition is not supported in this browser. Try Google Chrome.");
    return;
  }

  if (!window.isSecureContext) {
    showReply("Microphone needs a secure page. Try Acode's local preview or a trusted HTTPS page.");
    return;
  }

  if (listening) {
    return;
  }

  try {
    recognition = new SpeechRecognition();

    recognition.lang = "en-IN";
    recognition.continuous = false;
    recognition.interimResults = false;
    recognition.maxAlternatives = 1;

    recognition.onstart = function() {
      listening = true;
      circle.classList.add("listening");
      micButton.innerText = "🎤 Listening...";
      statusBox.innerText = "Speak now...";
    };

    recognition.onresult = function(event) {
      const text = event.results[0][0].transcript;
      messageBox.value = text;
      statusBox.innerText = "You said: " + text;
      respond(text);
    };

    recognition.onerror = function(event) {
      const errors = {
        "not-allowed": "Microphone permission denied. Allow microphone access.",
        "service-not-allowed": "Speech recognition service is blocked.",
        "network": "Network error. Check your internet connection.",
        "no-speech": "No speech detected. Please try again.",
        "audio-capture": "Microphone not found or unavailable.",
        "language-not-supported": "This language is not supported."
      };

      showReply(errors[event.error] ||
        "Voice error: " + event.error);
    };

    recognition.onend = function() {
      listening = false;
      circle.classList.remove("listening");
      micButton.innerText = "🎤 Speak";
      statusBox.innerText = "Assistant is ready.";
    };

    recognition.start();

  } catch (error) {
    showReply("Could not start microphone: " + error.message);
    listening = false;
  }
}

function stopListening() {
  if (recognition && listening) {
    recognition.stop();
  }

  if ("speechSynthesis" in window) {
    speechSynthesis.cancel();
  }
}

</script>
</body>
</html>
# my-ai
my first ai on GitHub 
