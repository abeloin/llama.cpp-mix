# PROGRESS — état courant du fork

Aiguillage, pas d'historique : le passé vit dans mix/MIX.md et git.
En cas de divergence, le dépôt gagne. Une entrée terminée se supprime.
Règles : mix/AGENTS.md fait foi pour toute action.

## En cours
- (aucun)

## À faire
- [ ] Couche 2 unsloth — jamais posée (étape 4 du brief fondateur) :
      lister les PR unsloth utiles (numéro, SHA, description, retenu
      ou écarté) et la soumettre au propriétaire AVANT tout merge ;
      pins unsloth du biais relatif FA (#385/#386/#413) EXCLUS par le
      propriétaire (2026-11-24, MIX.md § Conflits). Puis banc de
      la couche 2 : couple témoin/candidat défini avec le propriétaire
      AVANT de lancer (base + pins sans RDNA3 ? mix complet vs couche 1 ?).
- [ ] Cycle amont : au prochain tag b#### utile — rejouer les couches
      sur la nouvelle base (mix/AGENTS.md § cycle), build gate, banc
      vs référence, tag mix-<b####>-rN. Base courante : b12670
      (mix-b12670-r1).

## Blocages / questions ouvertes
- Prochaine décision du propriétaire : banc qwen r1 recette-prod
  (prêt, côté llama-mix-notes) OU couche 2 unsloth (commence par la
  liste de PR à approuver, ci-dessus).

## Dernière synchronisation
2026-11-24, ~4 h 40 — calque pr ggml-org#32883 posé (chunking de
l'attention Inkling 2) après banc r3 [bon] : Inkling prod 49/49 couches
GPU, tg ×4,3. Tag de gouvernance mix-b12670-r2. Pins unsloth du biais
relatif exclus de la couche 2 (MIX.md § Conflits).
