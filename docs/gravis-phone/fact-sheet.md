# Gravis Phone — Research Fact Sheet (2026-10-14)
Sources: 3 research sub-agents (web, official docs). V = verified at source, U = unverified/inferred.

## User
Itai, Israeli private individual (not a business), Hebrew speaker. Uses claude.ai (web+mobile), Claude Code (CLI + cloud sessions), connectors (Gmail, Calendar, Drive, Make). "Gravis" = his Claude assistant persona (Hebrew, feminine grammar).
Goals: (1) Gravis calls Itai when it needs a decision, then continues working with the answer. (2) Gravis calls third parties in Israel on his behalf (businesses, service providers, friends) in natural Hebrew. (3) Maybe inbound later.
Trigger: reel by Riley Brown (Oct 12 2026): "give Claude Code a phone in 5 minutes with a Bland API key; agent calls you when it needs a decision, then keeps working with your answer."

## Platforms
- Bland: simplest (one POST with task; MCP create_call/wait_for_call; CLI --wait). NO Hebrew (no `he`; experimental "babel" only) [V]. No LLM choice — not Claude [V]. Own numbers US/CA only [V]. => out for Hebrew.
- Vapi: one POST /call with INLINE assistant (prompt per call) [V]. Official MCP https://mcp.vapi.ai/mcp (create_call, get_call, list_calls) + official skills [V]. Native Anthropic up to claude-sonnet-5 / haiku-4-5 (5.5 not listed yet) [V]; custom-llm [V]. Hebrew STT: Deepgram Nova-3 `he`, Soniox `he`, Azure he-IL [V]; TTS ElevenLabs v3/v4 Turbo [V]. DTMF, transferCall, voicemail detection [V]. ~$0.05/min platform + providers [V]. Free numbers US-only -> import Twilio/Telnyx/Zadarma IL number [V]. $5 free credit.
- Retell: he-IL in official language list [V]; native Claude incl. Sonnet 5.5 (sources disagree whether Haiku 5.5 is native) [V/U]. Hosted MCP https://mcp.retellai.com exposes only generic invoke_api_endpoint (opens ALL endpoints incl. delete) [V]. Needs saved agent + from_number; per-call agent_override [V]. Press-digit IVR, transfer, voicemail detection (first 3 min only) [V]. Retell-managed numbers cannot dial Israel -> bring Twilio/Telnyx with IL allowed [V]. $0.055/min + LLM + voice. $10 free.
- ElevenLabs Agents: strongest Hebrew TTS (Hebrew on v3 / v3 Conversational / v4 Turbo per one source; only v3 Conversational per another) [V, conflicting]. Scribe STT Hebrew "Good" (10–20% WER) [V]. Native Claude Sonnet 5.5 / Opus 5.5, Haiku only 4.5 [V]. Outbound via REST POST /v1/convai/twilio/outbound-call — needs pre-created agent_id + imported number; per-call overrides/dynamic variables [V]. Hosted MCP (OAuth, in Claude directory) manages agents/transcripts but NO documented place-call tool [V]. System tools: keypad DTMF, transfer_to_number, voicemail detection [V]. ~$0.08/min platform + LLM. Also outbound WhatsApp calls (needs WhatsApp Business number + recipient permission) [V].
- Self-built (Twilio ConversationRelay / LiveKit / Pipecat): highest ceiling; Soniox STT best on Israeli phone audio (13.6% WER vs OpenAI 61%, small vendor-adjacent study) [V/U]; Claude as brain; 1–3 days build + maintenance [U].
- Open-source "call me" for Claude Code: ZeframLou/call-me (2.6k stars; Claude Code itself is the brain; Telnyx/Twilio + ngrok + OpenAI STT/TTS -> weak Hebrew) [V]; sns45/better-call-claude; abracadabra50/claude-code-voice-skill [V].
- OpenAI Realtime: not Claude; outbound SIP limited; Hebrew weak on phone audio [V/U].

## Israeli numbers
- Israel blocks calls originating abroad that present an Israeli caller ID (General License regs §69א; mobile operators enforced since Feb 2025) [V]. A cloud platform presenting an Israeli number may be blocked/replaced unless the call enters the network in Israel [I]. Twilio IL caller ID: "Local, not guaranteed" [V].
- Telnyx Israeli local number + "Local Calling" (routes via in-country carriers, Israel listed) — from ~$3/mo; individual docs: name + address proof (<3 months) [V]. Stock not confirmed [U].
- Twilio IL numbers: local $5.50/mo, mobile $15/mo; to IL mobile $0.0646/min from IL number vs $0.1868/min from foreign number [V]. IL docs requirements not found [NF].
- Zadarma IL numbers $2–3/mo, ID + address; Vapi has a Zadarma guide [V]; outbound CLI survival U.
- Israeli licensed SIP trunk (e.g. Bezeq Business ₪199/mo) — strongest legal basis; likely requires business registration [U].
- US +1 number: simplest; Israelis advised not to answer unexpected foreign calls -> poor pickup for third parties; fine for calling Itai himself (he saves it as a contact) [V/I].
- Itai's own mobile as Twilio verified caller ID: likely blocked/replaced (foreign origin + Israeli mobile CLI) -> avoid [I].

## Legal (not legal advice)
- Recording: Israel = one-party consent; Itai as party via his agent likely OK (untested) [I]; safest: agent announces recording/transcription.
- AI disclosure: no binding Israeli law; Privacy Authority draft guidance (Apr 2025) says disclose [V]. Recommendation: always disclose ("עוזרת AI של איתי").
- Spam law §30A: only advertisements — personal calls outside [V/I]. Never use for promotion without written consent.
- Privacy Amendment 13: personal-use collection excluded from database duties [V]; regulator July 2026 advice on personal AI agents: minimal access, delete recordings/transcripts after summarising [V].
- Voice cloning own voice legal; never impersonate Itai (Penal Law impersonation/fraud risk) [V/I]. US numbers: FCC — AI voice needs prior express consent (TCPA) [V].

## Claude integration
- claude.ai custom connector (remote MCP) works on web, desktop, mobile (uses connectors already added), Claude Code CLI (auto when signed in), cloud sessions (connector traffic bypasses network allowlist) [V]. claude.ai per-tool-call limit 240 s -> can't block for a full call; create then poll [V].
- Skills: ZIP upload on claude.ai (needs code execution); ~/.claude/skills in Claude Code; claude.ai skills sync to Claude Code when signed in [V]. claude.ai sandbox calling APIs directly needs egress + allowlist (buggy) and has no secret store -> avoid there [V/I].
- Claude Code: script create -> poll -> JSON; Bash run_in_background up to 2 h in cloud; key via env var locally or "network secret" in cloud environments (proxy injects it; session never sees it) [V].
- In-call LLM: Haiku 5.5 fastest (low effort for voice); prefer native Claude option over Anthropic's OpenAI-compatible endpoint (not production-ready) [V].
- Risks: third-party transcript = untrusted data (prompt injection); every call costs money -> confirm before dialing; Retell generic MCP tool can delete things.
