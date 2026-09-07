# Changelog

Tutti gli aggiornamenti sulle funzionalità sono registrati in questo file.

## [1.0.3] - 2026-09-07

### Added

- Gestione options HA: HA espone le options dell'addon, Supervisor le salva in /data/options.json, lo script run.sh dovrà occuparsi di leggerle usando jq e mapparle in opportune variabili d'ambiente leggibili da Spring Boot.
- Aggiunta JAVA_OPTS da linea di comandi esposte tramite option HA.
- ApplicationConfigInfo - Visualizzazione opzioni (readonly) anche direttamente dalla GUI - Solo ROLE_ADMIN.

### Changed

- Dockerfile: supporto jq per lettura delle opzioni HA e impostazione JAVA_OPTS.
- run.sh: supporto jq e JAVA_OPTS

## [1.0.2] - 2026-09-04

### Added

- logo.png aggiunto alle risorse statiche

### Changed

- workflow CI tra i repository unicoanello e unicoanello-ha-addons: il workflow su unicoanello si attiva solo quando viene effettuato un tag del repository e si occupa di effettuare il build del progetto e dell'immagine docker; a questo punto effettua il trigger del workflow sul repository pubblico (usando il PAT HA_ADDONS_TOKEN) passando il numero di versione e nome del tag. Il workflow su unicoanello-ha-addons effettua il checkout di questo tag (usando il token UNICOANELLO_READ_TOKEN), aggiorna concordemente config.yaml, changelog e readme ed effettua commit e tag.
- Ottimizzazione Dockerfile.
- Preparazione per migrazione a Spring Boot 4.

## [1.0.1] - 2026-09-03

### Added

- Workflow Action CI/CD per pubblicazione artefatti per generare immagine docker per addon HA tramite webhook.
- .dockerignore per escludere i file IDE dal build context.

### Changed

- Variabili d'ambiente per proprietà applicative su application.yml.
- Dockerfile più specifico.

### Removed

- App Repository e Addon Home Assistant, spostato su progetto separato unicoanello-ha-addons.

## [1.0.0] - 2026-09-02

### Added

- Schema SQL con dati iniziali.
- KB Completa dal manuale base.
- UI per Edit Compagnia.
- UI per schede personaggio degli eroi.
- Onboarding Utenti.