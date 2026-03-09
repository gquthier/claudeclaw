---
name: generate-documents
description: Generate professional business documents (invoice, contract/proposal) as HTML files then convert to PDF for a client. Use when users ask to create an invoice, a contract, a proposal, a quote, a devis, a facture, a contrat, generate client documents, make a PDF invoice, create billing documents, write a service agreement. Trigger phrases include "create an invoice", "generate a contract", "make a facture", "create a contrat", "new invoice", "new contract", "generate documents for client", "créer une facture", "créer un contrat", "faire une facture", "faire un devis", "générer des documents", "documents pour client".
---

# Generate Documents

Generate a professional **invoice** (facture) and/or **contract/proposal** (contrat / proposition de services) as HTML files, then convert them to PDF using headless Chrome. Place output files on the user's Desktop or in the specified directory.

Use `$ARGUMENTS` to determine what the user wants (client name, service description, amount, etc.).

---

## Step 1 — Gather information

Ask the user (or extract from `$ARGUMENTS`) the following:

**Emitter (Prestataire) — prefill from context if known:**
- Company name
- Email
- Website
- Phone

**Client (Facturé à / LE CLIENT):**
- Company name
- Contact name & title
- Address (street, postal code, city)
- Email

**Mission details:**
- Short service description (one line, e.g. "Gestion et mise en place complète et optimisation des campagnes publicitaires Meta")
- Amount (TTC, e.g. 749,00 €)
- Invoice number (e.g. DG-2026-003)
- Date (default: today)
- Contract duration in days (default: 30)
- Stripe payment link (or other payment URL)

**Output:**
- Which documents to generate: invoice only, contract only, or both (default: both)
- Output directory (default: ~/Desktop)
- File base name (default: `<ClientName>-<ServiceShort>`, e.g. `BEC-Meta`)

---

## Step 2 — Generate the Invoice HTML

Create `<output_dir>/Facture-<basename>.html` using this exact template structure:

```html
<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>Facture - <EMITTER> / <CLIENT></title>
<style>
  @page { margin: 2cm; size: A4; @bottom-left { content: none; } @bottom-center { content: none; } @bottom-right { content: none; } }
  body { font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 13px; line-height: 1.5; color: #222; max-width: 750px; margin: 0 auto; padding: 40px; }
  .header { display: flex; justify-content: space-between; align-items: flex-start; margin-bottom: 40px; }
  .company-name { font-size: 24px; font-weight: bold; }
  .invoice-title { font-size: 28px; font-weight: bold; text-align: right; }
  .invoice-meta { text-align: right; color: #555; font-size: 13px; }
  .parties { display: flex; justify-content: space-between; margin: 30px 0 40px; }
  .party-box { width: 45%; }
  .party-box h4 { font-size: 11px; text-transform: uppercase; color: #888; letter-spacing: 1px; margin-bottom: 8px; }
  .party-box p { margin: 3px 0; }
  table { width: 100%; border-collapse: collapse; margin: 30px 0; }
  thead th { background: #222; color: #fff; padding: 10px 12px; text-align: left; font-size: 12px; text-transform: uppercase; letter-spacing: 0.5px; }
  tbody td { padding: 12px; border-bottom: 1px solid #eee; }
  .amount-col { text-align: right; }
  .total-section { margin-top: 20px; text-align: right; }
  .total-row { display: flex; justify-content: flex-end; margin: 5px 0; }
  .total-label { width: 200px; text-align: right; padding-right: 20px; color: #555; }
  .total-value { width: 120px; text-align: right; font-weight: bold; }
  .grand-total { font-size: 18px; border-top: 2px solid #222; padding-top: 10px; margin-top: 10px; }
  .payment-info { margin-top: 40px; padding: 20px; background: #f8f8f8; border-radius: 6px; }
  .payment-info h4 { margin: 0 0 10px; font-size: 14px; }
  .status { display: inline-block; padding: 4px 12px; border-radius: 20px; font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: 1px; }
  .status.pending { background: #fff3cd; color: #856404; }
</style>
</head>
<body>

<div class="header">
  <div>
    <div class="company-name"><!-- EMITTER_NAME --></div>
    <p style="color: #555; margin: 5px 0;"><!-- EMITTER_EMAIL --><br><!-- EMITTER_WEBSITE --><br><!-- EMITTER_PHONE --></p>
  </div>
  <div>
    <div class="invoice-title">FACTURE</div>
    <div class="invoice-meta">
      <p><strong>N° :</strong> <!-- INVOICE_NUMBER --></p>
      <p><strong>Date :</strong> <!-- DATE --></p>
      <p><strong>Échéance :</strong> À réception</p>
      <p><span class="status pending">En attente</span></p>
    </div>
  </div>
</div>

<div class="parties">
  <div class="party-box">
    <h4>Émetteur</h4>
    <p><strong><!-- EMITTER_NAME --></strong></p>
    <p><!-- EMITTER_EMAIL --></p>
    <p><!-- EMITTER_PHONE --></p>
  </div>
  <div class="party-box">
    <h4>Facturé à</h4>
    <p><strong><!-- CLIENT_COMPANY --></strong></p>
    <p><!-- CLIENT_CONTACT -->, <!-- CLIENT_TITLE --></p>
    <p><!-- CLIENT_ADDRESS --></p>
    <p><!-- CLIENT_CITY --></p>
    <p><!-- CLIENT_EMAIL --></p>
  </div>
</div>

<table>
  <thead>
    <tr>
      <th style="width: 50%;">Désignation</th>
      <th>Quantité</th>
      <th>Prix unitaire</th>
      <th class="amount-col">Montant</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong><!-- SERVICE_DESCRIPTION --></strong></td>
      <td>1</td>
      <td><!-- AMOUNT --></td>
      <td class="amount-col"><!-- AMOUNT --></td>
    </tr>
  </tbody>
</table>

<div class="total-section">
  <div class="total-row">
    <div class="total-label">Sous-total HT</div>
    <div class="total-value"><!-- AMOUNT --></div>
  </div>
  <div class="total-row">
    <div class="total-label">TVA (0%)</div>
    <div class="total-value">0,00 €</div>
  </div>
  <div class="total-row grand-total">
    <div class="total-label">TOTAL TTC</div>
    <div class="total-value"><!-- AMOUNT --></div>
  </div>
</div>

<p style="font-style: italic; font-size: 11px; color: #888; text-align: right; margin-top: 5px;">
  TVA non applicable — art. 293 B du CGI (le cas échéant)
</p>

<div class="payment-info">
  <h4>Paiement en ligne</h4>
  <p><a href="<!-- PAYMENT_LINK -->" style="color: #0066cc;"><!-- PAYMENT_LINK --></a></p>
</div>

</body>
</html>
```

Replace all `<!-- ... -->` placeholders with the actual values collected in Step 1.

---

## Step 3 — Generate the Contract HTML

Create `<output_dir>/Contrat-<basename>.html` using this exact template structure:

```html
<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>Proposition de Services - <EMITTER> / <CLIENT></title>
<style>
  @page { margin: 2.5cm; size: A4; @bottom-left { content: none; } @bottom-center { content: none; } @bottom-right { content: none; } }
  body { font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 13px; line-height: 1.6; color: #222; max-width: 750px; margin: 0 auto; padding: 40px; }
  h1 { font-size: 28px; text-align: center; margin-bottom: 5px; }
  .subtitle { text-align: center; font-size: 13px; color: #555; margin-bottom: 60px; }
  .parties { margin: 40px 0; }
  .parties h2 { font-style: italic; text-align: center; font-size: 20px; margin-bottom: 30px; }
  .party { margin: 20px 0; }
  .party.client { text-align: right; }
  .party .label { font-style: italic; color: #555; }
  h3 { font-style: italic; font-size: 18px; margin-top: 35px; margin-bottom: 15px; }
  ul { padding-left: 25px; }
  ul li { margin-bottom: 6px; }
  ul ul { list-style-type: circle; }
  .price { font-size: 16px; font-weight: bold; margin: 15px 0; }
  .tva-note { font-style: italic; font-size: 12px; color: #555; }
  .separator { border: none; border-top: 1px solid #ddd; margin: 30px 0; }
</style>
</head>
<body>

<h1>PROPOSITION DE SERVICES</h1>
<p class="subtitle">Date : <!-- DATE --> &nbsp;&nbsp; Validité de l'offre : 30 jours</p>

<div class="parties">
  <h2><em>ENTRE LES SOUSSIGNÉS :</em></h2>

  <div class="party">
    <p><strong>LE PRESTATAIRE</strong> <!-- EMITTER_NAME --><br>
    Email : <!-- EMITTER_EMAIL --><br>
    Site web : <!-- EMITTER_WEBSITE --><br>
    Téléphone : <!-- EMITTER_PHONE --><br>
    <span class="label">(Ci-après désigné "le Prestataire")</span></p>
  </div>

  <p style="text-align: center; margin: 30px 0; font-weight: bold;">ET</p>

  <div class="party client">
    <p><strong>LE CLIENT</strong> <!-- CLIENT_COMPANY --><br>
    Représentée par <!-- CLIENT_CONTACT -->, <!-- CLIENT_TITLE --><br>
    <!-- CLIENT_ADDRESS -->, <!-- CLIENT_CITY --><br>
    Email : <!-- CLIENT_EMAIL --><br>
    <span class="label">(Ci-après désigné "le Client")</span></p>
  </div>
</div>

<hr class="separator">

<h3>1. OBJET DE LA MISSION</h3>
<p>La présente lettre de mission a pour objet <!-- SERVICE_DESCRIPTION_LONG -->.</p>

<h3>2. PÉRIMÈTRE DE LA PRESTATION (INCLUS)</h3>

<p><strong>A. Cadrage & préparation</strong></p>
<ul>
  <li>Appel de lancement (brief) : activité, offre, client idéal, zone géographique, objectifs.</li>
  <li>Recommandations : positionnement et angle d'acquisition.</li>
  <li>Stratégie : Définition des messages, traitement des objections, éléments de réassurance.</li>
</ul>

<p><strong>B. Production des actifs</strong></p>
<ul>
  <li>Création de contenus publicitaires : formats adaptés aux plateformes (vidéo et/ou visuels, textes, variantes).</li>
  <li>Landing Page : Création ou adaptation d'une page de destination dédiée à la conversion.</li>
  <li>Branding : Mise en cohérence avec la marque du Client (sur la base des éléments fournis par le Client).</li>
</ul>

<p><strong>C. Mise en place technique & diffusion</strong></p>
<ul>
  <li>Setup technique : Création et/ou configuration des éléments techniques publicitaires (compte publicitaire, pixels/évènements, paramètres de conversion selon accès disponible).</li>
  <li>Lancement : Mise en ligne des campagnes sur les audiences pertinentes.</li>
  <li>Routage : Paramétrage du routage des leads (réception par email et/ou intégration CRM).</li>
</ul>

<p><strong>D. Pilotage & optimisation</strong></p>
<ul>
  <li>Suivi de performance : optimisations régulières (ciblage, messages, créas, page, budget, etc.).</li>
  <li>Points d'avancement : fréquence adaptée à la phase de lancement (asynchrone et/ou court point visio si nécessaire).</li>
</ul>

<h3>3. LIVRABLES</h3>
<p>Le Client recevra notamment :</p>
<ul>
  <li>Les actifs publicitaires produits (créations + textes).</li>
  <li>Une page de destination opérationnelle.</li>
  <li>Des campagnes configurées et actives.</li>
  <li>Un suivi synthétique (statuts, actions menées, axes d'amélioration).</li>
</ul>

<h3>4. DÉLAIS & DÉMARRAGE</h3>
<ul>
  <li>Démarrage opérationnel : Sous réserve de réception du paiement, des accès techniques et des éléments de marque.</li>
  <li>Objectif : Mise en route rapide et lancement dès validation des prérequis. Premières campagnes live sous 7 jours.</li>
</ul>

<h3>5. CONDITIONS FINANCIÈRES</h3>
<p>En contrepartie de la réalisation de la mission, le Client s'engage à verser au Prestataire une rémunération forfaitaire globale.</p>

<p class="price">Prix de la prestation : <!-- AMOUNT --> TTC (Mois 1 — période de test)</p>
<p class="tva-note">(TVA non applicable, art. 293 B du CGI.)</p>

<p>Conditions de règlement :</p>
<ul>
  <li>Facturation : Payable à réception de facture.<br>
      <strong>Lien de paiement :</strong> <a href="<!-- PAYMENT_LINK -->"><!-- PAYMENT_LINK --></a></li>
  <li>Budget Média (Non inclus) : Les dépenses publicitaires sont facturées directement par les plateformes et restent à la charge exclusive du Client. Budget recommandé : 500–600 €/mois (~20 €/jour).</li>
  <li>Frais annexes (Non inclus) : Coûts éventuels d'outils tiers (CRM, tracking avancé, hébergement, nom de domaine, etc.).</li>
</ul>

<h3>6. RÔLES ET RESPONSABILITÉS DU CLIENT</h3>
<p>Le Client s'engage à :</p>
<ul>
  <li>Fournir les accès nécessaires (Meta/Business Manager, site, DNS/hébergement, analytics).</li>
  <li>Pourvoir les éléments de marque (logo, charte, visuels, textes, offres, témoignages).</li>
  <li>Répondre rapidement aux demandes de validation (idéalement sous 24–72h).</li>
  <li>Traiter les leads reçus dans des délais courts.</li>
  <li>Garantir la conformité légale de son offre (mentions légales, CGV, politique de confidentialité, etc.).</li>
</ul>

<h3>7. CONFIDENTIALITÉ</h3>
<p>Le Prestataire s'engage à conserver confidentielles toutes informations, documents, données et stratégies communiqués par le Client, sauf obligation légale contraire.</p>

<h3>8. PERFORMANCE</h3>
<p>Le Client reconnaît que la performance dépend de facteurs externes (marché, concurrence, budget média, offre, saisonnalité, délais de réponse, etc.). En conséquence, aucun volume de leads, coût par lead, chiffre d'affaires ou résultat commercial n'est garanti.</p>
<ul>
  <li>Estimation indicative : À titre de repère opérationnel (non contractuel) : environ 20 leads qualifiés pour un budget média investi de 500 €.</li>
  <li>Clause de réactivité : En cas de performance significativement inférieure aux prévisions indicatives après 10 jours de campagne, le Prestataire s'engage à organiser un rendez-vous visio d'urgence sous 72h, analyser les causes, ajuster immédiatement et mettre en place un plan de redressement.</li>
</ul>

<h3>9. PROPRIÉTÉ INTELLECTUELLE & RÉVERSIBILITÉ</h3>
<ul>
  <li>Les livrables spécifiques (créations, landing page) sont cédés au Client après paiement intégral.</li>
  <li>Le Prestataire conserve ses méthodes, frameworks et savoir-faire générique.</li>
  <li>À la fin de la mission, le Prestataire facilite la réversibilité selon les limites techniques des plateformes.</li>
</ul>

<h3>10. DURÉE ET SUITE DE LA MISSION</h3>
<p>La présente mission est conclue pour une durée ferme de <!-- DURATION_DAYS --> jours à compter de la date de signature. Elle ne fait pas l'objet d'une tacite reconduction.</p>
<p>Ce tarif étant appliqué dans le cadre d'une période de test, une nouvelle proposition commerciale sera soumise au Client à l'issue de cette période.</p>

<h3>11. RESPONSABILITÉ</h3>
<p>La responsabilité du Prestataire ne saurait être engagée au titre des refus de plateformes, des indisponibilités techniques tierces, des conséquences d'un mauvais traitement commercial des leads, ou de toute communication portée par le Client.</p>

<h3>12. DONNÉES PERSONNELLES (RGPD)</h3>
<p>Chaque Partie s'engage à respecter la réglementation applicable. Le Client est responsable du traitement des données des prospects. Le Prestataire agit comme Sous-traitant pour les opérations nécessaires à la mission.</p>

<div style="page-break-before: always; padding-top: 40px;">
  <h3 style="font-style: italic; font-weight: bold; font-size: 22px;"><em>BON POUR ACCORD</em></h3>
  <p>Fait à Paris, le <!-- DATE_SHORT --></p>
  <p>En deux exemplaires originaux.</p>

  <div style="margin-top: 40px;">
    <div style="margin-bottom: 50px;">
      <p><strong>Pour le Client (<!-- CLIENT_COMPANY -->)</strong></p>
      <p style="margin-top: 5px; color: #777;">Nom et prénom : ___________________________________________</p>
      <p style="margin-top: 15px; color: #777;">Fonction : ___________________________________________</p>
      <p style="margin-top: 15px; color: #777;">Date : ___________________________________________</p>
      <p style="margin-top: 15px; color: #777;">Mention "Lu et approuvé" :</p>
      <div style="border: 1px dashed #ccc; border-radius: 8px; height: 120px; margin-top: 10px; display: flex; align-items: center; justify-content: center;">
        <span style="color: #bbb; font-size: 14px;">Signature du Client</span>
      </div>
    </div>

    <hr style="border: none; border-top: 1px solid #ddd; margin: 40px 0;">

    <div>
      <p><strong>Pour le Prestataire (<!-- EMITTER_NAME -->)</strong></p>
      <p style="margin-top: 5px; color: #777;">Nom et prénom : <!-- EMITTER_SIGNATORY_NAME --></p>
      <p style="margin-top: 15px; color: #777;">Fonction : <!-- EMITTER_SIGNATORY_TITLE --></p>
      <p style="margin-top: 15px; color: #777;">Date : <!-- DATE_SHORT --></p>
      <div style="border: 1px dashed #ccc; border-radius: 8px; height: 120px; margin-top: 10px; display: flex; align-items: center; justify-content: center;">
        <span style="color: #bbb; font-size: 14px;">Signature du Prestataire</span>
      </div>
    </div>
  </div>
</div>

</body>
</html>
```

Replace all `<!-- ... -->` placeholders with the actual values.

---

## Step 4 — Convert HTML to PDF

For each generated HTML file, convert to PDF using headless Chrome:

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless \
  --disable-gpu \
  --no-sandbox \
  --print-to-pdf="<output_dir>/<filename>.pdf" \
  --print-to-pdf-no-header \
  --no-pdf-header-footer \
  "file://<output_dir>/<filename>.html"
```

Run both conversions in parallel (background `&` then `wait`).

---

## Step 5 — Confirm

Tell the user:
- Which files were created (HTML + PDF for each document)
- Their full paths
- Any missing information that was left as placeholder
