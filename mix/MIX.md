# MIX.md — journal du mix

## Base

- Courante : b12670 (239242322), posée par merge --no-ff (ead929993)
  le 2026-11-23 au soir — 378 commits amont, 3 conflits documentés au
  message du merge. Historique : b12292 (af6b5243d), choix 2026-11-23.
- Couple banc décidé par le propriétaire : vanilla b12670 vs
  mix-b12670-rN (tag après build gate), recette prod (FA_QUANTS=all,
  LTO, NATIVE, amdclang theRock).

## Couches

- rdna3: 15 fusions kernel (rafasmour/llama.cpp-RDNA3,
  rdna3-integration @ d0b250594), posée sur b12292.

## Conflits

Néant en cours. Reporté (arbitrage propriétaire au moment voulu,
jamais seul — AGENTS § Conflits) : la base b12670 transporte
upstream #32740 (compact bias FLASH_ATTN_EXT) en natif, concurrent
des pins unsloth #385/#386 (biais relatif FA), ABSENTS du mix ; une
future couche pins unsloth devra arbitrer les deux routes — elles
ne coexisteront jamais dans un même binaire.

## Défauts connus

- mix-b12670-r1 (`5db315d62`, consigné le 2026-11-23 au soir) :
  « arrêt silencieux : breakdown jamais imprimé côté candidat
  (×2 + idle), VRAM libérée, pas de fuite, pas de crash noyau, voie
  byte-identique → comportemental, origine inconnue : merge ou
  couche (question posée à rafasmour) ». Dossier du côté notes :
  verdict r1 § Défaillance + PROGRESS (méthodo de reprise a-b-c).
  Le témoin b12670 imprime systématiquement le breakdown (0,3 s
  après « cleaning up before exit… »).

## Limitations connues

- UI web désactivée au build (`-DLLAMA_USE_PREBUILT_UI=OFF`) :
  le resolver de version a saisi le numéro b12325, venu du fetch du
  fork rdna3 (tag inexistant sur ggml-org) — le dist HF « apparié »
  à ce numéro ne peut pas exister, et le fallback `latest` apporte
  un layout SvelteKit incompatible avec la validation de b12292
  (vérifié : b12292 lui-même n'est plus servi non plus, 404).
  Recovery manuel si réactivé : `rm -rf build/tools/ui`, puis
  reconfigure avec le drapeau.
- Serveur headless attendu : avertissement CMake « no assets
  available » normal, pas une erreur.
