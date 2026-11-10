# Fun
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>KEYBOARD CHAOS</title>
<style>
*{box-sizing:border-box}
body{
 margin:0;background:#10111a;color:white;
 font-family:Arial,sans-serif;text-align:center;
 padding:20px;min-height:100vh
}
header{padding:12px;border-bottom:1px solid #33364a}
h1{letter-spacing:4px;font-size:29px}
header p{color:#929bb2;font-size:12px;letter-spacing:2px}
.game{max-width:480px;margin:25px auto}
.face{font-size:95px;height:115px;transition:.3s}
.shake{animation:shake .3s infinite}
@keyframes shake{
 25%{transform:rotate(-7deg)}
 75%{transform:rotate(7deg)}
}
.speech{
 background:#222638;padding:15px;border-radius:14px;
 margin:10px auto 20px;color:#dce5fa;font-weight:bold
}
.panel{
 background:#1b2030;padding:22px;
 border:1px solid #353d56;border-radius:17px
}
.label{font-size:12px;color:#99a7c5;letter-spacing:2px}
#round{margin:10px;color:#76d8ff;font-weight:bold}
input{
 width:100%;padding:16px;border-radius:10px;
 border:2px solid #46516e;background:#111522;
 color:white;font-size:19px;text-align:center;
 outline:none;margin-top:18px
}
input:focus{border-color:#76d8ff}
button{
 padding:14px 20px;border:0;border-radius:10px;
 font-weight:bold;cursor:pointer;margin:7px 3px
}
.blue{background:#76d8ff;color:#101522}
.grey{background:#303951;color:white}
#submit{width:100%;margin-top:12px}
.meter{
 height:12px;background:#343b50;
 border-radius:20px;overflow:hidden;margin:10px 0
}
#bar{height:100%;width:0%;background:#f06a69;transition:.3s}
#level{font-size:13px;color:#ff9e9e}
#status{min-height:40px;margin-top:15px;color:#ffaaaa}
.small{font-size:12px;color:#8996b2;margin-top:25px}
</style>
</head>
<body>

<header>
<h1>KEYBOARD CHAOS</h1>
<p>LISTEN CAREFULLY. TYPE CORRECTLY.</p>
</header>

<div class="game">

<div class="face" id="face">😐</div>
<div class="speech" id="speech">
"Welcome. Listen to the word and type it."
</div>

<div class="panel">
<div class="label">CHALLENGE</div>
<div id="round">ROUND 1</div>

<button class="blue" onclick="speakWord()">🔊 HEAR WORD</button>
<button class="grey" onclick="speakWord()">🔁 REPLAY</button>

<input id="typing" placeholder="Type what you heard..."
autocomplete="off" spellcheck="false">

<button class="blue" id="submit" onclick="checkAnswer()">
SUBMIT ANSWER
</button>

<div id="status"></div>

<div class="label">CHARACTER ANNOYANCE</div>
<div class="meter"><div id="bar"></div></div>
<div id="level">0%</div>

</div>

<button class="grey" onclick="restart()">RESTART GAME</button>

<p class="small">Headphones recommended. Good luck. 💀</p>
</div>

<script>
const words=[
"APPLE","BANANA","DINOSAUR","ASTRONAUT",
"ELEPHANT","POTATO","COMPUTER","GALAXY",
"CHOCOLATE","BUTTERFLY","PINEAPPLE",
"VOLCANO","SPAGHETTI","TELESCOPE",
"UNIVERSE","WATERMELON","OCTOPUS",
"MICROSCOPE","PENGUIN","EXTRAORDINARY"
];

const reactions=[
"Nice! You actually listened.",
"Not bad. Keep going.",
"Are you even listening?",
"THAT IS NOT WHAT I SAID.",
"PLEASE LISTEN TO ME.",
"WHAT THE HECK IS WRONG WITH YOU?!",
"I'M LOSING MY PATIENCE.",
"THIS IS ABSOLUTELY RIDICULOUS.",
"I QUIT. I ACTUALLY QUIT."
];

let answer="";
let round=1;
let annoyance=0;
let active=true;
let audio=null;
let speaking=false;

const face=document.getElementById("face");
const speech=document.getElementById("speech");
const typing=document.getElementById("typing");
const status=document.getElementById("status");
const bar=document.getElementById("bar");
const level=document.getElementById("level");

function beep(freq=400,duration=.12,type="sine"){
 try{
  audio=audio||new(window.AudioContext||window.webkitAudioContext)();
  if(audio.state==="suspended")audio.resume();
  const osc=audio.createOscillator();
  const gain=audio.createGain();
  osc.type=type;
  osc.frequency.value=freq;
  gain.gain.setValueAtTime(.08,audio.currentTime);
  gain.gain.exponentialRampToValueAtTime(.001,audio.currentTime+duration);
  osc.connect(gain);
  gain.connect(audio.destination);
  osc.start();
  osc.stop(audio.currentTime+duration);
 }catch(e){}
}

function chooseWord(){
 let next;
 do{
  next=words[Math.floor(Math.random()*words.length)];
 }while(next===answer&&words.length>1);
 answer=next;
}

function speakWord(){
 if(!active)return;
 if(!("speechSynthesis" in window)){
  status.textContent="Your browser does not support voice playback.";
  return;
 }

 speechSynthesis.cancel();

 const utterance=new SpeechSynthesisUtterance(answer.toLowerCase());
 utterance.lang="en-US";
 utterance.rate=.85;
 utterance.pitch=1;

 utterance.onstart=()=>{speaking=true};
 utterance.onend=()=>{speaking=false};
 speechSynthesis.speak(utterance);

 status.textContent="Listen carefully...";
 beep(550,.1);
}

function updateCharacter(){
 annoyance=Math.max(0,Math.min(100,annoyance));

 let emoji="😐";
 let reaction=reactions[0];

 if(annoyance>=15){emoji="🙂";reaction=reactions[1]}
 if(annoyance>=30){emoji="😑";reaction=reactions[2]}
 if(annoyance>=45){emoji="😒";reaction=reactions[3]}
 if(annoyance>=60){emoji="😠";reaction=reactions[4]}
 if(annoyance>=75){emoji="😡";reaction=reactions[5]}
 if(annoyance>=90){emoji="🤬";reaction=reactions[7]}

 face.textContent=emoji;
 face.classList.toggle("shake",annoyance>=75);
 speech.textContent='"'+reaction+'"';

 bar.style.width=annoyance+"%";
 level.textContent=annoyance+"%";
}

function checkAnswer(){
 if(!active)return;

 const guess=typing.value.trim().toUpperCase();

 if(!guess){
  status.textContent="TYPE SOMETHING FIRST!";
  beep(180,.2,"square");
  return;
 }

 if(guess===answer){
  annoyance=Math.max(0,annoyance-15);
  status.textContent="CORRECT! WELL DONE! 🎉";
  beep(750,.15);
  round++;
  document.getElementById("round").textContent="ROUND "+round;
  typing.value="";
  updateCharacter();

  setTimeout(()=>{
   if(active){
    chooseWord();
    speakWord();
   }
  },1200);

 }else{
  annoyance=Math.min(100,annoyance+20);
  status.textContent="WRONG! LISTEN AGAIN! 💀";
  typing.select();
  beep(130,.3,"sawtooth");
  updateCharacter();

  if(annoyance>=100)finish();
 }
}

function finish(){
 active=false;
 speechSynthesis.cancel();
 face.textContent="💀";
 face.classList.remove("shake");
 speech.textContent='"I AM DONE. GOODBYE."';
 status.textContent="GAME OVER. YOU BROKE THE LITTLE GUY.";
 typing.disabled=true;
 document.getElementById("submit").disabled=true;
}

function restart(){
 speechSynthesis.cancel();
 active=true;
 round=1;
 annoyance=0;
 typing.disabled=false;
 document.getElementById("submit").disabled=false;
 typing.value="";
 status.textContent="";
 document.getElementById("round").textContent="ROUND 1";
 face.textContent="😐";
 face.classList.remove("shake");
 speech.textContent='"Welcome. Listen to the word and type it."';
 updateCharacter();
 chooseWord();
 setTimeout(speakWord,500);
}

typing.addEventListener("keydown",e=>{
 if(e.key==="Enter")checkAnswer();
});

restart();
</script>
</body>
</html>
