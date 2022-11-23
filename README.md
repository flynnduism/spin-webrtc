# spin-webrtc

This is a [Spin](https://github.com/fermyon/spin) app for doing
browser-to-browser WebRTC video calls.  It uses
[websocket-bridge](https://github.com/fermyon/websocket-bridge) and
[Redis](https://redis.io) to establish indirect connections between peers, which
the client-side JS code uses to exchange WebRTC signaling messages (e.g. SDP and
ICE handshakes) and thereby establish direct connections for exchanging audio
and video data.

## Building and Running

### Prerequisites

- Rust
- Spin (recent enough to include [this PR](https://github.com/fermyon/spin/pull/915))
- [websocket-bridge](https://github.com/fermyon/websocket-bridge)
- Redis server (or use a free [redislabs.com](https://redislabs.com) account)
- A TLS cert your browser will accept (e.g. one from [letsencrypt.org](https://letsencrypt.org))
 - You may need two of these if you run `websocket-bridge` and `spin` on separate servers

First, start `websocket_bridge`, configuring it to allow traffic to the Spin
server we'll run in the next step.  Note that the `--whitelist` option takes a
regular expression (and can be specified multiple times), so be sure to escape
any dots in the name (e.g. "foo.example.com" -> "foo\.example\.com").

```
RUST_LOG=info websocket_bridge \
    --address 0.0.0.0:9443 \
    --base-url https://$YOUR_WEBSOCKET_BRIDGE_SERVER:9443 \
    --cert $PATH_TO_YOUR_WEBSOCKET_BRIDGE_TLS_CERT \
    --key $PATH_TO_YOUR_WEBSOCKET_BRIDGE_TLS_CERT \
    --whitelist 'https://$YOUR_SPIN_SERVER_WITH_DOTS_ESCAPED:9534/.*'
```

Next, build and run this app using Spin, specifying the URL of your Redis
server, and also editing static/index.js to point to your `websocket-bridge`
server (TODO: provide a more elegant way to configure this, e.g. via a Cargo
build script).

```
sed -i 's/\[insert your websocket-bridge server here\]/'$YOUR_WEBSOCKET_BRIDGE_SERVER':9443/' \
    static/index.js
export REDIS_URL=redis://$YOUR_REDIS_SERVER
spin build
spin up \
    --follow-all \
    --listen 0.0.0.0:443 \
    --tls-cert $PATH_TO_YOUR_SPIN_TLS_CERT \
    --tls-key $PATH_TO_YOUR_SPIN_TLS_CERT
```

Finally, visit `https://$YOUR_SPIN_SERVER/` in a modern browser on a couple of
devices, and you should have a video call running.  If anything goes wrong,
please file an issue in this repo!