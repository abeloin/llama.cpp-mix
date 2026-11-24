# MIX.md — journal du mix

## Base

- Courante : b12292 (af6b5243d), choix 2026-11-23.
- Cycle en cours : b12670 (239242322), choix 2026-11-23 au soir —
  dernier tag amont confirmé par ls-remote (ggml-org), porteuses
  vérifiées par ancêtre : arch inkling2 5c008e000, fix experts host
  5bacb7c, compact bias #32740 (= tête du tag). Merge --no-ff à
  poser ; couple banc : vanilla b12670 vs mix-b12670-rN, recette
  prod (FA_QUANTS=all, LTO, NATIVE, amdclang theRock).

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
