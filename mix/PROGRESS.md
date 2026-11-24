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
      le conflit biais relatif FA (pins unsloth vs #32740 amont) est
      un arbitrage du propriétaire, jamais de l'agent. Puis banc de
      la couche 2 : couple témoin/candidat défini avec le propriétaire
      AVANT de lancer (base + pins sans RDNA3 ? mix complet vs couche 1 ?).
- [ ] Cycle amont : au prochain tag b#### utile — rejouer les couches
      sur la nouvelle base (mix/AGENTS.md § cycle), build gate, banc
      vs référence, tag mix-<b####>-rN. Base courante : b12670
      (mix-b12670-r1).

## Blocages / questions ouvertes
- (aucun)

## Dernière synchronisation
2026-11-24 — fichier déplacé de la racine vers mix/ (brief fondateur :
aucun fichier hors mix/ sauf amont). Banc Inkling-2 débloqué par le
cycle b12670 puis réceptionné côté notes (r1 indicatif ; r2 : arrêt
silencieux levé, artefact de harnais — mesures/2026.md). master
synchronisé sur upstream (4af4bb6d8) et protégé côté GitHub (force
push et suppression refusés, admins inclus).
