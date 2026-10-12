# Assistant de tri d'e-mails avec un agent IA

Projet personnel : un petit agent qui lit des e-mails, les classe, les résume, propose l'action à faire et rédige un brouillon de réponse. **Il ne répond jamais seul** : je relis et je décide.

Réalisé avec Make, Google Sheets et Claude (Anthropic).

## Le problème

Quand on cherche une alternance, la boîte mail se remplit vite : réponses de recruteurs, messages de l'école, factures, newsletters, e-mails suspects. Un message urgent peut se perdre au milieu des autres. Je voulais un outil qui trie et qui prépare la réponse, sans prendre de décision à ma place.

## Comment ça marche

```
Google Sheets (e-mails)  →  Claude (analyse)  →  Lecture du JSON  →  Google Sheets (résultats)
```

1. **Google Sheets – Search Rows** : lit chaque e-mail de la feuille (expéditeur, objet, corps).
2. **Anthropic Claude – Simple Text Prompt** : envoie l'e-mail à Claude avec un prompt qui impose une réponse au format JSON.
3. **JSON – Parse JSON** : transforme la réponse en champs utilisables.
4. **Google Sheets – Update a Row** : écrit le résultat dans les colonnes de la même ligne.

### Ce que l'agent renvoie pour chaque e-mail

| Champ | Contenu |
|---|---|
| `categorie` | candidature, ecole, administratif, evenement, perso, promo ou spam |
| `priorite` | haute, moyenne ou basse |
| `resume` | une phrase de 20 mots maximum |
| `action` | l'action à faire, ou « Aucune » |
| `brouillon` | un brouillon de réponse poli, ou vide si aucune réponse n'est attendue |
| `doute` | `true` si l'agent hésite sur la catégorie ou la priorité |

Le prompt complet est dans [`prompt_agent_mail.txt`](prompt_agent_mail.txt).

## Choix de conception

- **Réponse en JSON uniquement**, avec des listes de catégories fermées : le résultat est toujours exploitable par la suite du scénario.
- **Un champ « doute »** : l'agent signale quand il n'est pas sûr, plutôt que de se tromper sans le dire.
- **Humain dans la boucle** : l'agent prépare des brouillons, il n'envoie rien. Une règle du prompt lui demande de ne pas s'engager sur une date ou un prix (elle n'a pas été toujours respectée, voir l'évaluation).
- **Règle anti-invention** : l'agent n'utilise que les informations présentes dans le message.

## Évaluation

J'ai écrit 10 e-mails de test fictifs, couvrant chaque catégorie, avec la catégorie et la priorité que j'attendais ([`emails_test.csv`](emails_test.csv)). J'ai ensuite comparé ces valeurs attendues à celles de l'agent (colonnes E/G pour la catégorie, F/H pour la priorité).

| Mesure | Version 1 du prompt |
|---|---|
| Catégorie correcte | **8 / 10** |
| Priorité correcte | **5 / 10** |
| Erreurs de catégorie signalées par le champ « doute » | **2 sur 2** |
| Erreurs de priorité signalées par le champ « doute » | 2 sur 5 |

### Ce que j'ai observé

- **Les deux erreurs de catégorie ont été signalées par l'agent** (`doute = true`) : l'offre d'alternance envoyée par une plateforme d'emploi classée « promo » au lieu de « candidature », et le message de suppression de compte classé « spam » au lieu de « administratif ». Les deux cas sont réellement ambigus.
- **La priorité est le point faible (5/10).** Pour 2 des 5 écarts (l'événement « demain à 19h », le compte supprimé « dans 3 jours »), l'agent applique ma règle « échéance proche = haute » : c'est mon étiquette attendue qui était discutable, pas seulement l'agent. Pour les autres (facture, e-mail frauduleux), l'agent est plus prudent que moi.
- **Les brouillons s'engagent trop.** Malgré la règle du prompt, trois brouillons confirment une disponibilité ou une venue à la place de la personne (« Je vous confirme que je suis disponible jeudi à 14h », « Je te confirme ma venue »). C'est le problème principal à corriger.

### Itération suivante

- Interdire explicitement toute confirmation dans le brouillon : il doit proposer une trame avec un passage à compléter, par exemple « [je confirme / je ne suis pas disponible] ».
- Préciser la différence entre « spam » (lien suspect ou demande d'informations) et « administratif » (message de service sans lien).
- Clarifier les règles de priorité, puis refaire le test sur le même jeu de 10 e-mails et comparer.

| Mesure | Version 1 | Version 2 |
|---|---|---|
| Catégorie correcte | 8 / 10 | *à compléter* |
| Priorité correcte | 5 / 10 | *à compléter* |
| Brouillons qui s'engagent à ma place | 3 | *à compléter* |

## Limites

- Test sur 10 e-mails fictifs seulement : ce n'est pas assez pour mesurer une vraie fiabilité, et améliorer le prompt sur ces mêmes 10 cas risque de le rendre trop adapté à ce jeu de test.
- Les étiquettes « attendues » sont mes choix : certaines sont discutables, et une partie des écarts de priorité vient de là.
- L'agent lit une feuille Google Sheets, pas directement ma boîte mail.
- Les brouillons restent à relire : ils ne sont jamais envoyés automatiquement.

## Pistes pour la suite

- Connecter directement Gmail à la place de la feuille.
- Tester sur un plus grand nombre d'e-mails réels (anonymisés).
- Ajouter une étape de validation avant tout envoi.

## Outils

Make · Google Sheets · Claude (Anthropic) · prompt engineering · JSON
