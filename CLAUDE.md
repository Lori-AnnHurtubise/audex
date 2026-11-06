# AUDEX Avocats — avoc.ca

Cabinet juridique à Québec et Portneuf. Site statique Astro 5 déployé sur Netlify.

## Stack technique

- **Framework** : Astro 5.1 (SSG — rendu statique, aucun serveur)
- **CSS** : Tailwind CSS + styles globaux dans `src/styles/global.css`
- **CMS** : Decap CMS (interface : avoc.ca/admin, auth via DecapBridge)
- **Déploiement** : Netlify — auto-deploy à chaque `git push` sur `main`
- **Dépôt** : GitHub → `Lori-AnnHurtubise/audex` (branche principale : `main`)
- **Langue** : Français québécois partout

## Commandes utiles

```bash
npm run dev      # serveur local http://localhost:4321
npm run build    # build de production dans dist/
npm run preview  # prévisualiser le build
```

Toujours faire un `npm run build` avant de committer pour vérifier qu'il n'y a pas d'erreurs.

## Façon de travailler

- **Un push sur `main` = mise en ligne sur avoc.ca en ~2 min.** Il n'y a pas d'environnement de préproduction.
- Le client modifie aussi le contenu via le CMS, ce qui crée des commits sur `main`. **Toujours faire `git pull` avant de commencer** une modification.
- Ajouter les fichiers un par un (`git add <fichier>`) plutôt que `git add -A`, pour ne pas committer de fichiers parasites.
- Pour annuler une mise en ligne : `git revert <commit>` puis `git push`.
- Les nouvelles demandes du client sont déposées dans le Google Drive partagé ADUX :
  `Disques partagés\ADUX - D4 - PRODUCTION INTERNE\voute_adux_strategies\Adux Stratégies\Audex Avocat\_Claude Code\nouvelles demandes\`

## Architecture des fichiers

```
src/
  content/          # Données gérées par le CMS
    blog/           # Articles (.md) — publishDate, tags, draft
    services/       # Pages de services (.md) — 5 domaines actifs
    team/           # Fiches membres (.json) — nom, rôle, bio, practiceAreas, photo
    pages/          # Contenu des pages statiques (.md)
    settings/       # analytics.json (GTM ID)
  pages/
    index.astro           # Accueil
    notre-equipe/
      index.astro         # Liste de l'équipe
      [slug].astro        # Fiche individuelle — rendu bio markdown via marked
    services/[slug].astro # Pages de pratique (générées depuis src/content/services/)
    blog/
      index.astro         # Liste des articles
      [slug].astro        # Article individuel
    nous-joindre.astro
    le-cabinet.astro
    politique-confidentialite.astro
    merci.astro           # Page de confirmation formulaire (exclue du sitemap)
  layouts/
    BaseLayout.astro      # Layout principal — canonical, GTM, header, footer
    ServiceLayout.astro   # Layout pages de services — inclut ServiceTeam
                          # Gabarit « deux publics » (employeurs/employés) activé si le .md
                          # définit `audiences` (ex. droit-emploi-travail) : cartes d'ancre,
                          # FAQ en 2 colonnes (faq[].group) et JSON-LD généré (page, service, FAQ)
                          # Forfaits à prix fixe activés si le .md définit `packageGroups`
                          # (ex. droit-affaires) : grille, étapes, encadré régions, outil
                          # « Trouvez le bon forfait » (sélection + forfait suggéré, aucun champ
                          # personnel ni envoi) et JSON-LD avec OfferCatalog
  components/
    cards/TeamCard.astro           # Carte membre (grille /notre-equipe/)
    sections/ServiceTeam.astro     # Avocats liés à un domaine (services + blog)
    sections/ArticleRelatedServices.astro
  data/
    company.ts            # Coordonnées, téléphone, réseaux sociaux
    services.ts           # Liste des services : menu déroulant + grille de l'accueil
    tag-to-service.ts     # Mapping tags blog → slugs de service (pour ServiceTeam)
  styles/global.css       # .services-grid = nb de colonnes de la grille de l'accueil
public/
  admin/config.yml  # Configuration Decap CMS — MODIFIER ICI pour champs CMS
  robots.txt        # User-agent *, Sitemap: https://avoc.ca/sitemap-index.xml
netlify.toml        # Redirections 301/302, headers Content-Type, build config
astro.config.mjs    # Config Astro — sitemap (filter + priorities), integrations
```

## Domaines de pratique (slugs)

5 domaines actifs, dans cet ordre partout (alphabétique, médiation à la fin).
Un service se retrouve à 4 endroits qu'il faut garder synchronisés :
`src/content/services/<slug>.md`, `src/data/services.ts`, `src/data/tag-to-service.ts`
et les options `practiceAreas` de `public/admin/config.yml`.

| Slug | Label affiché |
|------|---------------|
| `droit-criminel-penal` | Droit criminel et pénal |
| `droit-affaires` | Droit des affaires |
| `droit-emploi-travail` | Droit du travail |
| `droit-familial` | Droit familial |
| `mediation-familiale` | Médiation familiale |

Services archivés (fichiers supprimés, redirections 301 dans `netlify.toml`) :
`droit-commercial` → droit-affaires, `droit-jeunesse` → droit-familial,
`droit-civil-general`, `droit-administratif`, `droit-municipal` → accueil.

## CMS Decap — comment ça fonctionne

- URL : **avoc.ca/admin**
- Authentification : **DecapBridge** (decapbridge.com) — les utilisateurs du CMS sont invités
  depuis le tableau de bord DecapBridge, ils n'ont pas besoin de compte GitHub
- Chaque sauvegarde dans le CMS = un commit automatique sur la branche `main`
  (message « … via DecapBridge »)
- Le commit déclenche un déploiement automatique sur Netlify (~2 min)
- Configuration du CMS : `public/admin/config.yml`

## Déploiement Netlify

- Tout `git push` sur `main` déclenche un build automatique
- Build command : `npm run build`
- Publish directory : `dist`
- Node version : 20 (configuré dans `netlify.toml`)
- Site Netlify : `audex-site` (audex-site.netlify.app) — domaine avoc.ca, DNS chez Cloudflare
- Les fichiers statiques ont priorité sur les redirections `netlify.toml`

## Redirections (netlify.toml)

- 37 redirections 301 permanentes (anciennes URL WordPress → nouvelle structure)
- 1 redirection 301 pour `audrey-castonguay` (ex-membre sans fiche)
- `/sitemap.xml` → `/sitemap-index.xml` (301)
- Voir commentaires dans `netlify.toml` pour le détail par groupe

## Sitemap

Généré automatiquement par `@astrojs/sitemap` à chaque build.
- `https://avoc.ca/sitemap-index.xml` → index pointant vers `sitemap-0.xml`
- Pages exclues : `/merci/` et `/landing/`
- Soumis à Google Search Console : les deux URL ci-dessus

## Google Tag Manager

- GTM ID géré via `src/content/settings/analytics.json`
- Modifiable dans Decap CMS → Paramètres → Analytics
- Chargé dans `BaseLayout.astro` uniquement si `gtmId` est défini

## Phase 2 — à compléter

- [ ] Biographies des avocats à saisir dans Decap CMS (widget markdown WYSIWYG)
- [ ] Photos haute résolution manquantes pour certains membres
- [ ] Redirections 302 équipe → convertir en 301 quand les fiches sont complètes
  (pour l'instant toutes les fiches existent, les 302 ont été supprimées)
- [ ] Consentement cookies / Loi 25 (CMP à choisir : Axeptio ou CookieYes)
- [ ] Pages géolocalisées à enrichir (avocat-quebec-ville, portneuf, donnacona)
- [ ] Grille de services de l'accueil : `repeat(4, 1fr)` à 1024px alors qu'il y a 5 services
  (Médiation familiale tombe seule sur une 2e ligne) — à valider avec le client
- [x] Page droit des affaires : forfaits à prix fixe, « Comment ça fonctionne », outil de
  sélection du forfait (sans champs personnels, à la demande du client) et nouvelles FAQ

## Membres de l'équipe avec champs de pratique configurés

| Fichier JSON | Domaines |
|---|---|
| `kevin-plamondon.json` | Familial, Médiation, Criminel, Affaires, Travail |
| `yannick-drapeau.json` | Familial, Travail |
| `mireille-moreau.json` | Familial, Affaires |
| `audrey-auclair.json` | Familial |
| `audrey-michaud-roy.json` | Familial |
| `frederique-drolet.json` | Criminel |
| `jacob-lachance.json` | Criminel (stagiaire en droit) |

Les autres membres (dont Véronik Levesque, étudiants, personnel admin) ont `practiceAreas: []`.
