<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Sender</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      padding: 1rem;
      margin: 0;
      box-sizing: border-box;
      display: flex;
      flex-direction: column;
      gap: 1rem;
      background-color: #f8f9fa;
    }

    h2, h3 {
      margin: 0.5rem 0;
    }

    video {
      width: 100%;
      max-width: 600px;
      height: auto;
      background-color: black;
      border-radius: 8px;
    }

    textarea {
      width: 50%;
      height: 100px;
      padding: 0.5rem;
      font-family: monospace;
      font-size: 0.9rem;
      resize: vertical;
    }

    button {
      padding: 0.6rem 1rem;
      font-size: 1rem;
      margin-top: 0.5rem;
      cursor: pointer;
      border: none;
      background-color: #007bff;
      color: white;
      border-radius: 4px;
	width:15%;
    }

    button:hover {
      background-color: #0056b3;
    }

    .offer-section {
      position: relative;
    }

    .copy-btn {
      position: absolute;
      right: 0.5rem;
      top: 0.5rem;
      padding: 0.3rem 0.6rem;
      font-size: 0.8rem;
      background-color: #28a745;
	width:5%;
    }

    .copy-btn:hover {
      background-color: #1e7e34;
    }

    @media (max-width: 600px) {
      button, .copy-btn {
        width: 100%;
        margin-top: 0.5rem;
      }
    }
  </style>
</head>
<body>
  <h2>Phone 1 - Start Streaming</h2>
  <video id="localVideo" autoplay playsinline></video>
  <button id="startButton">Start Streaming</button>

  <h3>Offer To Reciver</h3>
  <div class="offer-section">
    <textarea id="offerText" readonly></textarea>
    <button class="copy-btn" onclick="copyOffer()">Copy</button>
  </div>

  <h3>Answer From Reciver</h3>
  <textarea id="answerInput" placeholder="Paste SDP Answer from Phone 2"></textarea>
  <button id="connectButton">Connect with SDP Answer</button>

  <script>
    const localVideo = document.getElementById('localVideo');
    const startButton = document.getElementById('startButton');
    const offerText = document.getElementById('offerText');
    const answerInput = document.getElementById('answerInput');
    const connectButton = document.getElementById('connectButton');
    let localStream;
    let peerConnection;

    const iceServers = {
      iceServers: [{ urls: 'stun:stun.l.google.com:19302' }]
    };

    startButton.addEventListener('click', startStreaming);
    connectButton.addEventListener('click', connectWithAnswer);

    function startStreaming() {
      navigator.mediaDevices.getUserMedia({ video: true, audio: true })
        .then((stream) => {
          localStream = stream;
          localVideo.srcObject = localStream;
          createPeerConnection();
        })
        .catch((err) => {
          console.error('❌ Error accessing media devices:', err);
        });
    }

    function createPeerConnection() {
      peerConnection = new RTCPeerConnection(iceServers);

      localStream.getTracks().forEach(track => {
        peerConnection.addTrack(track, localStream);
        console.log("📡 Sending track:", track.kind);
      });

      peerConnection.onicecandidate = (event) => {
        if (event.candidate) {
          console.log("📡 ICE Candidate:", event.candidate);
        }
      };

      peerConnection.createOffer()
        .then((offer) => peerConnection.setLocalDescription(offer))
        .then(() => {
          offerText.value = JSON.stringify(peerConnection.localDescription);
          console.log("📩 Offer for Phone 2:", peerConnection.localDescription);
        })
        .catch((err) => {
          console.error("❌ Error creating offer:", err);
        });

      peerConnection.onconnectionstatechange = () => {
        console.log("🔄 Connection state:", peerConnection.connectionState);
      };

      peerConnection.oniceconnectionstatechange = () => {
        console.log("❄️ ICE state:", peerConnection.iceConnectionState);
      };
    }

    function connectWithAnswer() {
      if (!peerConnection) {
        console.error("❌ Peer connection is not initialized");
        return;
      }

      const answer = JSON.parse(answerInput.value);

      if (peerConnection.signalingState !== "have-local-offer") {
        console.warn("⚠️ Cannot set remote description in current state:", peerConnection.signalingState);
        return;
      }

      peerConnection.setRemoteDescription(new RTCSessionDescription(answer))
        .then(() => {
          console.log("✅ SDP Answer received and set as remote description");
        })
        .catch((err) => {
          console.error("❌ Error setting SDP Answer:", err);
        });
    }

    function copyOffer() {
      offerText.select();
      document.execCommand("copy");
    }
  </script>
</body>
</html>
