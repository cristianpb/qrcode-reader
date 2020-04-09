<main>
  <h1>jsQR Reader</h1>
  <div id="loadingMessage">🎥 Unable to access video stream (please make sure you have a webcam enabled)</div>
  <canvas id="canvas" hidden></canvas>
  {#if ready}
    <div class="output">
      {#if message}
        <p>{message}</p>
      {:else}
        <p>No QR code detected.</p>
      {/if}
    </div>
  {/if}
  {#if hoursDiff > 2}
    <p class="output">More than {hoursDiff} from now</p>
  {/if}
</main>

<script>
  import jsQR from "jsqr";
  import { onMount } from 'svelte';

  let message
  let ready
  let hoursDiff

  onMount(async() => {
    var video = document.createElement("video");
    var canvasElement = document.getElementById("canvas");
    var canvas = canvasElement.getContext("2d");
    var loadingMessage = document.getElementById("loadingMessage");

    function drawLine(begin, end, color) {
      canvas.beginPath();
      canvas.moveTo(begin.x, begin.y);
      canvas.lineTo(end.x, end.y);
      canvas.lineWidth = 4;
      canvas.strokeStyle = color;
      canvas.stroke();
    }

    // Use facingMode: environment to attemt to get the front camera on phones
    navigator.mediaDevices.getUserMedia({ video: { facingMode: "environment" } }).then(function(stream) {
      video.srcObject = stream;
      video.setAttribute("playsinline", true); // required to tell iOS safari we don't want fullscreen
      video.play();
      requestAnimationFrame(tick);
    });

    function tick() {
      loadingMessage.innerText = "⌛ Loading video..."
      if (video.readyState === video.HAVE_ENOUGH_DATA) {
        loadingMessage.hidden = true;
        canvasElement.hidden = false;
        ready = true;

        canvasElement.height = video.videoHeight;
        canvasElement.width = video.videoWidth;
        canvas.drawImage(video, 0, 0, canvasElement.width, canvasElement.height);
        var imageData = canvas.getImageData(0, 0, canvasElement.width, canvasElement.height);
        var code = jsQR(imageData.data, imageData.width, imageData.height, {
          inversionAttempts: "dontInvert",
        });
        if (code) {
          drawLine(code.location.topLeftCorner, code.location.topRightCorner, "#FF3B58");
          drawLine(code.location.topRightCorner, code.location.bottomRightCorner, "#FF3B58");
          drawLine(code.location.bottomRightCorner, code.location.bottomLeftCorner, "#FF3B58");
          drawLine(code.location.bottomLeftCorner, code.location.topLeftCorner, "#FF3B58");
          message = code.data;
          console.log(code.data.split(";"))
          let array = code.data.split(";")
          let nom = array[1].split(":")[1].trim();
          let prenom = array[2].split(":")[1].trim();
          let naissance = array[3].split(":")[1].trim();
          let sortie = array[5].split(":")[1].trim();
          let sortieDate = new Date(sortie.substring(6,10), sortie.substring(3,5) - 1, sortie.substring(0,2), sortie.substring(13,15), sortie.substring(16,18))
          hoursDiff = (new Date() - sortieDate)/( 3600  * 1000 )

        } 
      }
      requestAnimationFrame(tick);
    }

  })

</script>


<style>
  body {
    font-family: 'Ropa Sans', sans-serif;
    color: #333;
    max-width: 640px;
    margin: 0 auto;
    position: relative;
  }

  #githubLink {
    position: absolute;
    right: 0;
    top: 12px;
    color: #2D99FF;
  }

  h1 {
    margin: 10px 0;
    font-size: 40px;
  }

  #loadingMessage {
    text-align: center;
    padding: 40px;
    background-color: #eee;
  }

  #canvas {
    width: 100%;
  }

  .output {
    margin-top: 20px;
    background: #eee;
    padding: 10px;
    padding-bottom: 0;
  }

  #noQRFound {
    text-align: center;
  }
</style>
