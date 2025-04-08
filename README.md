<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Receiver</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      padding: 1rem;
      background-color: #f4f4f4;
      display: flex;
      flex-direction: column;
      gap: 1rem;
    }

    h2, h3 {
      margin: 0.5rem 0;
    }

    video {
      width: 100%;
      max-width: 700px;
      border: 2px solid #000;
      border-radius: 8px;
    }

    textarea {
      width: 50%;
      max-width: 700px;
      padding: 0.5rem;
      font-family: monospace;
      font-size: 0.9rem;
      resize: vertical;
      box-sizing: border-box;
      position: relative;
    }

    .section {
      position: relative;
    }

    .answer-section {
      position: relative;
    }

    .answer-section textarea {
      padding-top: 2rem; /* Make room for the button inside */
    }

    .copy-btn {
      position: absolute;
      top: 5px;
      right: 10px;
      padding: 0.3rem 0.6rem;
      font-size: 0.8rem;
      background-color: #28a745;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
      z-index: 2;
	width:5%;
    }

    .copy-btn:hover {
      background-color: #1e7e34;
    }

    button {
      margin-top: 0.5rem;
      padding: 0.6rem 1rem;
      font-size: 1rem;
      background-color: #007bff;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
	width:15%;
    }

    button:hover {
      background-color: #0056b3;
    }

    @media (max-width: 600px) {
      .copy-btn, button {
        width: auto;
        margin-top: 0.5rem;
      }
    }
  </style>
</head>
<body>
  <h2>Phone 2 - Start Receiving</h2>
  <video id="video" autoplay playsinline></video>

  <h3>Offer From Sender</h3>
  <div class="section">
    <textarea id="offer" rows="8" placeholder="Paste the SDP offer here..."></textarea>
  </div>
  <button onclick="createAnswer()">Create Answer</button>

  <h3>Answer To Sender</h3>
  <div class="answer-section">
    <button class="copy-btn" onclick="copyText('answer')">Copy</button>
    <textarea id="answer" rows="8" readonly></textarea>
  </div>

  <script>
    let peer = new RTCPeerConnection();

    peer.ontrack = event => {
      document.getElementById('video').srcObject = event.streams[0];
    };

    async function createAnswer() {
      const offerText = document.getElementById('offer').value;
      if (!offerText) return alert("Please paste the sender's offer.");

      const offer = JSON.parse(offerText);
      await peer.setRemoteDescription(offer);
      const answer = await peer.createAnswer();
      await peer.setLocalDescription(answer);

      // Wait for ICE gathering to complete
      peer.onicegatheringstatechange = () => {
        if (peer.iceGatheringState === 'complete') {
          document.getElementById('answer').value = JSON.stringify(peer.localDescription);
        }
      };
    }

    peer.onicecandidate = event => {
      if (event.candidate) {
        console.log("Receiver ICE Candidate:", event.candidate);
      }
    };

    function copyText(id) {
      const textarea = document.getElementById(id);
      textarea.select();
      textarea.setSelectionRange(0, 99999); // For mobile compatibility
      document.execCommand("copy");
    }
  </script>
</body>
</html>
