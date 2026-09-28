# AWN (عون) - The Digital First Responder — PRD

## Original problem statement
Arabic-language (RTL, dark-first) digital emergency platform:
1. Instant SOS & Smart Dispatch with priority categories and live radar/map tracking.
2. Public AED Locator map.
3. Interactive First-Aid Guide (12 acute emergencies, anti-myth warnings).
4. Digital Emergency Medical ID shared with responders.
5. Rapid Medical Store with simulated Apple Pay / Mada checkout and 15% VAT.
6. Gemini AI integration → **Smart First-Aid Assistant** (Arabic chat, step-by-step instructions + risk warnings), model **Gemini 3 Flash** via Emergent LLM key.

User communicates in English; app UI is strictly Arabic.

## Architecture
- Frontend: Expo SDK 57 / expo-router (tabs: الطوارئ, AED, المساعد, الدليل, المتجر), TanStack Query, react-native-keyboard-controller, react-native-maps lazily required (native only; web = radar fallback).
- Backend: FastAPI + Motor (MongoDB). Assistant uses `emergentintegrations` LlmChat → `gemini/gemini-3-flash-preview`, streamed via SSE.
- DB collections: guides, products, medical_profiles, emergencies, orders, assistant_messages.

## Key API endpoints
- GET /api/guides, GET /api/guides/{slug}
- GET /api/products, POST /api/checkout, GET /api/orders?device_id
- GET /api/aeds?lat&lng
- POST /api/emergencies, GET /api/emergencies/{id}, POST /api/emergencies/{id}/cancel
- GET/PUT /api/medical-id/{device_id}
- GET /api/assistant/history?device_id, DELETE /api/assistant/history?device_id
- POST /api/assistant/chat  (SSE: `data: {"delta"}` … `data: {"done", "id", "content"}` / `data: {"error"}`)

## What's done
- [x] All 5 core features scaffolded and seeded (previous session)
- [x] Gemini 3 Flash Smart First-Aid Assistant (backend SSE + chat tab + home shortcut) — 2026-06
- [x] Smart Case Triage: optional note in SOS sheet → `GET /api/emergencies/{id}/triage` (Gemini JSON, cached) → TriageCard on tracking screen
- [x] Voice input: mic button (expo-audio) → `POST /api/assistant/transcribe` (Whisper via Emergent key, ar) → auto-send
- [x] Guide handoff: keyword-matched `guide_slug/guide_title` on assistant replies + triage; "اقرأ الدليل الكامل" buttons
- [x] Share steps: native share / clipboard fallback on web + WhatsApp (wa.me) under assistant replies
- [x] Fixed runtime crashers: raw `ROW` string inside style arrays (emergency, checkout, guide, med-id), missing `fonts` import (store, checkout, med-id)

## Conventions / gotchas
- Never put a raw string (e.g. `ROW`) inside a style array — use `{ flexDirection: ROW }`.
- `react-native-maps` must stay lazily `require`d inside the native-only component.
- Streaming client uses `expo/fetch` (supports `response.body` streaming on native + web).
- `BottomTabBarHeightContext` imported from `expo-router/build/react-navigation/bottom-tabs` (no top-level @react-navigation/bottom-tabs).

## Backlog
- Real payment integration (currently simulated).
- Voice must be validated on a real device (Expo Go supports recording; web preview varies by browser).
