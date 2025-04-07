<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Phone 1 - Streamer</title>
</head>
<body>
    <h2>Phone 1 - Start Streaming</h2>
    <video id="localVideo" autoplay playsinline></video>
    <button id="startButton">Start Streaming</button>

    <h3>Offer to Share with Phone 2 (QR or URL)</h3>
    <textarea id="offerText" readonly></textarea>

    <h3>Enter SDP Answer from Phone 2</h3>
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
    </script>
</body>
</html>
