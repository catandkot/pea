<div align="center">

[한국어](README.md) · [简体中文](README_zh.md) · [繁體中文](README_zh-TW.md) · [日本語](README_ja.md) · [Türkçe](README_tr.md) · [Русский](README_ru.md) · [Tiếng Việt](README_vi.md) · [ไทย](README_th.md) · [Deutsch](README_de.md) · [Español](README_es.md) · **Français** · [Українська](README_uk.md) · [Polski](README_pl.md) · [Português (Brasil)](README_pt-BR.md) · [العربية](README_ar.md) · [فارسی](README_fa.md)

# 🫛 PEA

### L'agent de la ferme intelligente — il lit les données du terrain, il décide, il agit.

**Les capteurs parlent déjà. PEA leur donne une voix compréhensible.**

[![license](https://img.shields.io/badge/license-MIT-f4c430?style=flat-square)](LICENSE)
[![status](https://img.shields.io/badge/status-concept-5c4ee5?style=flat-square)](#état-du-projet)

[Démarrage rapide](#démarrage-rapide) · [Ce qu'il fait](#ce-quil-fait) · [Comment ça marche](#comment-ça-marche) · [Principes de sécurité](#principes-de-sécurité) · [Feuille de route](#feuille-de-route)

</div>

---

PEA (le pois) est un **agent de ferme intelligente** pensé pour les cultures sous serre et en plein champ. Il rassemble les valeurs des capteurs — humidité du sol, température et humidité de l'air, rayonnement solaire, CO2, EC/pH du drainage — pour expliquer *ce qui se passe en ce moment*, puis pilote les actionneurs (irrigation, ventilation, ombrage, éclairage d'appoint) **dans les limites de sécurité**.

Chaque décision s'accompagne de ses justifications. PEA distingue et rapporte séparément les **valeurs observées**, les **valeurs déduites** et les **valeurs encore inconnues**. « Le sol est sec » relève de l'historique des capteurs, « le risque de condensation est élevé cet après-midi » est une déduction, et « la vanne d'irrigation s'est-elle réellement ouverte ? » reste inconnu tant qu'un débitmètre ne l'a pas confirmé.

Le travail reste à la ferme. Les seules données qui sortent sont celles que l'utilisateur a explicitement choisies.

## Démarrage rapide

> **Les commandes et le nom du paquet ci-dessous ne sont pas encore définitifs.** Aucun paquet `pea` n'est actuellement publié sur npm. Ce document sera mis à jour avec les commandes réelles au moment de la publication.

```bash
# 1) install (package not published yet)
npx pea-agents setup

# 2) readiness check — gateway, sensors and actuators
pea doctor

# 3) ask about today's greenhouse
pea analyze ./farm.toml
```

Une fois connecté à l'agent, il suffit de lui demander :

```text
L'irrigation du matin dans la serre 3 a dépassé la cible de 12 %. Quelle en est la cause ?
Comment modifier le planning d'irrigation de cet après-midi pour ramener le taux de drainage à 25 % ?
Indique-moi la durée d'humectation foliaire de la nuit dernière, d'après la température minimale.
```

## Ce qu'il fait

| Domaine | Ce que fait PEA | Données utilisées |
| --- | --- | --- |
| Environnement de culture | Température et humidité de la serre, VPD, suivi du rayonnement cumulé et signalement des plages anormales | Capteurs de température et d'humidité, pyranomètre |
| Irrigation et solution nutritive | Calcul des volumes d'irrigation à partir du rayonnement cumulé, alerte en cas d'écart au taux de drainage cible | Sondes d'humidité et de poids du sol, EC/pH du drainage |
| Ventilation et chauffage/climatisation | Maintien de la consigne de VPD, propositions de paliers pour les ouvrants et les ventilateurs | Retour d'ouverture des ouvrants, météo extérieure |
| Alertes maladies et ravageurs | Calcul du risque de maladies fongiques à partir de la durée d'humectation foliaire et des combinaisons température/humidité | Historique température/humidité, capteur d'humectation foliaire |
| Journal de culture | Enregistrement automatique des irrigations, de l'environnement et des interventions, aide à la saisie des observations de croissance | Journaux d'activité, photos de terrain |
| Prévision de récolte | Estimation de la date de récolte à partir des degrés-jours cumulés, comparaison entre campagnes | Historique des températures, paramètres variétaux |

Les éléments et les seuils du tableau sont **des valeurs à recalibrer pour chaque culture, variété et région**. PEA ne fait que proposer des valeurs par défaut ; il ne les présente jamais comme les valeurs de référence de cette ferme.

## Comment ça marche

Quand l'agent interroge PEA via MCP, PEA rassemble les données de la passerelle et renvoie un résultat accompagné de ses justifications. L'agent s'en sert ensuite pour expliquer la situation ou proposer un plan de contrôle. Le même workflow est disponible à l'identique depuis le CLI du terminal.

```text
agent → MCP → PEA → passerelle → capteurs
                    ← observations + déductions + inconnues
```

- **Local d'abord** — la collecte et l'analyse sont traitées à la ferme.
- **Les preuves d'abord** — chaque conclusion est conservée avec les valeurs brutes, l'horodatage et la source.
- **L'inconnu reste inconnu** — les capteurs absents ou les interruptions de communication ne sont pas comblés par des estimations : ils sont signalés explicitement.
- **Résultats identiques** — le CLI et MCP partagent les mêmes workflows et le même contrat de résultats.

## Intégrations prises en charge

| Intégration | Méthode | État |
| --- | --- | --- |
| Passerelle MQTT | Collecte des topics de capteurs par abonnement au broker | Prévu |
| Modbus RTU / TCP | Interrogation des registres du contrôleur | Prévu |
| LoRaWAN | Capteurs de plein champ via la passerelle | Prévu |
| Données météo | API météo régionale, température, humidité et précipitations extérieures | Prévu |
| Systèmes de gestion de culture existants | Fusion de l'historique par import de fichiers (CSV/JSON) | Prévu |

## Principes de sécurité

Actionner un équipement est irréversible. C'est pourquoi PEA respecte les règles suivantes.

1. **Approbation humaine par défaut** — toute action réelle (irrigation, ventilation, éclairage d'appoint) fait d'abord l'objet d'un plan affiché et d'une approbation.
2. **Plafonds stricts** — les limites configurées pour la ferme (volume d'irrigation maximal par intervention, plage de température autorisée, etc.) ne peuvent pas être franchies par une décision de l'agent.
3. **Sécurité par défaut** — en cas de coupure de communication ou de panne de courant, le système bascule du côté sûr (arrêt de l'irrigation, ouverture de la ventilation, etc.).
4. **Aucune pulvérisation automatisée** — PEA n'applique ni pesticides ni engrais ; il ne propose que le moment et les justifications.
5. **Traçabilité complète** — chaque commande est consignée avec sa cible, son horodatage, ses justifications et son résultat d'exécution.

## Feuille de route

- **M0 — Concept et documentation** *(actuel)* : définition du produit, principes de sécurité, conception du contrat de résultats
- **M1 — Collecte** : intégration de la passerelle, stockage de l'historique des capteurs, consultation des tendances environnementales
- **M2 — Conseil** : assistants d'irrigation et de ventilation, risque de maladies et de ravageurs, automatisation du journal de culture
- **M3 — Contrôle avec approbation** : contrôle automatique sous approbation humaine et sécurité par défaut
- **M4 — Exploitation** : prévision de récolte, comparaison multi-fermes, rapports de campagne

## État du projet

Ce document décrit le **concept produit de PEA**. Le code présent dans le checkout actuel est un outil de rétro-ingénierie (`rea-agents`) et n'implémente pas encore les fonctions de collecte et de contrôle décrites ci-dessus. Les noms de commandes, la liste des intégrations et les seuils seront fixés au fil de l'implémentation. Les README traduits se fondent encore sur la documentation précédente et seront progressivement mis à jour pour refléter ce changement de marque.

## Contribution

Les issues et les pull requests sont les bienvenues. Les valeurs observées sur le terrain, les cas d'échec et les valeurs de calibration par culture sont particulièrement utiles.

## Licence

[MIT](LICENSE)
