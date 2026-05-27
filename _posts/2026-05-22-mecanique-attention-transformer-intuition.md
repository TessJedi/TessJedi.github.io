---
title: "La mécanique de l’attention, sans les maths"
subtitle: "Comment un Transformer pondère son contexte"
date: 2026-05-22 09:00:00 +0200
lang: fr
categories: [ai]
tags: [transformer, attention, deep-learning, arxiv]
author: mt
reading_time: 7
cover: /assets/img/posts/attention-cover.jpg
description: "Lecture intuitive du mécanisme d’attention introduit dans Vaswani et al., Attention Is All You Need, arXiv:1706.03762 (2017)."
---

L’attention est une opération de pondération apprise. Pour chaque position d’un texte, le modèle calcule une distribution sur les autres positions, puis combine leurs représentations selon cette distribution. La formule de Vaswani et al. (2017) la condense :

$$ \text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^\top}{\sqrt{d_k}}\right) V $$

Trois matrices entrent en jeu, toutes issues du même input par trois projections linéaires apprises ($$W_Q, W_K, W_V$$). La requête $$Q$$ représente ce qu’on cherche. Les clés $$K$$ représentent ce qu’on indexe. Les valeurs $$V$$ représentent le contenu qu’on combine. Le produit $$QK^\top$$ mesure la similarité entre chaque requête et chaque clé, le softmax la normalise en distribution, le produit final avec $$V$$ produit la moyenne pondérée.

## L’image qui aide

Une bibliothécaire reçoit ta demande de lecture. Elle a un catalogue (les clés), des livres sur les étagères (les valeurs), une requête en tête. Elle compare ta requête à chaque fiche du catalogue. Elle pondère les livres par leur pertinence. Elle te donne un mélange des plus pertinents. L’attention fait exactement cela, à chaque position de chaque couche, des milliards de fois pendant l’entraînement.

Le facteur $$\sqrt{d_k}$$ ? Une normalisation d’échelle. Sans lui, les produits scalaires explosent quand la dimension grandit, le softmax sature, le gradient meurt en chemin.

## Pourquoi 2017 change la donne

Les LSTM et GRU traitaient une phrase de gauche à droite. Le début se diluait au fur et à mesure que le modèle avançait. L’attention permet à chaque position de regarder toutes les autres sans détour séquentiel. Le coût est en $$O(n^2)$$ au lieu de $$O(n)$$ en mémoire, mais le calcul devient parallélisable sur GPU. Sans cette parallélisation, GPT-2, BERT et leurs descendants n’auraient pas été économiquement viables au-delà de quelques millions de paramètres.

## Ce que la formule masque

Elle ne dit pas ce qu’il faut regarder. Les matrices $$W_Q, W_K, W_V$$ sont ajustées par descente de gradient sur des milliards de phrases. La fonction objectif fait émerger les structures pertinentes pour la tâche, sans supervision sur ce qui doit retenir l’attention.

Elle ne dit pas non plus pourquoi plusieurs têtes en parallèle marchent mieux qu’une seule. La lecture mécanistique (Elhage et al., 2021, transformer-circuits.pub) montre que les têtes se spécialisent : certaines suivent la position relative, d’autres la coréférence, d’autres des patterns syntaxiques. Aucune supervision n’a été donnée pour cette répartition, elle émerge.

## Pour aller plus loin

- Vaswani et al., *Attention Is All You Need*, [arXiv