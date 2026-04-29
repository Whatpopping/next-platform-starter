# Robust Discord AI Co-Pilot Setup

This guide describes a production-style approach for a Discord bot that can:
- join a voice channel,
- listen to you,
- understand your PC screen through a local desktop agent,
- and respond live with spoken commentary.

## 1) Core constraints (important)

A Discord bot cannot directly read your desktop. You need a **local PC agent** running on the machine whose screen you want analyzed.

Recommended model:
1. Phone controls Discord bot with slash commands.
2. Discord bot orchestrates sessions.
3. Local PC agent streams frames/audio to backend.
4. AI backend performs STT + vision + reasoning + TTS.
5. Bot plays generated speech in voice channel.

## 2) Reference architecture

### Components
- **Discord bot service** (Node.js + `discord.js` + `@discordjs/voice`)
- **Realtime backend** (WebSocket server)
- **AI pipeline**
  - STT (speech-to-text)
  - Vision model (screen understanding)
  - LLM (reasoning + response planning)
  - TTS (voice output)
- **Desktop agent** (Windows/macOS/Linux)
  - Screen capture (`mss` or platform APIs)
  - Optional mic capture
  - Event throttling and privacy filters
- **State store**
  - Redis for session locks, pub/sub, short-term context

### Recommended stream design
- Voice inbound: Opus -> PCM -> STT segments (VAD-gated)
- Screen inbound: JPEG keyframes every 1-2s, plus "change event" frames
- Context builder: rolling summary + current frame tags + recent transcript
- Voice outbound: short TTS segments (2-8 seconds each)

## 3) Latency and reliability targets

Set measurable SLOs:
- P95 user speech end -> bot speech start: **< 1.8 seconds**
- Frame capture -> model insight availability: **< 1.2 seconds**
- Session drop rate per hour: **< 1%**

Use:
- WebSocket heartbeats every 5 seconds
- Backpressure queue limits
- Circuit breaker around model calls
- Automatic degrade mode (text-only if TTS fails)

## 4) Security and privacy hardening

Minimum controls:
- Session auth token signed by backend (short TTL)
- TLS everywhere
- Per-session encryption key for agent uploads
- Allowlist of capture scopes:
  - active window only,
  - selected monitor,
  - selected application
- Sensitive-region redaction (password fields, notifications, OTP windows)
- Explicit "streaming on" indicator in desktop agent UI
- Audit log without storing raw frames by default

Policy suggestions:
- Default retention: 0 raw frames, 0 raw audio
- Keep only derived summaries for 24 hours (configurable)
- Require explicit consent before enabling recording

## 5) Cost control strategy

To avoid runaway AI cost:
- Dynamic frame cadence:
  - idle UI: 1 frame/3s
  - active changes: up to 1 frame/s
- Use prompt caching for stable UI contexts
- Summarize transcript every 8-12 turns
- Cap response length (e.g., 60 words unless asked for details)
- Per-session budget guardrails

## 6) Recommended commands

- `/ai start` - create session, join voice, request agent link code
- `/ai link <code>` - bind desktop agent to Discord session
- `/ai mode commentary|assistant|silent-watch`
- `/ai privacy window|monitor|app`
- `/ai rate low|medium|high`
- `/ai stop`

## 7) Failure-mode handling

Plan for these cases:
- Agent disconnect -> bot announces reconnection attempt for 30s
- Vision model timeout -> fallback to last known frame summary
- STT degraded -> switch to push-to-talk command mode
- Discord voice reconnect -> preserve session context in Redis

## 8) Implementation skeleton

### Phase 1 (MVP)
- Slash commands + voice join/leave
- Desktop agent sends frame every 2 seconds
- Basic STT -> LLM -> TTS loop

### Phase 2 (robust)
- VAD + interrupt handling (barge-in)
- Privacy redaction pipeline
- Rolling context summarizer
- Metrics dashboard (latency, errors, cost/session)

### Phase 3 (production)
- Multi-region backend
- Queue-based model workers
- Autoscaling policies
- On-call alerts + runbooks

## 9) Testing checklist

### Functional
- Command lifecycle (`start`, `link`, `mode`, `stop`)
- Screen commentary correctness on common workflows
- Voice interruption and resume

### Load
- 10/50/100 simultaneous sessions
- Soak test for 4+ hours

### Security
- Invalid token rejection
- Replay attack checks
- Scope-escape attempts (agent must not capture disallowed apps)

### Privacy
- Confirm redaction on known sensitive dialogs
- Verify retention policies actually delete content

## 10) What to build first

If you want the fastest path:
1. Build the desktop agent + backend WebSocket sessioning first.
2. Add Discord voice playback second.
3. Add privacy controls before scaling beyond personal use.

That order gives a working vertical slice quickly while avoiding unsafe defaults.
