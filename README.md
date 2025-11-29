# n8n-assistant
INSTRUCTIONS PROJET – VERSION MASTER Tu agis comme chef de projet IA et architecte système. Tu guides l’infra IA perso (Raspberry Pi, Docker, n8n, PostgreSQL, Cloudflare, Tailscale, Telegram). Tu donnes étapes, priorités, corrections, sécurité et mises à jour de docs, toujours clairement et sans détour.
1. SYNTHÈSE GÉNÉRALE DU PROJET

Ton assistant IA personnel fonctionne déjà et fait un aller-retour complet :

Entrée utilisateur → OpenAI → Post-processing → Mémoire IA (PostgreSQL) → Réponse.

La phase “Assistant IA v1” est quasi terminée.

La mémoire vectorielle, l’Ollama future, le dashboard token, la centralisation logs, et la refonte modulaire totale sont les prochaines étapes.

2. CE QUI EST OPÉRATIONNEL
✅ 2.1 Toute la chaîne de traitement du message utilisateur

Webhook → normalisation → appel OpenAI → extraction → enregistrement dans la mémoire → retour utilisateur

Voir WF-A, WF-C et WF-B :

WF-A-ASSISTANT-ENTRY (webhook d’entrée)


WF-C-ASSISTANT-IA (appel OpenAI + mémoire)


WF-B-MEMOIRE-IA (insert DB)


✅ 2.2 Connexion OpenAI correcte

Credential HTTP Request Credentials – Header Auth, clé via variable d’environnement (bonne pratique)

Header Content-Type: application/json présent

Structure JSON propre
→ OK.

✅ 2.3 Mémoire IA “froide” opérationnelle

Tables créées : ai_session, ai_memory.
Les deux sont insérées correctement, sans duplication de session (ON CONFLICT).

✅ 2.4 Logs d’erreurs PostgreSQL

WF-E-LOGS-ERROR :

→ Fonctionnel. Table ai_logs utilisée.

✅ 2.5 Post-processing de texte

WF-D-POSTPROCESSING : nettoyage du texte

→ Fonctionnel.

✅ 2.6 Architecture Docker / env

N8N_ALLOW_ENV_VARIABLES activé

OPENAI_API_KEY OK

PostgreSQL OK

Tailscale + Cloudflare Tunnel OK

⭐ En résumé :

L’assistant fonctionne, parle, comprend, et mémorise correctement.

3. CE QUI EST PARTIELLEMENT FAIT
◼ Mémoire vectorielle

Tu as préparé l’infra et la réflexion, mais :

PGVector pas encore installé

Pas de workflow de vectorisation

Pas de schéma pour embeddings

Pas de requête sémantique en entrée des réponses IA

→ Phase non commencée.

◼ Sécurisation avancée

Credential OK

Rotation automatisée non mise en place

Pas encore de politique multi-utilisateur n8n (mais prévue)

◼ Nettoyage de workflow

WF-C contient encore un JSON brut dans le body, pas encore refactorisé en “Add Fields Below”.
→ Fonctionnel mais pas optimal.

◼ Pas encore de gestion “Token cost”

Tu veux un dashboard + logs token : rien n’est encore implémenté.

4. CE QUI MANQUE
❌ 4.1 Tables manquantes pour la phase suivante

TABLE ai_embeddings
(mémoire vectorielle – future)

TABLE ai_conversations_summary
(résumés périodiques de session — conseillé)

TABLE ai_token_usage
(pour ton futur dashboard coût/token)

TABLE ai_photos / ai_files
(préparation future — classification photo/mail)

5. ÉTAT DES TABLES POSTGRESQL
✔️ Tables déjà créées (identifiées dans les workflows)

ai_session
Gérée par WF-B (PG-Ensure-Session)


ai_memory
Archivage du message utilisateur + réponse assistant


ai_logs
Utilisée par WF-E


❗ Tables non créées mais prévues dans l’architecture long terme

ai_embeddings (vectorisation, futur)

ai_conversations_summary (résumés)

ai_token_usage (consommation token)

ai_photos / ai_files (indexation multimédia)

ai_tasks (futur agent)

6. ARCHITECTURE DES WORKFLOWS ACTUELLE
6.1 Workflow d’entrée : WF-A

Reçoit : user_id, text, session_id, channel

Normalise et appelle WF-C


6.2 Workflow IA principal : WF-C

Construit le contexte

Appelle OpenAI

Extrait la réponse

Appelle la mémoire (WF-B)

Retourne : reply_text, session_id, user_id


6.3 Workflow mémoire : WF-B

Insère la session (si nouvelle)

Insère message utilisateur

Insère réponse assistant


6.4 Workflow post-processing : WF-D

Nettoie le texte


6.5 Workflow error logs : WF-E

Capture et stocke les erreurs


7. PROCHAINES ÉTAPES PRIORITAIRES (POUR CLAUDE)

Voici ce qu’il faut demander à Claude pour continuer exactement où nous en sommes :

PRIORITÉ 1 — Finaliser la mémoire vectorielle

Installer PGVector

Créer la table ai_embeddings

id

session_id

user_id

content

embedding (vector)

created_at

Workflow “Vector-Insert”

Workflow “Vector-Search”

Intégration dans WF-C avant appel OpenAI

PRIORITÉ 2 — Refactor du body HTTP OPENAI

Passer du JSON brut → “Add Fields Below”

Migrer vers /v1/responses (OpenAI recommandé 2025)

PRIORITÉ 3 — Dashboard Token

Créer table ai_token_usage

Intercepter headers OpenAI (x-request-cost, usage)

Workflow de log token

Dashboard auto-hébergé (HTML minimal ou Grafana)

PRIORITÉ 4 — Architecture scalable

Diviser WF-C en workflows atomiques :

Build-Context

Call-LLM

Post-processing

Memory-Write

Memory-Vector

Préparer l’arrivée d’Ollama

8. TEXTE À FOURNIR DIRECTEMENT À CLAUDE

Voici la version prête à copier-coller :

CONTEXTE À IMPORTER PAR CLAUDE

Je te transmets l’intégralité de l’état d’avancement de mon projet d’assistant IA personnel sur Raspberry Pi (Docker + n8n + PostgreSQL + Tailscale + Cloudflare).
L’assistant fonctionne déjà :

Réception d’un message via webhook

Appel OpenAI

Nettoyage du texte

Mémoire IA dans PostgreSQL

Logs d’erreurs
Les workflows en place sont WF-A, WF-C, WF-B, WF-D, WF-E.

Les tables existantes : ai_session, ai_memory, ai_logs.
Les tables manquantes : ai_embeddings, ai_token_usage, ai_conversations_summary.

Je veux que tu m’aides à terminer la mémoire vectorielle, refactorer proprement l’HTTP Request, mettre en place le dashboard token, et structurer l’assistant de façon modulaire et future-proof (intégration future d’Ollama).
