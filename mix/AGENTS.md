# AGENTS.md (mix/) — Mix personnel llama.cpp pour cartes RDNA3

> Ce fichier vit à `mix/AGENTS.md` dans le fork, PAS à la racine : l'amont a
> son propre `AGENTS.md` racine qu'on ne touche jamais. Tout fichier de ce
> dépôt qui n'est pas de l'amont vit sous `mix/` ou dans un dépôt privé
> séparé. Les personnalisations machine (matériel exact, presets, chiffres)
> vivent dans un dépôt annexe privé, hors de ce dépôt.

## Objectif du dépôt

Fork de maintenance **personnel**, pas un projet à part entière. Il produit
un binaire llama-server (ROCm/HIP) optimisé pour les cartes AMD RDNA3,
usage agentique local, par assemblage de trois sources :

1. **Base amont** : un tag `b####` de ggml-org/llama.cpp (jamais master
   flottant) ;
2. **Pins unsloth** : PR de unslothai/llama.cpp sélectionnés ;
3. **Commits RDNA3** : commits de perf sélectionnés dans des fours tiers
   (kernel FA WMMA, MoE mat-vec, retunes RDNA3).

## Politique de branches — master reste un miroir propre

Trois remotes, aucune pollution :

```bash
git remote add upstream https://github.com/ggml-org/llama.cpp
git remote add unsloth   https://github.com/unslothai/llama.cpp
git remote add rdna3     https://github.com/rafasmour/llama.cpp-RDNA3
git remote set-url --push upstream DISABLE
git remote set-url --push unsloth   DISABLE
git remote set-url --push rdna3     DISABLE
```

Les remotes tiers sont **read-only par construction** (pushurl invalidé).
Seul `origin` est poussable, avec branch protection sur `master`.

**Les remotes sont la seule source d'amont** : pas de clones locaux
d'upstream/unsloth/rdna3 dans le groupe de travail. `git fetch --all` avant
toute lecture d'un fichier amont (`git show <remote>/<ref>:<fichier>`,
`git log <remote>/<ref>`), et les bancs passent par
`git worktree add --detach ../<nom> <tag>` — jamais par un clone séparé.
Un seul banc actif à la fois suffit (mesures séquentielles sur GPU unique) ;
l'état testé est identifié par un tag (amont, de gouvernance, ou tag jetable
`banc-<AAAAMMJJ>-<sujet>` pour un wip), le worktree est détruit après la
mesure — build/ frais à chaque banc, l'origine du test vivant dans le
journal.

- **`master`** : miroir exact de ggml-org. Fast-forwards depuis
  `upstream/master` uniquement. **Aucun commit local, jamais.**
- **`mix/gfx1101`** : LA branche de travail du mix. Part du tag `b####`,
  reçoit pins et commits RDNA3 par merges `--no-ff`. Seule branche jamais
  built.
- Branches éphémères pour un pin en conflit : `mix/wip-<sujet>`, mergées dans
  `mix/gfx1101` puis supprimées.
- Interdiction : merger du contenu d'`unsloth`/`rdna3` dans `master`.

## Règles absolues

- **Pas de rebase/recréation d'historique** : les couches s'ajoutent par
  merges `--no-ff` au-dessus du tag amont.
- **Chaque commit garde sa provenance** dans son message :
  `pin: <description> (unslothai/llama.cpp#NNN, commit SHORTHASH)` ou
  `rdna3: <description> (rafasmour/llama.cpp-RDNA3, commit SHORTHASH)`.
- **Aucun commit/push vers les dépôts amont** sans instruction explicite.
- **Pas de réécriture de code importé** : les pins entrent verbatim ; si un
  conflit oblige une adaptation, le diff reste minimal et signalé dans le
  message de merge.
- **Aucune communication externe autre que les opérations git vers
  `origin`** : pas d'issue, de PR, de commentaire ou de message vers un
  dépôt amont ou un tiers, par quelque canal que ce soit (CLI, web,
  API). Une question à un auteur amont se pose au propriétaire, jamais
  envoyée par l'agent.

## Conflits pin/upstream — LA règle de vigilance

Deux implémentations concurrentes de la même fonctionnalité existent (biais
relatif FA : pins #385/#386 unsloth **vs** #32740 amont). Règles :

1. Avant tout ajout de pin, **grepper la couverture complète** : « qui
   utilise chaque symbole » sur tout le fichier — pas seulement « que le
   code déplacé appelle ».
2. Conflit de fonctionnalité → **demander au propriétaire**, ne pas choisir
   seul. Documenter la collision dans `mix/MIX.md`.
3. Après réconciliation, vérifier que le modèle n'utilise **qu'une seule**
   des routes concurrentes, jamais deux.

## Fichiers d'État

- `mix/AGENTS.md` — ce document.
- `mix/MIX.md` — journal du mix : tag de base, liste ordonnée des
  pins/commits (numéro, SHA, description, source), conflits et résolutions.
  **Une ligne par couche, écrite au moment du merge.** Jamais de rédaction
  différée.

Les personnalisations machine (presets de boot, mesures, journal de banc)
vivent dans le dépôt annexe privé, hors de ce dépôt.

## Cycle de mise à jour du mix

1. Choisir le nouveau tag amont. Le noter dans MIX.md avant tout merge.
2. Merger `--no-ff` le tag dans `mix/gfx1101`.
3. Rejouer le pin-set unsloth (l'ordre MATÉRISE — liste dans MIX.md).
4. Rejouer la sélection RDNA3 (liste des SHA dans MIX.md).
5. **Build gate obligatoire** — le mix n'est bon qu'après build vert + smoke
   test serveur + mesure contre la référence précédente.
6. Tagger localement : `mix-<b####>-r<N>`.

## Build et validation — point de contact avec la réalité TÔT

**Un agent ne travaille jamais longtemps sans faire tourner le compilateur.**
Les annonces ne valent rien ; seul le build vert clôt une étape.

Référence (ROCm, RDNA3) :

```bash
cmake -B build -DGGML_HIP=ON -DAMDGPU_TARGETS=<gfx####> \
      -DLLAMA_SERVER=ON -DCMAKE_BUILD_TYPE=Release
cmake --build build --target llama-server -j
```

Validation, dans l'ordre (chaque étape doit passer avant la suivante) :

1. Build vert des cibles `ggml llama-server` ;
2. Boot serveur avec un `-fitt` généreux ; vérifier les lignes
   `buffer size`/`MiB free` du log ;
3. **Vrai prompt de production** (~10-15k tokens, pas un warmup vide) : le
   warmup ne déclenche pas les allocations workspace cuBLAS ;
4. Mesure (prefill moyen, tg sur ≥ 300 tokens, MTP acceptance) vs mix
   précédent. Régression > 5 % → investiguer (variance d'état connue :
   ±10 % sur cellule unique).
5. Aucune éviction VRAM inattendue au breakdown d'arrêt.

## Protocole d'incertitude

- Mesure unique ≠ mesure : tout verdict de perf s'appuie sur un A/B ou est
  noté « indicatif » dans MIX.md.
- En cas d'erreur du compilateur : corriger à partir du message exact ;
  interdit de re-vérifier un fait déjà tranché par une source plus
  autoritaire (compilateur, log serveur).

## Sauvegarde et récupération

1. **Push systématique** après chaque couche : un commit non poussé n'existe
   que sur une machine.
2. `git stash` (ou branche wip/) AVANT toute commande destructive ;
   `git status` avant tout `reset --hard`.
3. **Interdit** : `git clean -fdx` (détruit les fichiers ignorés),
   rebase/filter-branch sur `mix/gfx1101`.
4. Réflexe de récupération : `git reflog` → `git fsck --lost-found` →
   `git log --all -- <fichier>`. Ne jamais re-créer un fichier disparu à la
   main avant d'avoir épuisé ces commandes.
