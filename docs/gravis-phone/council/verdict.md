# Council verdict + extended-arena winner (2026-10-14)

## Council (5 advisors → 5 blind peer reviews → chairman)
All 5 reviewers independently: strongest = Executor (D); biggest blind spot = Expansionist (A).

### Agreement
- The reel's Bland route doesn't fit Hebrew (no Hebrew, no Claude).
- Vapi is the platform: inline assistant, official MCP with create_call, native Claude, Hebrew STT.
- Goal 1 (call Itai) and goal 2 (call third parties) are separate problems: a +1 number Itai saves as a contact suffices for goal 1; for goal 2 the critical unknown is the number (caller ID), not the platform.
- §69א: no third-party calls until an Israeli Telnyx number with Local Calling passes a caller-ID test.
- The skill is a contract: brief → Itai's "yes" → AI disclosure → no commitments/payments/IDs → exit line "אבדוק עם איתי ואחזור" → structured summary.
- Transcript = untrusted data (prompt injection vs Gmail/Drive/Make).
- claude.ai 240 s limit → create then poll; fully automatic "continue with the answer" only in Claude Code.

### Clashes (resolved by chairman)
- Scope: hold-the-line + warm transfer is low risk but costs per minute → later gate with a cap; negotiation contradicts the no-commitment rule.
- Voice for decisions: call, but with a closed question (yes/no or ≤3 options), read-back, and the same options in writing.
- Inbound: from day 1 at least an answer that says who this is, or forward to Itai.
- Friends: don't call friends with a bot; send a message.

### Blind spots caught in review
- The in-call speaker is a memory-less, tool-less model → the brief must hold all context.
- Spoken Hebrew: TTS without niqqud, gendered numbers, unknown gender of the other side → neutral phrasing, numbers/times in words, pronunciation test.
- "Delete after summary" doesn't delete vendor copies → recordingEnabled:false, check retention.
- Toll fraud → destinations restricted to Israel at provider level, no auto-reload, daily cap.
- Calling hours: no Shabbat/holidays/night; cloud sessions run in UTC.
- Itai doesn't answer → backup channel, safe default = do nothing.
- Me/Truecaller may tag a new number as spam → register a name.

### Recommendation
- Vapi + claude-haiku-5-5 (reasoningEffort none). STT A/B: Soniox stt-rt-v5 vs Deepgram nova-3 (he). TTS A/B: ElevenLabs eleven_v3 vs Azure he-IL-HilaNeural vs Cartesia sonic-3 (he).
- Numbers: Telnyx +1 imported to Vapi for calls to Itai; Telnyx IL with Local Calling for third parties after the caller-ID gate; if it fails → Israeli SIP trunk or drop goal 2.
- Claude Code: skill + script (inline assistant → poll in background → structuredOutputs → continue). Key: env var locally, network secret in cloud.
- claude.ai/mobile: saved "Gravis" assistant + variableValues via MCP (inline via MCP undocumented).
- Gates: 0 = 10 calls to Itai (voice/latency/numbers) → 1 = goal 1 live → 2 = caller-ID matrix across carriers + Me/Truecaller + inbound answer → 3 = staged rehearsal, then info-only calls → 4 = DTMF + hold-the-line with warm transfer.
- First thing: open Telnyx today — IL number with Local Calling (KYC) + a +1 number to import into Vapi.

## Extended arena (deep: 10 routes, 3 rounds)
Round-1 leaders: Ruthless cut 73, Edge-cases-first 71, Connector-first 70 … Self-built 50.
Final: **"Make-gated Vapi" 87** vs "Ruthless v3 (script-only guardrails)" 82.
Winner: the Vapi key lives only in Make; 4 on-demand Make scenarios exposed as tools through Itai's existing Make connector —
gravis_call_itai, gravis_call_contact (+972 only, hours, daily cap, fixed opening, max duration), gravis_call_result (≤90 s wait per call → stays under 240 s), gravis_call_close (delete after summary).
Server-side guardrails, works on every surface incl. mobile, no server to host. Runner-up = script-only in Claude Code (use if Make sync output/quota fails).
Note: the arena assumed Haiku 5.5 unsupported in Vapi — superseded: claude-haiku-5-5 is in Vapi's current API enum.

## Final synthesis (session lead)
Both paths share one brief format and one prompt template:
- Path A — Claude Code (works today): skills/gravis-phone/scripts/gravis_call.py with guardrails in code.
- Path B — every surface incl. mobile: Make gateway (spec in skills/gravis-phone/references/make-gateway.md), built only after Itai approves.
- Raw Vapi MCP on claude.ai: not recommended (any number, any config, safety only in prompt).
