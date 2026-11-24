# MIX.md — journal du mix

## Base

- Courante : b12670 (239242322), posée par merge --no-ff (0e4789e95)
  le 2026-11-23 au soir — 378 commits amont, 3 conflits documentés au
  message du merge. Historique : b12292 (af6b5243d), choix 2026-11-23.
- Couple banc décidé par le propriétaire : vanilla b12670 vs
  mix-b12670-rN (tag après build gate), recette prod (FA_QUANTS=all,
  LTO, NATIVE, amdclang theRock).

## Couches

- rdna3: 15 fusions kernel (rafasmour/llama.cpp-RDNA3,
  rdna3-integration @ d0b250594), posée sur b12292.

## Conflits

Néant en cours.

- TRANCHÉ le 2026-11-24 par le propriétaire — biais relatif FA : la
  lignée native (upstream #32740, biais indexé sur FLASH_ATTN_EXT, déjà
  dans la base b12670) l'emporte ; les pins unsloth du biais relatif
  (#385 chunking, #386 FLASH_ATTN_EXT_BANDED, #413 lecture du biais
  dans le kernel vec) sont EXCLUS de la couche 2. Motifs : #386 étend
  l'op banded de l'Inkling 1 de juillet (ggml-org#25731, ouvert depuis
  le 2026-07-15, op absent de l'amont) — voie parallèle au design
  retenu par l'amont ; #385 redondant avec ggml-org#32883 (même idée,
  plus général) ; #413 inopérant en prod (K/V q5_0 ; utile seulement
  en f16, et sans objet une fois le biais indexé adopté). La couche 2
  peut prendre d'autres pins unsloth, hors biais relatif. Détails :
  llama-mix-notes, banc/notes/inkling-fast-validation-externe-2026-11-24.md.

## Défauts connus

- (aucun)
- LEVÉ le 2026-11-24 — mix-b12670-r1 (`e9207edb3`), « arrêt
  silencieux : breakdown jamais imprimé côté candidat » : artefact de
  harnais, pas un défaut du mix. run-banc.sh tuait le serveur sans
  l'attendre ; l'orphelin était coupé en plein teardown (log async).
  Avec le trap corrigé (wait + exit consigné), banc r2 du 2026-11-24 :
  exit=0 et breakdown imprimé des deux côtés. Côté notes :
  mesures/2026.md (ligne 2026-11-24).

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

## Dérogations (autorisées par le propriétaire)

- 2026-11-24 — réécriture d'historique de mix/gfx1101, malgré « pas de
  re-création d'historique » : courriel d'auteur erroné sur les 15
  commits du propriétaire (faute de user.email local). Auteur et
  committer seuls changés, arbres identiques octet pour octet ; les
  30 commits rafasmour et la base amont sont inchangés ; tag
  mix-b12670-r1 recréé sur le nouveau SHA (e9207edb3), push forcé avec
  bail. Table ancien → nouveau SHA : llama-mix-notes,
  banc/notes/remappage-sha-2026-11-24.md.
