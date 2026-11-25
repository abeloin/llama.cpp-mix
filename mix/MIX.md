# MIX.md — journal du mix

## Base

- **Courante (branche mix/wip-b12723, en attente d'accord avant merge dans
  mix/gfx1101) : b12723 (abca69200)**, choisie par le propriétaire le
  2026-11-25, posée par merge --no-ff (a940ab925) — 53 commits amont depuis
  b12670, dont #32940 (clip Inkling 2 en l2_norm + scale), #32218 (DSpark),
  #32948 (graphe inkling2 allégé), #32500 (fusion RMS_NORM + MUL + MROPE),
  #31669 (GDN par rejeu). 7 conflits, résolutions au message du merge.
- Précédente : b12670 (239242322), posée par merge --no-ff (0e4789e95)
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
  le biais indexé côté CPU (ggml-cpu.cpp) — refus levé chez nous, voir le
  calque « code à nous » ci-dessous.
  RETRAIT : dès qu'une PR CUDA/HIP du biais indexé entre dans la base amont.
- pin: CUDA: pack several short rows per block in mul_mat_vec_f (unslothai/llama.cpp#400, commit a5667a9c0) — couche 2, liste A approuvée par mesure (banc r5 : Inkling tg +16,6 % pour l'ensemble des 7 pins).
- pin: CUDA: fuse the SiTU gate of the MoE experts (unslothai/llama.cpp#402, commit 0bfd204bd) — couche 2, liste A approuvée par mesure (banc r5 : Inkling tg +16,6 % pour l'ensemble des 7 pins).
- pin: CUDA: fuse the short convolution chain with its state store and residual (unslothai/llama.cpp#411, commit 7e633d60b) — couche 2, liste A approuvée par mesure (banc r5 : Inkling tg +16,6 % pour l'ensemble des 7 pins).
- pin: CUDA: fuse tanh-mul-add/sub, router weight normalization and add + rms_norm chains (unslothai/llama.cpp#412, commit 874c88cad) — couche 2, liste A approuvée par mesure (banc r5 : Inkling tg +16,6 % pour l'ensemble des 7 pins).
- pin: llama : one MoE cache entry per expert weight for looped layers (unslothai/llama.cpp#421, commit b336a7e65) — couche 2, liste A approuvée par mesure (banc r5 : Inkling tg +16,6 % pour l'ensemble des 7 pins).
- RETIRÉ (2026-11-25, base b12723, revert 80211b438 avec ses cas de test) —
  #32940 n'émet plus le motif à 8 ops ; décision du propriétaire du 2026-11-24.
  Ancienne ligne : pin: CUDA: fuse the row L2 clip chain (unslothai/llama.cpp#401, commit 6c0e84ddf) — couche 2, liste A approuvée par mesure (banc r5 : Inkling tg +16,6 % pour l'ensemble des 7 pins) ; cas de test du pin non repris (conflit tests seul).
- pin: CUDA: fuse the depth mix (stack concat, scores, softmax, mix) into one kernel (unslothai/llama.cpp#414, commit f766457c8) — couche 2, liste A approuvée par mesure (banc r5 : Inkling tg +16,6 % pour l'ensemble des 7 pins) ; cas de test du pin non repris (conflit tests seul).
- rdna3: sélection r3, 12 commits (rafasmour/llama.cpp-RDNA3, master, PR du fork
  #47 #61 #71 #73 #75 #77 #81 #85 #87 #94 #96 #98), cherry-pick -x, auteur
  conservé ; #43 (cefeb9b41) vide ici, son changement est déjà présent.
  #61 et #75 sont des ports de PR amont ouvertes (ggml-org #31746, #31761,
  ravel7524). #73 et #98 : cas de test non repris (conflit tests seul).
  #77 corrige l'issue amont #30175 (prefill FA non déterministe sur RDNA3).
  Mesures : banc r6 (Gemma4 tg +12,7 %, prefill +9,8 % ; Qwen3.6 +8,3 % ;
  Inkling +1,7 %) et r8 (ensemble).
- calque « code à nous » (b92222d86, merge 52b91bc1a, approuvé par le
  propriétaire le 2026-11-24) : ggml-cpu accepte FLASH_ATTN_EXT avec biais
  indexé (retrait d'une garde de supports_op ajoutée par
  ggml-org/llama.cpp#32740 ; le noyau CPU de référence est amont). Motif :
  les 24 cas de biais indexé de test-backend-ops étaient sautés faute de
  référence CPU. Banc r11 (Inkling UD-Q4_K_XL, experts en mémoire hôte) :
  aucun effet à l'exécution. RETRAIT : si l'amont lève le refus.
- tests: cas de test de #401, #414, rafasmour #73 et #98 réintégrés
  verbatim (même merge) — les mentions « cas de test non repris » ci-dessus
  sont closes.
- pin: CUDA: add IQ1_M to MMQ (unslothai/llama.cpp#304, commit e4e1eba07) — couche 2, set B : seul pin retenu (approuvé le 2026-11-24 ; #264 et #251 écartés, #405 DSpark mis de côté). Banc r12 : Qwen3.6-35B-A3B UD-IQ1_M prefill +41 % (1388 → 1954 tok/s), tg inchangé ; test-backend-ops 21766/21766 (merge 45d2f8a84).

## Conflits

Néant en cours.

- TRANCHÉ le 2026-11-25 par le propriétaire — fusion RMS_NORM + MUL + RoPE
  multi-sections : l'amont ggml-org#32500 (base b12723) l'emporte sur le
  commit rdna3 22275994b (rafasmour, même fusion pour MROPE/IMROPE). Au merge
  de b12723 : rope.cu et l'aiguillage de ggml-cuda.cu pris de l'amont, cas de
  test de 22275994b retirés. #32500 n'a pas été testée sous ROCm par son
  auteur : banc Qwen3.6 contre r4 au build gate.
- Compatibles, conservés tous deux (merge b12723) : rdna3 ee09feff8 (lecture
  en place des états récurrents) et ggml-org#31669 (rejeu GDN). Le rejeu
  suppose n_rs_seq > 0, cas où is_s_copy_identity() renvoie déjà faux.

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
