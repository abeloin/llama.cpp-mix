# PROGRESS — état courant du fork

Aiguillage, pas d'historique : le passé vit dans mix/MIX.md et git.
En cas de divergence, le dépôt gagne. Une entrée terminée se supprime.
Règles : mix/AGENTS.md fait foi pour toute action.

## En cours
- [ ] Banc Inkling-2 de la couche 1 (réplication du verdict rdna3,
      exécuté depuis llama-mix-notes ; résultat attendu dans mesures/)
      — BLOQUÉ au boot : arch `inkling2` absente du couple v1
      (af6b5243d vanilla comme 26c603c4b couche 1, et de mix/gfx1101 ;
      arrivée en amont après b12292, premiers tags porteurs ~b12641).
      Échec consigné côté notes (verdict-2026-11-23-inkling-echec-arch.md,
      d0a0466) ; redébloqué par le cycle amont ci-dessous, puis réception
      selon banc.md étapes 6-12

## À faire
- [ ] Banc couche 2 : mesurer les pins unsloth isolément — définir le
      couple témoin/candidat avec le propriétaire AVANT de lancer
      (candidat = base + pins sans RDNA3 ? mix complet vs couche 1 ?)
- [ ] Cycle amont : au prochain tag b#### utile — rejouer les couches
      sur la nouvelle base (mix/AGENTS.md § cycle), build gate, banc
      vs référence, tag mix-<b####>-rN (le banc Inkling-2 ci-dessus en
      dépend : base ≥ premier tag porteur de l'arch inkling2)

## Blocages / questions ouvertes
- Arch `inkling2` absente du couple v1 — blocage du banc Inkling-2
  (détails et preuves côté llama-mix-notes, d0a0466) ; se lève par le
  cycle amont ci-dessus

## Dernière synchronisation
2026-11-23, fin de soirée — màj post-échec banc inkling (blocage arch consigné)
