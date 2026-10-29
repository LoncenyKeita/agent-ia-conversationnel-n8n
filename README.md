# Agent IA de réservation d'activités à Bali (n8n)

Agent conversationnel construit avec n8n. Il répond aux questions des clients
sur des activités à Bali en s'appuyant sur une base de connaissance (RAG), 
enregistre les prospects dans Google Sheets, crée les réservations dans Google 
Calendar et prévient l'équipe sur Slack.

# Statut du projet : 
fonctionne avec le chat intégré de n8n. La connexion WhatsApp n'est pas finalisée (voir Pistes d'évolution).

# Sommaire

1- Fonctionnalités

2- Architecture

3- Prérequis

4- Contenu du dépôt

5- Installation

6- Configuration des services

7- Base de connaissance

8- Prompt de l'agent

9- Tester le projet

10- Limites connues

11- Dépannage

12- Pistes d'évolution

13- Sécurité
14- 

# Fonctionnalités

- Présenter les activités, prix, durées, réductions et conditions à partir d'un fichier de connaissance.
- Collecter les informations du prospect : nom, email, téléphone, activité souhaitée, date.
- Enregistrer le prospect dans un Google Sheet.
- Créer un événement dans un Google Agenda dédié.
- Notifier l'équipe dans un canal Slack.
- Garder la mémoire de la conversation (Simple Memory de n8n).

# Architecture

Le workflow contient deux branches indépendantes.

Branche 1 : indexation de la base de connaissance (à lancer une fois)

Google Drive (fichier .txt)
        │
        ▼

Download file (Google Drive)
        │
        ▼
        
Simple Vector Store  ── mode "Insert Documents"
   ├── Default Data Loader (+ Recursive Character Text Splitter)
   └── Embeddings OpenAI

Branche 2 : conversation avec le client

Chat Trigger ──► AI Agent
                   ├── Chat Model : OpenAI Chat Model
                   ├── Memory : Simple Memory
                   └── Tools :
                        ├── Simple Vector Store (mode "Retrieve Documents (As Tool for AI Agent)")
                        │      └── Embeddings OpenAI (même modèle qu'à l'insertion)
                        ├── Google Sheets : Append row in sheet
                        ├── Google Calendar : Create an event
                        ├── Slack : Send a message
                        └── Gmail : Send a message   (présent dans le workflow exporté)
                    
Un nœud SerpApi (recherche Google) figurait dans la première version. C'est un nœud 
communautaire (n8n-nodes-serpapi) : il n'est pas présent dans une installation neuve et 
peut être supprimé ou installé séparément (voir Dépannage).

# Prérequis

Élément	                 Détail

n8n	                    Testé en local, puis avec Docker Desktop
OpenAI	                Clé API avec crédits (un abonnement ChatGPT ne suffit pas)
Google Cloud	        Un projet avec un client OAuth de type « Application Web »
Google Drive	        Pour stocker le fichier de connaissance
Google Sheets	        Un tableur pour les prospects
Google Calendar	        Un agenda dédié (ici nommé « Agence Bali »)
Slack	                Un espace de travail et une application avec un bot

# Installation

# 1. Lancer n8n avec Docker

docker run -it --rm --name n8n -p 5678:5678 \-v n8n_data:/home/node/.n8n \docker.n8n.io/n8nio/n8n

- Le volume n8n_data conserve tes workflows et credentials. Sans lui, tout est perdu à la suppression du conteneur.
- Pour un lancement en arrière-plan, retire --rm et ajoute -d.
- n8n est ensuite disponible sur http://localhost:5678.

# 2. Importer le workflow
    1- Dans n8n, crée un nouveau workflow.
    2- Menu ··· → Import from file → choisis workflow.json.