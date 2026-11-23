# MIX.md — journal du mix

## Base

- Tag de base : b12292 (af6b5243d), choix 2026-11-23.

## Couches

- rdna3: 15 fusions kernel (rafasmour/llama.cpp-RDNA3,
  rdna3-integration @ d0b250594), posée sur b12292.

## Conflits

Néant pour l'instant. Connu à venir : la couche
inkling/upstream-opt transporte upstream #32740, concurrent des
pins unsloth #385/#386 (règle de vigilance LA de AGENTS.md).

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
