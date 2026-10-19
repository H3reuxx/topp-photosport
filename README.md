# Topp Photosport

Plateforme de photo sportive : les participants d'une course retrouvent leurs photos en tapant leur numéro de dossard, et peuvent les acheter en ligne.

Site : [topp-photosport.fr](https://topp-photosport.fr)

Le code de production est dans un dépôt privé (données des participants, logique métier, config). Ce dépôt sert à présenter le projet.

## Contexte

J'ai lancé Topp Photosport pour couvrir des courses à pied (route, trail, marathon). Sur un événement, on prend plusieurs milliers de photos, et chaque coureur veut retrouver les siennes rapidement. Trier à la main n'est pas envisageable, donc j'ai construit un système qui lit les dossards sur les photos et les range automatiquement par coureur.

Lire un numéro sur une photo, ça a l'air simple. En pratique, les dossards sont souvent cachés par un bras, pliés, coupés par le cadre, ou plusieurs coureurs se chevauchent. Et le pire cas, ce n'est pas une photo non trouvée : c'est une photo envoyée au mauvais coureur. Une bonne partie du travail a consisté à éviter ça.

## Fonctionnement

1. Les photos d'une course sont importées et stockées.
2. Un traitement sur GPU détecte les coureurs et lit les dossards.
3. Les résultats sont recoupés : un coureur apparaît sur plusieurs photos, donc si son dossard est lisible sur l'une, on peut le retrouver sur les autres.
4. Les cas douteux partent dans une interface où je les valide ou les corrige à la main.
5. Côté site, le participant cherche son dossard, voit ses photos, paie et les reçoit par email.

```
photos -> détection / lecture (GPU) -> recoupement -> base de données
                                                          |
                                             vérification manuelle
                                                          |
participant -> recherche dossard -> galerie -> paiement -> envoi par email
```

## Stack

Traitement des photos : Python, Modal (GPU à la demande), YOLOv8 pour la détection, Google Cloud Vision pour la lecture des numéros, InsightFace pour la réidentification.

Site et services : JavaScript sans framework, Tailwind CSS, Netlify (hébergement et fonctions serverless), Supabase (PostgreSQL), S3 pour le stockage, Stripe pour le paiement, Resend pour les emails.

J'ai choisi des GPU à la demande parce que l'activité fonctionne par pics : rien entre deux courses, puis beaucoup de photos d'un coup.

## Ce qui a été le plus difficile

Les coureurs proches les uns des autres. Les premières versions attribuaient parfois le dossard d'un coureur à son voisin. J'ai revu l'ordre des traitements et ajouté des règles pour qu'une déduction peu fiable ne puisse jamais remplacer un numéro réellement lu sur la photo.

Les lectures partielles. Il arrive que seul un morceau du numéro soit lu, et le résultat ressemble à un vrai numéro. En analysant les erreurs sur de vraies courses, j'ai repéré les lectures à risque, qui passent maintenant systématiquement en vérification manuelle.

Les rafales où le coureur est de dos. Sans visage, la réidentification devient compliquée. J'ai étudié plusieurs méthodes de suivi pour garder la trace d'un coureur d'une photo à l'autre.

De façon générale, chaque dossard attribué garde la trace de sa provenance, ce qui me permet de remonter à l'origine d'une erreur quand il y en a une.

## Le site

Frontend en JavaScript pur avec Tailwind, design volontairement brut (pas d'arrondis, typo en capitales, bordures marquées). Chaque course a ses propres pages et métadonnées pour le référencement, et un sitemap est généré automatiquement. Les paiements sont rattachés à la course concernée. Je collecte aussi quelques statistiques de fréquentation pour savoir ce qui est réellement utilisé.

## Méthode

Je mesure avant de modifier : les réglages du traitement sont choisis à partir des résultats sur des courses réelles. Le projet a une suite de tests que je fais passer avant chaque changement. Côté sécurité, j'ai fait des audits avec OWASP ZAP, la production et la préproduction sont séparées, et aucune clé n'est dans le code.

## Pour la suite

- mieux suivre les coureurs sur les longues rafales
- réidentifier sans dépendre du visage
- un tableau de bord pour les organisateurs
- tenir la charge sur des courses de plusieurs milliers de participants


Rozès Alban, étudiant en BUT Informatique à l'IUT de Rodez.
