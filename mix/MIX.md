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
- RETIRÉ (2026-11-24, intégration r3) — remplacé par le biais indexé GPU, mesuré
  meilleur (bancs r4, r7 : prefill Inkling +61 % vs #32883, compute 774 vs
  1051 MiB). Ancienne ligne :
  pr: chunking des requêtes de l'attention Inkling 2 (ggml-org/llama.cpp#32883,
  commit ebc0f114a, 3 commits de sanmai, importés verbatim), posé le
  2026-11-24 sur la base b12670. Premier calque de type « pr » : PR
  amont OUVERTE, non fusionnée, importée sur décision du propriétaire
  (2026-11-24) pour son effet mesuré — banc r3 [bon] : Inkling prod
  1/49 → 49/49 couches GPU, tg 5,10 → 21,86 tok/s, prefill +73 %
  (llama-mix-notes, mesures/2026.md). Build gate : le banc a tourné sur
  un arbre de code identique (461b341e0). RETRAIT : à rejouer au
  prochain cycle tant que la PR est ouverte ; à retirer dès qu'elle
  (ou un équivalent) entre dans la base amont ; à réviser si la PR
  évolue (prototype selon son auteur ; ubatch 1024/2048 suspects dans
  ses propres tests, notre profil prod est en -ub 256).
- calque « code adapté » : biais indexé CUDA/HIP de FLASH_ATTN_EXT et graphe
  Inkling 2 qui l'utilise (branche pr/cuda-indexed-bias : GPU retiré de
  ggml-org/llama.cpp#32740 avant merge — commit 9cd195304 inversé, auteur
  pwilkin — + ggml-org/llama.cpp#32899, garde NaN du MMA). Premier calque de
  ce type (accepté par le propriétaire le 2026-11-24). Conflit fattn-tile.cuh
  avec la couche rdna3 résolu à la main : type_K/type_V et
  use_rows/indexed_bias coexistent ; biais indexé et lignes K/V restent sur
  le chemin f16. test-backend-ops complet 21438/21438. NB : l'amont refuse
  le biais indexé côté CPU (ggml-cpu.cpp) — une couche d'attention laissée
  au CPU n'a donc pas de backend ; sans effet tant que tout tient au GPU.
  RETRAIT : dès qu'une PR CUDA/HIP du biais indexé entre dans la base amont.
- pin: CUDA: pack several short rows per block in mul_mat_vec_f (unslothai/llama.cpp#400, commit a5667a9c0) — couche 2, liste A approuvée par mesure (banc r5 : Inkling tg +16,6 % pour l'ensemble des 7 pins).
- pin: CUDA: fuse the SiTU gate of the MoE experts (unslothai/llama.cpp#402, commit 0bfd204bd) — couche 2, liste A approuvée par mesure (banc r5 : Inkling tg +16,6 % pour l'ensemble des 7 pins).
- pin: CUDA: fuse the short convolution chain with its state store and residual (unslothai/llama.cpp#411, commit 7e633d60b) — couche 2, liste A approuvée par mesure (banc r5 : Inkling tg +16,6 % pour l'ensemble des 7 pins).
- pin: CUDA: fuse tanh-mul-add/sub, router weight normalization and add + rms_norm chains (unslothai/llama.cpp#412, commit 874c88cad) — couche 2, liste A approuvée par mesure (banc r5 : Inkling tg +16,6 % pour l'ensemble des 7 pins).
- pin: llama : one MoE cache entry per expert weight for looped layers (unslothai/llama.cpp#421, commit b336a7e65) — couche 2, liste A approuvée par mesure (banc r5 : Inkling tg +16,6 % pour l'ensemble des 7 pins).
- pin: CUDA: fuse the row L2 clip chain (unslothai/llama.cpp#401, commit 6c0e84ddf) — couche 2, liste A approuvée par mesure (banc r5 : Inkling tg +16,6 % pour l'ensemble des 7 pins) ; cas de test du pin non repris (conflit tests seul).

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
