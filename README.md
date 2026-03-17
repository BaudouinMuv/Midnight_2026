# Midnight_2026
Construction d'une DAO pour les organisations locales en RDC
---
title: Construire une DAO pour les organisations locales en RD Congo
description: Apprenez à créer une Organisation Autonome Décentralisée (DAO) préservant la confidentialité pour les coopératives et associations locales en République Démocratique du Congo en utilisant Midnight.
sidebar_label: DAO pour les organisations en RDC
---

# Construire une DAO pour les organisations locales en RD Congo

Ce tutoriel vous guide dans la création d'une **Organisation Autonome Décentralisée (DAO)** conçue pour répondre aux besoins des organisations locales (coopératives agricoles, associations communautaires, micro-crédits) en République Démocratique du Congo (RDC). 

En utilisant **Midnight**, vous pouvez construire une gouvernance numérique qui protège l'anonymat des membres et la confidentialité des votes, tout en garantissant une transparence totale sur les résultats, ce qui est crucial pour renforcer la confiance dans les contextes locaux.

## Pourquoi une DAO sur Midnight pour la RDC ?

Dans de nombreuses régions de la RDC, les organisations locales font face à des défis de transparence et de sécurité. Une DAO sur Midnight offre :
- **Confidentialité du vote** : Les membres peuvent voter sans crainte de pressions extérieures.
- **Transparence financière** : Gestion partagée des fonds de l'organisation.
- **Accessibilité** : Une gouvernance numérique qui réduit les besoins de déplacements physiques coûteux.

## Objectifs du tutoriel

À la fin de ce tutoriel, vous saurez :
- Définir une structure de membre confidentielle en **Compact**.
- Implémenter un système de proposition et de vote anonyme.
- Gérer un trésor communautaire de manière décentralisée.

## Prérequis

- Avoir installé la [chaîne d'outils Midnight](https://docs.midnight.network/getting-started/installation ).
- Node.js v22+.
- Compréhension de base du langage **Compact**.

---

## 1. Définir le contrat de la DAO

Créez un fichier nommé `dao_rdc.compact` dans votre dossier `contracts`.

```compact
pragma language_version 0.21;

import "@midnight-ntwrk/compact-stdlib";

// Structure pour une proposition
export struct Proposal {
    title: Opaque<"string">,
    description: Opaque<"string">,
    amount_requested: Uint64,
    votes_for: Uint64,
    votes_against: Uint64,
    is_active: Boolean
}

// État public de la DAO
export ledger proposals: Map<Uint64, Proposal>;
export ledger total_proposals: Uint64;

// État privé (pour prouver l'appartenance sans révéler l'identité)
witness is_member(): Boolean;

/**
 * Créer une nouvelle proposition pour un projet local
 */
export circuit create_proposal(
    title: Opaque<"string">,
    description: Opaque<"string">,
    amount: Uint64
): Uint64 {
    assert is_member() "Seuls les membres peuvent créer des propositions";
    
    let proposal_id = total_proposals;
    proposals[proposal_id] = Proposal {
        title: title,
        description: description,
        amount_requested: amount,
        votes_for: 0,
        votes_against: 0,
        is_active: true
    };
    
    total_proposals = total_proposals + 1;
    return proposal_id;
}

/**
 * Voter de manière confidentielle
 */
export circuit vote(proposal_id: Uint64, support: Boolean): [] {
    assert is_member() "Seuls les membres peuvent voter";
    let proposal = proposals[proposal_id];
    assert proposal.is_active "La proposition n'est plus active";

    if (support) {
        proposal.votes_for = proposal.votes_for + 1;
    } else {
        proposal.votes_against = proposal.votes_against + 1;
    }
    
    proposals[proposal_id] = proposal;
}
