<!DOCTYPE html>
<html lang="en">
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Mini FL Studio Autotune</title>
<style>
  body {background:#111;color:#fff;font-family:sans-serif;text-align:center;padding:10px;}
  h1 {margin-bottom:5px;}
  button, input[type=range]{margin:5px;padding:10px;font-size:16px;}
  .keyboard {display:flex;justify-content:center;margin-top:10px;}
  .key {width:40px;height:100px;background:#444;margin:1px;border-radius:4px;cursor:pointer;}
  .key.white:hover {background:#666;}
</style>
</head>
<body>
<h1>🎛 Mini FL Studio</h1>
<p>Record, autotune, and play notes</p>

<button id="recordBtn">🎙 Record (5s)</button>
<button id="playBtn">▶ Play</button><br>

<label>Autotune Intensity: <span id="pitchVal">1.0</span></label>
<input type="range" id="pitchSlider" min="0.5" max="2" value="1" step="0.01"><br>

<label>Playback Speed: <span id="speedVal">1.0</span></label>
<input type="range" id="speedSlider" min="0.5" max="2" value="1" step="0.01"><br>

<div class="keyboard">
  <div class="key white" data-note="261.63"></div>
  <div class="key white" data-note="293.66"></div>
  <div class="key white" data-note="329.63"></div>
  <div class="key white" data-note="349.23"></div>
  <div class="key white" data-note="392.00"></div>
  <div class="key white" data-note="440.00"></div>
  <div class="key white" data-note="493.88"></div>
</div>

<audio id="audio" controls></audio>

<script>
let chunks=[],recorder;
const audioEl=document.getElementById("audio");
const recordBtn=document.getElementById("recordBtn");
const playBtn=document.getElementById("playBtn");
const pitchSlider=document.getElementById("pitchSlider");
const speedSlider=document.getElementById("speedSlider");
const pitchVal=document.getElementById("pitchVal");
const speedVal=document.getElementById("speedVal");

recordBtn.onclick=async()=>{
  const stream=await navigator.mediaDevices.getUserMedia({audio:true});
  recorder=new MediaRecorder(stream);
  recorder.ondataavailable=e=>chunks.push(e.data);
  recorder.onstop=()=>{
    const blob=new Blob(chunks,{type:"audio/webm"});
    audioEl.src=URL.createObjectURL(blob);
    chunks=[];
  };
  recorder.start();
  setTimeout(()=>recorder.stop(),5000);
};

playBtn.onclick=()=>{
  audioEl.playbackRate=parseFloat(speedSlider.value);
  audioEl.play();
};

pitchSlider.oninput=()=>{pitchVal.textContent=pitchSlider.value;}
speedSlider.oninput=()=>{speedVal.textContent=speedSlider.value;}

// Mini keyboard
document.querySelectorAll(".key").forEach(k=>{
  k.onclick=()=>{
    const ctx=new (window.AudioContext||window.webkitAudioContext)();
    const osc=ctx.createOscillator();
    osc.type="sine";
    osc.frequency.value=parseFloat(k.dataset.note)*parseFloat(pitchSlider.value);
    osc.connect(ctx.destination);
    osc.start();
    osc.stop(ctx.currentTime+0.5);
  };
});
</script>
</body>
</html>
