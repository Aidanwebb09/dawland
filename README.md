let audioElements = [];

document.getElementById('fileInput').addEventListener('change', function(event) {
  const files = event.target.files;
  const container = document.getElementById('tracksContainer');
  container.innerHTML = '';

  audioElements = [];

  for (let file of files) {
    const trackDiv = document.createElement('div');
    trackDiv.className = 'track';

    const audio = new Audio();
    audio.src = URL.createObjectURL(file);
    audio.controls = true;
    audio.preload = 'metadata';

    const volumeControl = document.createElement('input');
    volumeControl.type = 'range';
    volumeControl.min = 0;
    volumeControl.max = 1;
    volumeControl.step = 0.01;
    volumeControl.value = 1;
    volumeControl.className = 'volume';

    volumeControl.addEventListener('input', () => {
      audio.volume = volumeControl.value;
    });

    const canvas = document.createElement('canvas');
    drawWaveform(audio.src, canvas);

    const controlsDiv = document.createElement('div');
    controlsDiv.className = 'track-controls';
    controlsDiv.appendChild(audio);
    controlsDiv.appendChild(volumeControl);

    trackDiv.appendChild(controlsDiv);
    trackDiv.appendChild(canvas);
    container.appendChild(trackDiv);

    audioElements.push(audio);
  }
});

function playAll() {
  for (let audio of audioElements) {
    audio.play();
  }
}

function pauseAll() {
  for (let audio of audioElements) {
    audio.pause();
  }
}

function drawWaveform(src, canvas) {
  const ctx = canvas.getContext('2d');
  const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  fetch(src)
    .then(response => response.arrayBuffer())
    .then(arrayBuffer => audioCtx.decodeAudioData(arrayBuffer))
    .then(audioBuffer => {
      const rawData = audioBuffer.getChannelData(0);
      const samples = 500;
      const blockSize = Math.floor(rawData.length / samples);
      const filteredData = [];
      for (let i = 0; i < samples; i++) {
        let sum = 0;
        for (let j = 0; j < blockSize; j++) {
          sum += Math.abs(rawData[i * blockSize + j]);
        }
        filteredData.push(sum / blockSize);
      }
      
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.fillStyle = '#4caf50';
      const height = canvas.height;
      filteredData.forEach((val, i) => {
        const x = (canvas.width / samples) * i;
        const y = height - val * height;
        ctx.fillRect(x, y, 2, val * height);
      });
    });
}
