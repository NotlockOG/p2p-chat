# p2p-secure-chat

**Ultra-Secure P2P Group Chat & Image Sharing (No Server, End-to-End Encrypted)**

A simple, single-file, browser-based chat and image sharing tool using WebRTC mesh with X25519 and AES-GCM per-link encryption. No central server or relay. Strong cryptography. Instant friend codes.

## Features

- **Zero server**: Peer-to-peer over WebRTC data channels (no relay, no sniffable traffic)
- **Very strong encryption**: X25519 key agreement + AES-GCM per link. Everything is end-to-end encrypted.
- **Multi-user mesh**: Paste friend codes to connect everyone, group and private messages/images.
- **No install**: 100% static HTML+JS, works on modern desktop browsers (2023+)
- **No persistence**: All in browser memory only

## Usage

1. Each user opens `index.html` (can be run locally, no server needed).
2. Each sees their "join code" right away. Share these (securely!) with all group members.
3. Each user pastes all *other* join codes in their box, clicks Connect, and sends the "reply code" back to each originator.
4. Paste replies as needed, until all say "connected."
5. Start chatting and sharing images! Every message and image will be instantly, strongly encrypted and transmitted peer-to-peer.

> For full details and instructions, see the usage guide included inside the app.

## Security

- Keys never leave your browser.
- Every peer-to-peer connection uses a unique X25519 ECDH key agreement with AES-GCM symmetric encryption.
- No central relay or transport server—no Internet metadata leaks except to STUN (sphere of NAT traversal).
- Codes are ephemeral session keys; once gone, chat is lost forever.

## License

MIT
