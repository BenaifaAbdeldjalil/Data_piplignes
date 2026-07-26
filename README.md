# Data_piplignes
TUTORIEL COMPLET : Construire un Pipeline Data avec Python
🏟️ TUTORIEL COMPLET : Construire un Pipeline Data avec Python
📌 AVANT DE COMMENCER — La philosophie
Quand on fait du data engineering, on ne "bricole" pas. On suit des patterns (patrons de conception) qui sont des standards de l'industrie. Ce que tu vas apprendre ici, c'est exactement ce que font les data engineers dans les vraies entreprises.
Les 3 règles d'or de ce projet :
  1. Séparation des responsabilités — chaque fichier fait UNE chose bien
  2. Configuration externe — jamais de secrets dans le code
  3. Reproductibilité — n'importe qui (ou toi dans 6 mois) peut relancer le projet

ÉTAPE 0 — Vérifier ton environnement Windows
0.1 Ouvrir PowerShell
Appuie sur la touche Windows, tape PowerShell, puis Entrée.
Qu'est-ce que PowerShell ? C'est le terminal de Windows. Un terminal, c'est une fenêtre où tu écris des commandes textuelles au lieu de cliquer. C'est plus rapide et reproductible que les clics.

Ouvre PowerShell en tant qu'administrateur (clic droit → "Exécuter en tant qu'administrateur") et lance :
winget install -e --id Python.Python.3.12
Note : Si winget n'est pas disponible, télécharge Python 3.12 depuis python.org et coche "Add Python to PATH" lors de l'installation.


0.2 Vérifier Python
Dans PowerShell, colle cette commande :
powershell
python --version
Ce que ça fait : demande à Windows quel Python est installé.
Résultat attendu :
plain
Python 3.12.x
Si tu vois une erreur : télécharge Python sur https://www.python.org/downloads/ — coche "Add Python to PATH" pendant l'install.

0.3 Vérifier pip

  pip --version
Ce que c'est : pip est le "magasin d'applications" de Python. Il télécharge et installe des packages (bibliothèques de code faites par d'autres).

ÉTAPE 1 — Créer le dossier projet et l'environnement virtuel
Configurer PowerShell pour les scripts
Toujours en PowerShell administrateur :

Set-ExecutionPolicy -Scope CurrentUser RemoteSigned

Créer la structure du projet
1.1 Créer le dossier principal
# Va dans ton dossier Documents
cd $env:USERPROFILE\Documents

# Crée le dossier du projet
mkdir football_pipeline

# Rentre dedans
cd football_pipeline

Pourquoi $env:USERPROFILE ? C'est une variable d'environnement Windows qui pointe toujours vers C:\Users\TonNom. Ça évite d'écrire ton nom d'utilisateur en dur.

1.2 Créer l'environnement virtuel (venv)
python -m venv .venv
si erreur test: 
python -m venv .venv --without-pip

🧠 CONCEPT CLÉ — L'environnement virtuel :
Quand tu installes un package Python (comme pandas), par défaut il s'installe globalement sur ton PC. Le problème :
  # Ton projet A a besoin de pandas 1.5
  # Ton projet B a besoin de pandas 2.0
Conflit ! Tu ne peux pas avoir les deux en global.
La solution : un venv crée une "bulle" Python isolée dans ton dossier projet. Les packages installés dans .venv\Lib\site-packages n'interfèrent avec rien d'autre.
  # Le .venv est DEDANS football_pipeline/ — c'est le standard. Quand tu ouvres le dossier dans VS Code, tout est au même endroit.

Donc 
# football_pipeline\.venv

1.3 Activer le venv
.venv\Scripts\Activate.ps1

Ce que ça fait : modifie temporairement ta session PowerShell pour que quand tu tapes python, Windows utilise C:\...\football_pipeline\.venv\Scripts\python.exe au lieu du Python global.
Résultat attendu : ton prompt change :
(.venv) PS C:\Users\...\Documents\football_pipeline>
# Le (.venv) au début confirme que c'est activé.
# 💡 À retenir : à CHAQUE nouvelle fenêtre PowerShell, tu dois refaire cette commande. C'est normal.

1.4 Vérifier que le bon Python est utilisé
where python

Résultat attendu :
C:\Users\...\Documents\football_pipeline\.venv\Scripts\python.exe
C:\Users\...\AppData\Local\Programs\Python\Python312\python.exe

Le premier chemin doit contenir .venv — c'est celui qui est actif.


# ÉTAPE 2 — Créer la structure des dossiers
# Dans VS Code, crée cette arborescence exacte. Clic droit → New Folder.
# ####################################################################
football_pipeline/           ← Tu es ici
├── .venv/                   ← Déjà créé automatiquement
├── config/
├── data/
│   ├── raw/
│   │   ├── api/
│   │   └── kaggle/
│   └── processed/
├── scripts/
│   ├── ingestion/
│   └── transform/
├── sql/
│   ├── staging/
│   └── analytics/
├── logs/
└── tests/

sur powershell vscode:
Créer la structure des dossiers
# Depuis le dossier football_pipeline/
New-Item -ItemType Directory -Path "config" -Force
New-Item -ItemType Directory -Path "data\raw\api" -Force
New-Item -ItemType Directory -Path "data\raw\kaggle" -Force
New-Item -ItemType Directory -Path "data\processed" -Force
New-Item -ItemType Directory -Path "scripts\ingestion" -Force
New-Item -ItemType Directory -Path "scripts\transform" -Force
New-Item -ItemType Directory -Path "sql\staging" -Force
New-Item -ItemType Directory -Path "sql\analytics" -Force
New-Item -ItemType Directory -Path "logs" -Force
New-Item -ItemType Directory -Path "tests" -Force

Créer les fichiers de base :
# Fichiers Python avec contenu minimal
New-Item -ItemType File -Path "scripts\ingestion\__init__.py" -Force
New-Item -ItemType File -Path "scripts\transform\__init__.py" -Force
New-Item -ItemType File -Path "tests\__init__.py" -Force

# Fichiers de configuration
New-Item -ItemType File -Path "config\config.yaml" -Force
New-Item -ItemType File -Path "config\secrets.yaml" -Force

# Fichiers SQL vides
New-Item -ItemType File -Path "sql\staging\create_staging.sql" -Force
New-Item -ItemType File -Path "sql\analytics\analytics_queries.sql" -Force

# Fichiers racine
New-Item -ItemType File -Path "README.md" -Force
New-Item -ItemType File -Path "requirements.txt" -Force
New-Item -ItemType File -Path ".gitignore" -Force
New-Item -ItemType File -Path "main.py" -Force
New-Item -ItemType File -Path "docker-compose.yml" -Force
New-Item -ItemType File -Path "Dockerfile" -Force


# ####################################################################
🧠 POURQUOI cette structure ?
| Dossier              | Rôle                             | Analogie                         |
| -------------------- | -------------------------------- | -------------------------------- |
| `config/`            | Toute la config centralisée      | Le tableau de bord de la voiture |
| `data/raw/`          | Données brutes, jamais modifiées | Les ingrédients crus             |
| `data/processed/`    | Données transformées             | Le plat cuisiné                  |
| `scripts/ingestion/` | Code qui RÉCUPÈRE les données    | Le livreur                       |
| `scripts/transform/` | Code qui NETTOIE les données     | Le chef                          |
| `sql/staging/`       | Requêtes de création de tables   | Les plans de la cuisine          |
| `sql/analytics/`     | Requêtes de reporting            | Le menu du restaurant            |
| `logs/`              | Fichiers de suivi d'exécution    | Le journal de bord               |

2.1 Créer les fichiers __init__.py
Dans chaque dossier de code (config/, scripts/, scripts/ingestion/, scripts/transform/, tests/), crée un fichier vide nommé exactement :
__init__.py

🧠 CONCEPT CLÉ — Le __init__.py :
En Python, un dossier avec un __init__.py devient un package — un module importable. Sans ce fichier, Python refuse d'importer des fichiers depuis ce dossier.
C'est une particularité historique de Python. Dans les versions récentes (3.3+), il existe les "namespace packages" sans __init__.py, mais tout le monde continue de les mettre. C'est le standard.

# ÉTAPE 3 — Créer requirements.txt
Crée un fichier à la racine nommé requirements.txt et colle :

# === Core ===
requests>=2.31.0
python-dotenv>=1.0.0

# === Database ===
psycopg2-binary>=2.9.9

# === Data Processing ===
pandas>=2.1.0

# === Utils ===
loguru>=0.7.0

# Méthode powerschell : Créer avec echo (simple)
echo "pandas" > requirements.txt
echo "requests" >> requirements.txt
echo "psycopg2-binary" >> requirements.txt
echo "sqlalchemy" >> requirements.txt
echo "python-dotenv" >> requirements.txt
echo "pyyaml" >> requirements.txt
echo "kagglehub" >> requirements.txt
echo "jupyter" >> requirements.txt
echo "pytest" >> requirements.txt
echo "dotenv" >> requirements.txt
echo "psycopg2" >> requirements.txt

NB : 
>> ajout
> ecrase

🧠 POURQUOI chaque package ?
| Package           | À quoi ça sert                    | Analogie                             |
| ----------------- | --------------------------------- | ------------------------------------ |
| `requests`        | Faire des appels HTTP (API web)   | Le téléphone pour appeler l'API      |
| `python-dotenv`   | Lire le fichier `.env`            | Le coffre-fort qui garde tes secrets |
| `psycopg2-binary` | Se connecter à PostgreSQL         | Le traducteur Python ↔ PostgreSQL    |
| `pandas`          | Manipuler des tableaux de données | Excel, mais en code                  |
| `loguru`          | Écrire des logs propres           | Le journal de bord détaillé          |

Les >= signifient "cette version ou plus récente". Le .0 à la fin est le numéro de version (semantic versioning : MAJOR.MINOR.PATCH).

3.1 Installer les packages

# Syntaxe de base
pip install -r [nom_du_fichier]
pip install -r requirements.txt

Ce que ça fait : pip lit le fichier, télécharge chaque package depuis PyPI (le magasin officiel Python), et les installe dans .venv\Lib\site-packages\.

Vérification :
pip list
Tu dois voir requests, python-dotenv, psycopg2-binary, pandas, loguru dans la liste.


ÉTAPE 4 — Créer le fichier .env (sécurité)
Crée un fichier .env à la racine (avec le point devant) :
# Depuis le dossier football_pipeline/
New-Item -ItemType File -Path ".env" -Force

# === API Keys ===
FOOTBALL_API_KEY=ta_cle_api_ici

# === PostgreSQL ===
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=football_staging
POSTGRES_USER=postgres
POSTGRES_PASSWORD=ton_mot_de_passe


# Créer le fichier .env avec le contenu
# Depuis le dossier football_pipeline/ 

@"
# === API Keys ===
FOOTBALL_API_KEY=ta_cle_api_ici

# === PostgreSQL ===
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=football_staging
POSTGRES_USER=postgres
POSTGRES_PASSWORD=ton_mot_de_passe
"@ | Out-File -FilePath ".env" -Encoding utf8


🧠 CONCEPT CLÉ — Pourquoi un .env ?
Imaginons que tu publies ton projet sur GitHub. Si ta clé API est écrite en dur dans le code Python :
  1. Tout le monde la voit
  2. Quelqu'un peut la voler et faire des appels à ton nom
  3. L'API peut te bannir pour abus
La solution industrielle : stocker les secrets dans un fichier .env (non versionné) et le code lit ce fichier au démarrage.
4.1 Créer .gitignore
Crée un fichier .gitignore à la racine :
# Depuis le dossier football_pipeline/ 
@"
# Python
__pycache__/
*.py[cod]
.venv/

# Secrets
.env
.env.local

# Données (trop volumineux)
data/raw/*
data/processed/*
!data/raw/.gitkeep
!data/processed/.gitkeep

# Logs
logs/*.log
"@ | Out-File -FilePath ".gitignore" -Encoding utf8

🧠 CONCEPT CLÉ — .gitignore :
Ce fichier dit à Git : "ne versionne PAS ces fichiers". C'est crucial :
  .venv/ → plusieurs centaines de Mo, inutile sur GitHub
  .env → contient tes secrets
  data/raw/* → les JSON peuvent faire des Mo
Les lignes !data/raw/.gitkeep sont une astuce : on ignore tout dans data/raw/ sauf le fichier .gitkeep. Ce fichier vide sert juste à ce que Git crée le dossier vide sur GitHub (sinon un dossier vide n'est pas versionné).

# ÉTAPE 5 — Créer config/settings.py
Crée le dossier config/ et le fichier config/settings.py. Colle ce code :

@"
"""
Configuration centralisée du pipeline Football Data.
Toutes les variables d'environnement sont chargées ici.
"""

import os
from pathlib import Path
from dotenv import load_dotenv

# ============================================================
# CHARGEMENT DU FICHIER .ENV
# ============================================================
# load_dotenv() cherche un fichier .env à la racine du projet
# et charge toutes les variables comme des variables d'environnement
load_dotenv()


# ============================================================
# CHEMINS DU PROJET
# ============================================================
# Path(__file__) = chemin absolu de CE fichier (settings.py)
# .resolve() = convertit en chemin absolu propre
# .parent.parent = remonte de 2 niveaux : config/ → football_pipeline/
BASE_DIR = Path(__file__).resolve().parent.parent

# On construit les chemins des sous-dossiers
DATA_DIR = BASE_DIR / "data"
RAW_API_DIR = DATA_DIR / "raw" / "api"
RAW_KAGGLE_DIR = DATA_DIR / "raw" / "kaggle"
PROCESSED_DIR = DATA_DIR / "processed"
LOGS_DIR = BASE_DIR / "logs"

# Créer les dossiers s'ils n'existent pas encore
# parents=True → crée aussi les dossiers parents si manquants
# exist_ok=True → ne plante pas si le dossier existe déjà
for d in [RAW_API_DIR, RAW_KAGGLE_DIR, PROCESSED_DIR, LOGS_DIR]:
    d.mkdir(parents=True, exist_ok=True)


# ============================================================
# VARIABLES API
# ============================================================
# os.getenv("NOM") lit la variable d'environnement NOM
# Si elle n'existe pas, elle retourne None (ou la valeur par défaut si fournie)
FOOTBALL_API_KEY = os.getenv("FOOTBALL_API_KEY")
FOOTBALL_API_BASE_URL = "https://api.football-data.org/v4"


# ============================================================
# CONFIGURATION POSTGRESQL
# ============================================================
# On regroupe les paramètres dans un dictionnaire (dict)
# C'est plus propre que 5 variables séparées
POSTGRES_CONFIG = {
    "host": os.getenv("POSTGRES_HOST", "localhost"),
    "port": int(os.getenv("POSTGRES_PORT", "5432")),
    "database": os.getenv("POSTGRES_DB", "football_staging"),
    "user": os.getenv("POSTGRES_USER", "postgres"),
    "password": os.getenv("POSTGRES_PASSWORD", ""),
}


# ============================================================
# FONCTIONS DE VÉRIFICATION
# ============================================================

def check_api_key():
    """
    Vérifie que la clé API est bien configurée.
    Si non, lève une exception avec un message explicite.
    """
    if not FOOTBALL_API_KEY:
        raise ValueError(
            "❌ FOOTBALL_API_KEY non trouvée !\n"
            "   1. Crée un fichier .env à la racine du projet\n"
            "   2. Ajoute : FOOTBALL_API_KEY=ta_cle_api"
        )
    # Affiche les 4 derniers caractères pour confirmer sans exposer la clé
    print(f"✅ Clé API configurée (derniers 4 caractères : ...{FOOTBALL_API_KEY[-4:]})")


def check_postgres():
    """
    Vérifie que PostgreSQL est accessible.
    Retourne True si OK, False sinon.
    """
    import psycopg2
    try:
        # **POSTGRES_CONFIG "déplie" le dictionnaire en arguments nommés
        # Équivalent à : host="localhost", port=5432, database="...", etc.
        conn = psycopg2.connect(**POSTGRES_CONFIG)
        
        cur = conn.cursor()
        cur.execute("SELECT version();")
        version = cur.fetchone()[0]
        
        cur.close()
        conn.close()
        
        print(f"✅ PostgreSQL connecté : {version.split(',')[0]}")
        return True
    except Exception as e:
        print(f"❌ Erreur PostgreSQL : {e}")
        return False


# ============================================================
# BLOC D'EXÉCUTION DIRECTE
# ============================================================
# Ce bloc ne s'exécute QUE si on lance ce fichier directement
# (python config/settings.py)
# Il ne s'exécute PAS si on importe settings dans un autre fichier
if __name__ == "__main__":
    print("🔧 Vérification de la configuration...")
    check_api_key()
    check_postgres()
    print(f"📁 Dossier data : {DATA_DIR}")


"@ | Out-File -FilePath config/settings.py  -Encoding utf8

# Python attend du UTF-8. Le caractère \xff est un marqueur d'ordre d'octets (BOM) qui indique un encodage différent.


🧠 CONCEPTS PYTHON expliqués ligne par ligne :
| Concept                      | Explication simple                                                                |
| ---------------------------- | --------------------------------------------------------------------------------- |
| `import os`                  | Importe le module système d'exploitation (chemins, variables d'env)               |
| `from pathlib import Path`   | `Path` est la façon moderne de gérer les chemins de fichiers (remplace `os.path`) |
| `Path(__file__)`             | Chemin absolu du fichier en cours d'exécution                                     |
| `.resolve()`                 | Nettoie le chemin (résout les `..`, les liens symboliques)                        |
| `.parent`                    | Remonte d'un dossier                                                              |
| `os.getenv("X")`             | Lit la variable d'environnement `X`                                               |
| `dict = {"clé": "valeur"}`   | Un dictionnaire = collection de paires clé-valeur                                 |
| `**dict`                     | "Dépliage" d'un dictionnaire en arguments nommés                                  |
| `if __name__ == "__main__":` | Pattern standard : code qui ne tourne qu'en exécution directe                     |


5.1 Tester la config
python config/settings.py

Résultat attendu :
🔧 Vérification de la configuration...
✅ Clé API configurée (derniers 4 caractères : ...XXXX)
✅ PostgreSQL connecté : PostgreSQL 16.x
📁 Dossier data : C:\Users\...\Documents\football_pipeline\data


erreur : SyntaxError: Non-UTF-8 code starting with '\xff' in file
Cette erreur signifie que votre fichier settings.py est encodé en UTF-16 (ou autre) alors que Python attend du UTF-8. Le caractère \xff est un marqueur d'ordre d'octets (BOM) qui indique un encodage différent.
sollution rajoutée -Encoding utf8 à la creation du fichier

erreur rencontrée:
ModuleNotFoundError: No module named 'dotenv'
Le message indique que python-dotenv est installé globalement (dans le cache de l'utilisateur) mais pas dans votre environnement virtuel .venv.

SOLUTION 1 : Forcer l'installation dans .venv
# Utiliser python -m pip au lieu de pip seul
python -m pip install python-dotenv

python -c "import dotenv; print('✅ dotenv fonctionne !')"
resultat : ✅ dotenv fonctionne !

Utiliser un fichier requirements.txt

# Installer depuis requirements.txt avec python -m pip
python -m pip install -r requirements.txt


Retester la config
python config/settings.py

✅ Clé API configurée (derniers 4 caractères : ...2999)
❌ Erreur PostgreSQL : 'utf-8' codec can't decode byte 0xe9 in position 84: invalid continuation byte

# API==OK
# BDD=KO

# ÉTAPE 6 — Créer le script d'ingestion API
Crée scripts/ingestion/fetch_api_data.py et colle :
@"
"""
Script d'ingestion des données depuis l'API football-data.org.
Sauvegarde les réponses JSON brutes dans data/raw/api/


"""

import json
import sys
from datetime import datetime
from pathlib import Path

import requests
from loguru import logger

# ============================================================
# AJOUT DU DOSSIER RACINE AU PYTHONPATH
# ============================================================
# sys.path est la liste des dossiers où Python cherche les imports.
# On ajoute la racine du projet pour pouvoir faire :
#   from config.settings import ...
# sans erreur "ModuleNotFoundError"
sys.path.insert(0, str(Path(__file__).resolve().parent.parent))

from config.settings import (
    FOOTBALL_API_KEY,
    FOOTBALL_API_BASE_URL,
    RAW_API_DIR,
    check_api_key,
)


# ============================================================
# CONFIGURATION DES LOGS
# ============================================================
# loguru remplace le module logging standard de Python.
# C'est plus simple et plus beau.
# rotation="1 day" = crée un nouveau fichier chaque jour
# retention="7 days" = garde les 7 derniers jours
LOGS_DIR = Path(__file__).resolve().parent.parent / "logs"
LOGS_DIR.mkdir(exist_ok=True)

logger.add(
    LOGS_DIR / "ingestion_api_{time}.log",
    rotation="1 day",
    retention="7 days",
    level="INFO",
)


# ============================================================
# CLASSE CLIENT API
# ============================================================
# En Python, une "classe" est un moule qui crée des "objets".
# Ici, FootballAPIClient est un objet qui sait parler à l'API.
class FootballAPIClient:
    """
    Client pour interagir avec l'API football-data.org.
    Gère les requêtes, le rate limiting et la sauvegarde.
    """

    def __init__(self, api_key: str):
        """
        Constructeur : s'exécute quand on crée l'objet.
        Le "self" représente l'instance elle-même.
        """
        self.api_key = api_key
        self.base_url = FOOTBALL_API_BASE_URL
        
        # Dictionnaire des headers HTTP (métadonnées de la requête)
        self.headers = {
            "X-Auth-Token": api_key,      # Clé d'authentification
            "Content-Type": "application/json",  # Format attendu
        }
        
        # Session = connexion persistante, plus rapide que des requêtes isolées
        self.session = requests.Session()
        self.session.headers.update(self.headers)

    def _request(self, endpoint: str, params: dict = None) -> dict:
        """
        Méthode privée (underscore = convention "ne pas utiliser de l'extérieur")
        Effectue une requête GET et retourne le JSON.
        
        Args:
            endpoint: Chemin de l'endpoint (ex: /competitions/PL/standings)
            params: Paramètres de requête optionnels (dict)
            
        Returns:
            dict: Réponse JSON de l'API
        """
        url = f"{self.base_url}{endpoint}"
        logger.info(f"🌐 Requête : {url}")
        
        try:
            # GET = récupérer des données (comme taper une URL dans le navigateur)
            response = self.session.get(url, params=params, timeout=30)
            
            # raise_for_status() lève une exception si le code HTTP est 4xx ou 5xx
            response.raise_for_status()
            
            # Vérifier le rate limit (combien de requêtes il me reste)
            remaining = response.headers.get("X-Requests-Available-Minute", "N/A")
            logger.info(f"⏱️ Requêtes restantes/min : {remaining}")
            
            # .json() parse la réponse texte en dictionnaire Python
            return response.json()
            
        except requests.exceptions.HTTPError as e:
            if response.status_code == 403:
                logger.error("❌ Erreur 403 : Clé API invalide ou quota dépassé")
            elif response.status_code == 404:
                logger.error(f"❌ Erreur 404 : Endpoint non trouvé - {url}")
            else:
                logger.error(f"❌ Erreur HTTP {response.status_code} : {e}")
            raise  # "Relance" l'exception pour que l'appelant sache qu'il y a eu un problème
        except requests.exceptions.RequestException as e:
            logger.error(f"❌ Erreur de connexion : {e}")
            raise

    def save_raw(self, data: dict, filename: str) -> Path:
        """
        Sauvegarde les données JSON brutes sur disque.
        
        Args:
            data: Données à sauvegarder (dictionnaire Python)
            filename: Nom de base du fichier (sans extension ni date)
            
        Returns:
            Path: Chemin complet du fichier créé
        """
        # datetime.now() = date/heure actuelle
        # strftime = formatte la date en texte : 20250726_143022
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        filepath = RAW_API_DIR / f"{filename}_{timestamp}.json"
        
        # "w" = mode écriture, "utf-8" = encodage universel (accents supportés)
        with open(filepath, "w", encoding="utf-8") as f:
            # json.dump = écrit un dict Python en texte JSON
            # ensure_ascii=False = garde les accents (é, è, à)
            # indent=2 = formatte avec des sauts de ligne (lisible)
            json.dump(data, f, ensure_ascii=False, indent=2)
        
        file_size = filepath.stat().st_size
        logger.info(f"💾 Sauvegardé : {filepath.name} ({file_size:,} octets)")
        return filepath

    # ==================== ENDPOINTS DE L'API ====================

    def get_competitions(self) -> dict:
        """Récupère la liste de toutes les compétitions disponibles."""
        return self._request("/competitions")

    def get_competition_standings(self, competition_code: str) -> dict:
        """
        Récupère le classement d'une compétition.
        
        Args:
            competition_code: Code de la compétition 
                PL = Premier League (Angleterre)
                FL1 = Ligue 1 (France)
                BL1 = Bundesliga (Allemagne)
                SA = Serie A (Italie)
                PD = Primera División (Espagne)
        """
        return self._request(f"/competitions/{competition_code}/standings")

    def get_competition_matches(self, competition_code: str, matchday: int = None) -> dict:
        """
        Récupère les matchs d'une compétition.
        
        Args:
            competition_code: Code de la compétition
            matchday: Numéro de journée (optionnel)
        """
        params = {}
        if matchday:
            params["matchday"] = matchday
        return self._request(f"/competitions/{competition_code}/matches", params)

    def get_competition_teams(self, competition_code: str) -> dict:
        """Récupère les équipes d'une compétition."""
        return self._request(f"/competitions/{competition_code}/teams")


# ============================================================
# FONCTION PRINCIPALE
# ============================================================

def main():
    """
    Fonction principale : orchestre toute l'ingestion.
    C'est le "chef d'orchestre" qui appelle les autres méthodes.
    """
    print("=" * 60)
    print("🏟️  PIPELINE FOOTBALL - INGESTION API")
    print("=" * 60)

    # Vérifier que la config est OK
    check_api_key()
    print()

    # Créer une instance du client API
    # "client" est un objet FootballAPIClient avec ta clé API
    client = FootballAPIClient(FOOTBALL_API_KEY)

    # --- ÉTAPE 1 : Liste des compétitions ---
    print("\n📋 Étape 1/5 : Récupération des compétitions...")
    competitions = client.get_competitions()
    client.save_raw(competitions, "competitions")
    
    # len() = nombre d'éléments dans une liste
    # .get("competitions", []) = récupère la clé "competitions", ou une liste vide si absente
    count = len(competitions.get("competitions", []))
    print(f"   → {count} compétitions trouvées")

    # --- ÉTAPE 2 : Top 5 ligues européennes ---
    top_leagues = ["PL", "FL1", "BL1", "SA", "PD"]
    
    print("\n🏆 Étape 2/5 : Récupération des classements (Top 5)...")
    for league in top_leagues:
        try:
            # try/except = "essaie ce bloc, et si ça plante, exécute le except"
            standings = client.get_competition_standings(league)
            client.save_raw(standings, f"standings_{league}")
            print(f"   ✅ {league} : classement sauvegardé")
        except Exception as e:
            # Si une erreur survient (quota dépassé, compétition indisponible...)
            print(f"   ⚠️  {league} : {e}")

    # --- ÉTAPE 3 : Matchs des Top 5 ---
    print("\n⚽ Étape 3/5 : Récupération des matchs (Top 5)...")
    for league in top_leagues:
        try:
            matches = client.get_competition_matches(league)
            client.save_raw(matches, f"matches_{league}")
            total = matches.get("resultSet", {}).get("count", 0)
            print(f"   ✅ {league} : {total} matchs sauvegardés")
        except Exception as e:
            print(f"   ⚠️  {league} : {e}")

    # --- ÉTAPE 4 : Équipes ---
    print("\n👕 Étape 4/5 : Récupération des équipes (Top 5)...")
    for league in top_leagues:
        try:
            teams = client.get_competition_teams(league)
            client.save_raw(teams, f"teams_{league}")
            count = len(teams.get("teams", []))
            print(f"   ✅ {league} : {count} équipes sauvegardées")
        except Exception as e:
            print(f"   ⚠️  {league} : {e}")

    # --- ÉTAPE 5 : Récapitulatif ---
    print("\n📊 Étape 5/5 : Récapitulatif...")
    
    # list() convertit un "itérateur" en liste
    # .glob("*.json") = trouve tous les fichiers .json dans le dossier
    files = list(RAW_API_DIR.glob("*.json"))
    
    # sum() additionne tous les éléments
    # f.stat().st_size = taille du fichier en octets
    total_size = sum(f.stat().st_size for f in files)
    
    print(f"   📁 {len(files)} fichiers JSON créés")
    print(f"   💾 Taille totale : {total_size / 1024:.1f} Ko")
    print(f"   📂 Dossier : {RAW_API_DIR}")

    print("\n" + "=" * 60)
    print("✅ INGESTION TERMINÉE AVEC SUCCÈS !")
    print("=" * 60)


# ============================================================
# POINT D'ENTRÉE DU SCRIPT
# ============================================================
if __name__ == "__main__":
    main()

"@ | Out-File -FilePath "scripts/ingestion/fetch_api_data.py" -Encoding utf8

🧠 CONCEPTS PYTHON expliqués :
| Concept                      | Explication                                                                     |
| ---------------------------- | ------------------------------------------------------------------------------- |
| `class`                      | Moule pour créer des objets. `client = FootballAPIClient("clé")` crée un objet. |
| `def __init__(self, ...)`    | Constructeur : s'exécute à la création de l'objet.                              |
| `self`                       | Référence à l'objet lui-même. `self.api_key` = "la clé API DE CET objet".       |
| `-> dict`                    | Type hint : indique que la fonction retourne un dictionnaire.                   |
| `try / except`               | Gestion d'erreurs : "essaie, et si ça plante, fais ça".                         |
| `raise`                      | Relance une exception pour que l'appelant soit informé.                         |
| `f"...{variable}..."`        | f-string : insère une variable dans une chaîne de texte.                        |
| `.get("clé", valeur_defaut)` | Récupère une clé d'un dict, ou valeur\_defaut si absente.                       |
| `for x in liste:`            | Boucle : répète le bloc pour chaque élément.                                    |
| `list comprehension`         | `[f for f in files if "machin" in f.name]` = filtrage élégant.                  |

6.1 Lancer l'ingestion
python scripts/ingestion/fetch_api_data.py

# ÉTAPE 7 — Créer la base PostgreSQL
Ouvre pgAdmin (installé avec PostgreSQL) ou un nouveau terminal PowerShell.
7.1 Via pgAdmin (recommandé pour débutant)
  Ouvre pgAdmin
  Dans l'arborescence à gauche, clic droit sur Databases → Create → Database
  Onglet General :
    Database : football_staging
    Owner : postgres
  Clic sur Save
7.2 Alternative : ligne de commande
# Se connecter à PostgreSQL (il demande ton mot de passe)
psql -U postgres

# Dans le prompt psql, taper :
CREATE DATABASE football_staging WITH OWNER = postgres ENCODING = 'UTF8';
\q

# ÉTAPE 8 — Créer le script de chargement PostgreSQL
Crée scripts/ingestion/load_to_postgres.py et colle :
"""
Chargement des données JSON brutes vers PostgreSQL (tables staging).
Transforme les JSON en lignes SQL puis les insère en base.
"""

import json
import sys
from pathlib import Path

import psycopg2
from loguru import logger
from psycopg2.extras import execute_values

sys.path.insert(0, str(Path(__file__).resolve().parent.parent))
from config.settings import RAW_API_DIR, POSTGRES_CONFIG, check_postgres

LOGS_DIR = Path(__file__).resolve().parent.parent / "logs"
LOGS_DIR.mkdir(exist_ok=True)
logger.add(LOGS_DIR / "postgres_load_{time}.log", rotation="1 day", retention="7 days")


class PostgresLoader:
    """
    Charge les données JSON dans PostgreSQL.
    Encapsule toute la logique de connexion et d'insertion.
    """

    def __init__(self):
        self.config = POSTGRES_CONFIG
        self.conn = None

    def connect(self):
        """Ouvre la connexion PostgreSQL."""
        self.conn = psycopg2.connect(**self.config)
        logger.info(f"✅ Connecté à PostgreSQL ({self.config['database']})")
        return self

    def close(self):
        """Ferme proprement la connexion."""
        if self.conn:
            self.conn.close()
            logger.info("🔌 Connexion PostgreSQL fermée")

    def execute(self, sql: str, params=None):
        """Exécute une requête SQL simple."""
        with self.conn.cursor() as cur:
            cur.execute(sql, params)
            self.conn.commit()

    def create_staging_tables(self):
        """
        Crée les tables staging si elles n'existent pas.
        Le mot-clé IF NOT EXISTS évite une erreur si la table existe déjà.
        """
        logger.info("🏗️  Création des tables staging...")

        ddl = """
        -- Table des compétitions
        CREATE TABLE IF NOT EXISTS staging_competitions (
            id INTEGER PRIMARY KEY,
            name VARCHAR(100),
            code VARCHAR(10),
            type VARCHAR(20),
            emblem VARCHAR(500),
            current_season_start DATE,
            current_season_end DATE,
            current_matchday INTEGER,
            area_name VARCHAR(100),
            area_code VARCHAR(10),
            loaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        );

        -- Table des équipes
        CREATE TABLE IF NOT EXISTS staging_teams (
            id INTEGER PRIMARY KEY,
            name VARCHAR(100),
            short_name VARCHAR(50),
            tla VARCHAR(10),
            crest VARCHAR(500),
            address VARCHAR(300),
            website VARCHAR(200),
            founded INTEGER,
            club_colors VARCHAR(100),
            venue VARCHAR(100),
            competition_code VARCHAR(10),
            loaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        );

        -- Table des matchs
        CREATE TABLE IF NOT EXISTS staging_matches (
            id BIGINT PRIMARY KEY,
            competition_id INTEGER,
            competition_name VARCHAR(100),
            season VARCHAR(20),
            matchday INTEGER,
            status VARCHAR(20),
            utc_date TIMESTAMP,
            home_team_id INTEGER,
            home_team_name VARCHAR(100),
            away_team_id INTEGER,
            away_team_name VARCHAR(100),
            home_score INTEGER,
            away_score INTEGER,
            winner VARCHAR(20),
            referee VARCHAR(100),
            venue VARCHAR(100),
            loaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        );

        -- Table des classements
        CREATE TABLE IF NOT EXISTS staging_standings (
            id SERIAL PRIMARY KEY,
            competition_code VARCHAR(10),
            position INTEGER,
            team_id INTEGER,
            team_name VARCHAR(100),
            played_games INTEGER,
            won INTEGER,
            draw INTEGER,
            lost INTEGER,
            points INTEGER,
            goals_for INTEGER,
            goals_against INTEGER,
            goal_difference INTEGER,
            loaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        );
        """
        self.execute(ddl)
        logger.info("✅ Tables staging créées")

    def load_competitions(self, filepath: Path):
        """Charge le fichier competitions.json."""
        logger.info(f"📥 Chargement competitions : {filepath.name}")

        with open(filepath, "r", encoding="utf-8") as f:
            data = json.load(f)

        rows = []
        for comp in data.get("competitions", []):
            season = comp.get("currentSeason", {})
            area = comp.get("area", {})
            # Chaque "row" est un tuple qui correspond à une ligne SQL
            rows.append((
                comp.get("id"),
                comp.get("name"),
                comp.get("code"),
                comp.get("type"),
                comp.get("emblem"),
                season.get("startDate"),
                season.get("endDate"),
                season.get("currentMatchday"),
                area.get("name"),
                area.get("code"),
            ))

        # execute_values = insertion BULK (plus rapide que INSERT ligne par ligne)
        sql = """
            INSERT INTO staging_competitions 
            (id, name, code, type, emblem, current_season_start, current_season_end, 
             current_matchday, area_name, area_code)
            VALUES %s
            ON CONFLICT (id) DO UPDATE SET
                name = EXCLUDED.name,
                current_matchday = EXCLUDED.current_matchday,
                loaded_at = CURRENT_TIMESTAMP;
        """
        with self.conn.cursor() as cur:
            execute_values(cur, sql, rows)
            self.conn.commit()

        logger.info(f"   ✅ {len(rows)} compétitions insérées/mises à jour")

    def load_teams(self, filepath: Path):
        """Charge un fichier teams_XXX.json."""
        logger.info(f"📥 Chargement teams : {filepath.name}")

        with open(filepath, "r", encoding="utf-8") as f:
            data = json.load(f)

        comp_code = data.get("competition", {}).get("code", "UNK")
        rows = []
        for team in data.get("teams", []):
            rows.append((
                team.get("id"),
                team.get("name"),
                team.get("shortName"),
                team.get("tla"),
                team.get("crest"),
                team.get("address"),
                team.get("website"),
                team.get("founded"),
                team.get("clubColors"),
                team.get("venue"),
                comp_code,
            ))

        sql = """
            INSERT INTO staging_teams 
            (id, name, short_name, tla, crest, address, website, founded, 
             club_colors, venue, competition_code)
            VALUES %s
            ON CONFLICT (id) DO UPDATE SET
                name = EXCLUDED.name,
                short_name = EXCLUDED.short_name,
                competition_code = EXCLUDED.competition_code,
                loaded_at = CURRENT_TIMESTAMP;
        """
        with self.conn.cursor() as cur:
            execute_values(cur, sql, rows)
            self.conn.commit()

        logger.info(f"   ✅ {len(rows)} équipes insérées/mises à jour")

    def load_matches(self, filepath: Path):
        """Charge un fichier matches_XXX.json."""
        logger.info(f"📥 Chargement matches : {filepath.name}")

        with open(filepath, "r", encoding="utf-8") as f:
            data = json.load(f)

        comp = data.get("competition", {})
        season = data.get("filters", {}).get("season", "")
        rows = []
        for match in data.get("matches", []):
            home = match.get("homeTeam", {})
            away = match.get("awayTeam", {})
            score = match.get("score", {}).get("fullTime", {})
            winner = match.get("score", {}).get("winner", "")
            
            # Récupère le premier arbitre, ou None s'il n'y en a pas
            referees = match.get("referees", [])
            referee_name = referees[0].get("name") if referees else None
            
            rows.append((
                match.get("id"),
                comp.get("id"),
                comp.get("name"),
                season,
                match.get("matchday"),
                match.get("status"),
                match.get("utcDate"),
                home.get("id"),
                home.get("name"),
                away.get("id"),
                away.get("name"),
                score.get("home"),
                score.get("away"),
                winner,
                referee_name,
                match.get("venue"),
            ))

        sql = """
            INSERT INTO staging_matches 
            (id, competition_id, competition_name, season, matchday, status, utc_date,
             home_team_id, home_team_name, away_team_id, away_team_name,
             home_score, away_score, winner, referee, venue)
            VALUES %s
            ON CONFLICT (id) DO UPDATE SET
                status = EXCLUDED.status,
                home_score = EXCLUDED.home_score,
                away_score = EXCLUDED.away_score,
                winner = EXCLUDED.winner,
                loaded_at = CURRENT_TIMESTAMP;
        """
        with self.conn.cursor() as cur:
            execute_values(cur, sql, rows)
            self.conn.commit()

        logger.info(f"   ✅ {len(rows)} matchs insérés/mis à jour")

    def load_standings(self, filepath: Path):
        """Charge un fichier standings_XXX.json."""
        logger.info(f"📥 Chargement standings : {filepath.name}")

        with open(filepath, "r", encoding="utf-8") as f:
            data = json.load(f)

        comp_code = data.get("competition", {}).get("code", "UNK")
        rows = []
        for standing in data.get("standings", []):
            # On ne garde que le classement TOTAL (pas HOME/AWAY)
            if standing.get("type") != "TOTAL":
                continue
            for entry in standing.get("table", []):
                team = entry.get("team", {})
                rows.append((
                    comp_code,
                    entry.get("position"),
                    team.get("id"),
                    team.get("name"),
                    entry.get("playedGames"),
                    entry.get("won"),
                    entry.get("draw"),
                    entry.get("lost"),
                    entry.get("points"),
                    entry.get("goalsFor"),
                    entry.get("goalsAgainst"),
                    entry.get("goalDifference"),
                ))

        # Pour les standings, on supprime les anciennes données de cette compétition
        # car le classement évolue chaque semaine
        with self.conn.cursor() as cur:
            cur.execute("DELETE FROM staging_standings WHERE competition_code = %s", (comp_code,))
            
            sql = """
                INSERT INTO staging_standings 
                (competition_code, position, team_id, team_name, played_games,
                 won, draw, lost, points, goals_for, goals_against, goal_difference)
                VALUES %s;
            """
            execute_values(cur, sql, rows)
            self.conn.commit()

        logger.info(f"   ✅ {len(rows)} lignes de classement insérées")

    def load_all(self):
        """Orchestre le chargement de tous les fichiers JSON."""
        print("\n" + "=" * 60)
        print("🐘 CHARGEMENT POSTGRESQL - TABLES STAGING")
        print("=" * 60)

        check_postgres()
        self.connect()
        self.create_staging_tables()

        # Récupère tous les fichiers JSON du dossier raw/api/
        files = sorted(RAW_API_DIR.glob("*.json"))
        
        # 1. Competitions (à charger en premier car les autres en dépendent)
        for f in [f for f in files if "competitions" in f.name]:
            self.load_competitions(f)

        # 2. Teams
        for f in [f for f in files if "teams_" in f.name]:
            self.load_teams(f)

        # 3. Matches
        for f in [f for f in files if "matches_" in f.name]:
            self.load_matches(f)

        # 4. Standings
        for f in [f for f in files if "standings_" in f.name]:
            self.load_standings(f)

        self.close()

        print("\n" + "=" * 60)
        print("✅ CHARGEMENT POSTGRESQL TERMINÉ !")
        print("=" * 60)
        print("\nVérification rapide :")
        self._print_summary()

    def _print_summary(self):
        """Affiche un récapitulatif des données chargées."""
        self.connect()
        tables = ["staging_competitions", "staging_teams", "staging_matches", "staging_standings"]
        for table in tables:
            with self.conn.cursor() as cur:
                cur.execute(f"SELECT COUNT(*) FROM {table}")
                count = cur.fetchone()[0]
                print(f"   📊 {table:<30} : {count:>6} lignes")
        self.close()


def main():
    loader = PostgresLoader()
    loader.load_all()


if __name__ == "__main__":
    main()


🧠 CONCEPTS expliqués :
| Concept                               | Explication                                                                          |
| ------------------------------------- | ------------------------------------------------------------------------------------ |
| `ON CONFLICT (id) DO UPDATE`          | PostgreSQL : si l'ID existe déjà, met à jour au lieu de planter. C'est l'**UPSERT**. |
| `execute_values`                      | Insère plusieurs lignes en UNE requête SQL. 100x plus rapide que 100 INSERT.         |
| `SERIAL PRIMARY KEY`                  | Colonne auto-incrémentée (1, 2, 3...) qui sert de clé primaire.                      |
| `TIMESTAMP DEFAULT CURRENT_TIMESTAMP` | PostgreSQL remplit automatiquement avec la date/heure actuelle.                      |
| `with self.conn.cursor() as cur:`     | Context manager : ouvre un curseur, l'utilise, le ferme automatiquement.             |
| `DELETE FROM ... WHERE`               | Supprime les anciennes données avant d'insérer les nouvelles (rafraîchissement).     |


8.1 Lancer le chargement
python scripts/ingestion/load_to_postgres.py


# ÉTAPE 9 — Vérifier avec SQL
Ouvre pgAdmin ou DBeaver (ou psql), connecte-toi à football_staging, et exécute :

-- Compter les lignes par table
SELECT 'staging_competitions' AS table_name, COUNT(*) AS nb_lignes FROM staging_competitions
UNION ALL
SELECT 'staging_teams', COUNT(*) FROM staging_teams
UNION ALL
SELECT 'staging_matches', COUNT(*) FROM staging_matches
UNION ALL
SELECT 'staging_standings', COUNT(*) FROM staging_standings;


Top 5 Premier League :
SELECT team_name, played_games, won, draw, lost, points, goal_difference
FROM staging_standings
WHERE competition_code = 'PL'
ORDER BY position
LIMIT 5;

Derniers matchs :
SELECT 
    m.utc_date,
    m.home_team_name,
    m.home_score,
    m.away_score,
    m.away_team_name
FROM staging_matches m
WHERE m.competition_name LIKE '%Premier%'
ORDER BY m.utc_date DESC
LIMIT 10;

────────────────────────────────────────────────────────────────────────────────────────────────────────────
✅ RÉCAPITULATIF — Ce que tu as construit
┌──────────────────────────────────────────────────────────────┐
│  🏟️ PHASE 1 — INGESTION API  ✅ TERMINÉE                    │
│                                                              │
│  football-data.org API                                       │
│       │                                                      │
│       ▼                                                      │
│  FootballAPIClient (classe Python)                           │
│       │  • get_competitions()                                │
│       │  • get_competition_standings("PL")                   │
│       │  • get_competition_matches("PL")                     │
│       │  • get_competition_teams("PL")                       │
│       ▼                                                      │
│  data/raw/api/*.json  (JSON brut, jamais modifié)           │
├──────────────────────────────────────────────────────────────┤
│  🐘 PHASE 2 — STAGING PostgreSQL  ✅ TERMINÉE               │
│                                                              │
│  JSON brut  ──►  PostgresLoader (classe Python)               │
│       │  • parse JSON → tuples                               │
│       │  • execute_values (insertion bulk)                 │
│       │  • ON CONFLICT DO UPDATE (upsert)                    │
│       ▼                                                      │
│  PostgreSQL : football_staging                               │
│      ├── staging_competitions                                │
│      ├── staging_teams                                       │
│      ├── staging_matches                                     │
│      └── staging_standings                                   │
└──────────────────────────────────────────────────────────────┘

────────────────────────────────────────────────────────────────────────────────────────────────────────────

🚀 Prochaines étapes
| Phase                   | Ce qu'on fera                                     | Quand tu veux |
| ----------------------- | ------------------------------------------------- | ------------- |
| **3 — Databricks**      | Notebook PySpark, architecture Bronze/Silver/Gold | Dis "Phase 3" |
| **4 — Snowflake + dbt** | Warehouse, modèles dbt, tests                     | Dis "Phase 4" |
| **5 — Power BI**        | Connecteur, modèle sémantique, dashboards         | Dis "Phase 5" |


Databricks Lakehouse. C'est là que ton pipeline devient vraiment professionnel
sandbox:///mnt/agents/output/football_pipeline.zip

# 🏔️ PHASE 3 — Databricks Lakehouse : Bronze → Silver → Gold
🧠 QU'EST-CE QU'UN LAKEHOUSE ?
Avant de coder, comprenons le concept.
| Architecture                                  | Problème                                                      | Solution                    |
| --------------------------------------------- | ------------------------------------------------------------- | --------------------------- |
| **Data Lake** (S3, ADLS)                      | Données brutes, pas de structure, pas de transactions         | Stockage bon marché         |
| **Data Warehouse** (Snowflake)                | Structuré, cher, pas de données brutes                        | Requêtes rapides, SQL       |
| **Lakehouse** (Databricks) = Lake + Warehouse | Combine les deux : données brutes + transactions + SQL rapide | Le meilleur des deux mondes |

Le pattern Medallion (Médaille) organise les données en 3 couches :
  🥉 Bronze = Données brutes, immuables (historique complet)
  🥈 Silver = Données nettoyées, typées, dédupliquées
  🥇 Gold = Données agrégées, prêtes pour le BI

# ÉTAPE 1 — Créer ton compte Databricks Community Edition
1.1 Inscription
  Va sur https://community.cloud.databricks.com/
  Clique sur "Try Databricks Free"
  Remplis le formulaire avec ton email
  Vérifie ton email et connecte-toi
    Qu'est-ce que Community Edition ? C'est la version gratuite de Databricks. Limitations : cluster qui s'éteint après 2h d'inactivité, pas de collaboration, mais tout le code PySpark fonctionne exactement pareil que la version payante.
# ÉTAPE 2 — Créer ton cluster Spark
2.1 Dans l'interface Databricks
  Dans la barre latérale gauche, clique sur "Compute"
  Clique le bouton bleu "Create Compute"
  Remplis :
    Cluster name : football-cluster
    Single Node : ✅ Coche cette case (obligatoire pour CE)
    Databricks Runtime : choisis le plus récent (ex: 15.4 LTS)
  Clique "Create Compute"
  Attends 2-3 minutes que le statut passe de "Pending" → "Running"

🧠 CONCEPT — Qu'est-ce qu'un cluster Spark ?
Spark est un moteur de calcul distribué. Au lieu de traiter les données sur ton PC (1 processeur), Spark répartit le travail sur plusieurs machines. Même en "Single Node" (1 machine), Spark optimise l'exécution en mémoire (RAM) au lieu de disque, ce qui est 10-100x plus rapide que pandas pour de gros volumes.

# ÉTAPE 3 — Récupérer tes identifiants Databricks
3.1 Personal Access Token
  En haut à droite, clique sur ton nom → "User Settings"
  Va dans l'onglet "Access Tokens"
  Clique "Generate New Token"
  Comment : pipeline-token
  Lifetime : 90 days
  ⚠️ IMPORTANT : Copie le token affiché (il commence par dapi...). Tu ne pourras plus jamais le revoir !

3.2 Cluster ID
  Dans Compute, clique sur ton cluster football-cluster
  Regarde l'URL de ton navigateur :
    https://community.cloud.databricks.com/?o=123456789#setting/clusters/XXXX-XXXXXX-XXXXXXX/configuration
  Le Cluster ID est la partie XXXX-XXXXXX-XXXXXXX

3.3 Mettre à jour ton .env
Ajoute ces lignes dans ton fichier .env (à la racine de football_pipeline/) :

# === Databricks (Community Edition) ===
DATABRICKS_HOST=https://community.cloud.databricks.com
DATABRICKS_TOKEN=dapiXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
DATABRICKS_CLUSTER_ID=XXXX-XXXXXX-XXXXXXX


# ÉTAPE 4 — Upload tes données JSON dans Databricks
🚨 Problème à résoudre
Ton PostgreSQL est sur ton PC (localhost). Databricks CE tourne dans le cloud de Databricks. Ils ne peuvent pas se parler (ton PC est protégé par un firewall, et Databricks CE n'a pas accès à ton réseau local).
La solution : on upload les fichiers JSON directement dans l'espace de stockage de Databricks, appelé DBFS (Databricks File System).

4.1 Option A — Upload manuel (le plus simple)
  Dans Databricks, clique sur "Catalog" dans la barre latérale
  Clique sur "DBFS"
  Navigue dans /FileStore/
  Crée le dossier : clic droit → "Create Directory" → football_pipeline
  Dans football_pipeline, crée raw
  Dans raw, crée api
  Glisse-dépose tous tes fichiers .json depuis data/raw/api/ de ton PC vers ce dossier DBFS
4.2 Option B — Script Python automatique
  Si tu veux automatiser, j'ai créé un script. Mais pour l'instant, l'upload manuel est plus simple et fiable.


# ÉTAPE 5 — Créer le notebook PySpark

5.1 Créer le notebook
  Dans Databricks, clique sur "New" (en haut à gauche) → "Notebook"
  Name : lakehouse_bronze_silver_gold
  Default Language : Python
  Cluster : sélectionne football-cluster
  Clique "Create"
5.2 Comprendre l'interface
| Élément              | À quoi ça sert                                     |
| -------------------- | -------------------------------------------------- |
| **Cellule**          | Bloc de code que tu exécutes avec `Shift + Entrée` |
| **%md**              | Cellule Markdown (texte, pas de code)              |
| **%sql**             | Cellule SQL (tu peux écrire du SQL pur)            |
| **display(df)**      | Affiche un tableau interactif (comme Excel)        |
| **df.printSchema()** | Affiche la structure des colonnes                  |

# ÉTAPE 6 — Le notebook complet (copier-coller cellule par cellule)

Voici le code complet. Dans Databricks, crée une nouvelle cellule pour chaque bloc COMMAND ----------.
CELLULE 1 — Markdown (titre)

# Databricks notebook source
# MAGIC %md
# MAGIC # 🏔️ PHASE 3 — Lakehouse Databricks : Bronze → Silver → Gold
# MAGIC 
# MAGIC Architecture **Medallion** sur les données football.


🧠 CONCEPT : # MAGIC %md transforme la cellule en texte formaté (Markdown). C'est pour la documentation.


CELLULE 2 — Configuration
# COMMAND ----------

# Chemin DBFS où sont stockés les fichiers JSON
RAW_PATH = "/dbfs/FileStore/football_pipeline/raw/api"

# Chemins de sortie pour chaque couche
BRONZE_PATH = "/dbfs/FileStore/football_pipeline/bronze"
SILVER_PATH = "/dbfs/FileStore/football_pipeline/silver"
GOLD_PATH   = "/dbfs/FileStore/football_pipeline/gold"

CELLULE 3 — Markdown Bronze
# COMMAND ----------

# MAGIC %md
# MAGIC ---
# MAGIC # 🥉 COUCHE BRONZE — Données brutes
# MAGIC 
# MAGIC **Principe** : On lit les JSON exactement comme ils arrivent de l'API. 
# MAGIC AUCUNE transformation. C'est l'historique immuable.


CELLULE 4 — Lire les JSON en Bronze
# COMMAND ----------

# ============================================================
# BRONZE : LECTURE DES JSON BRUTS
# ============================================================

# spark.read.json() = lit tous les fichiers .json dans le dossier
# Spark infère AUTOMATIQUEMENT le schéma (structure des colonnes)
# option("multiLine", "true") = JSON avec objets imbriqués (tableaux, objets)

bronze_competitions = (
    spark
    .read
    .option("multiLine", "true")
    .json(f"{RAW_PATH}/competitions_*.json")
)

# printSchema() = affiche la structure des colonnes (comme DESC en SQL)
print("📋 Schéma BRONZE — Competitions :")
bronze_competitions.printSchema()


🧠 CONCEPTS PySpark expliqués :
| Code                           | Explication                                                                                                            |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------- |
| `spark`                        | L'objet SparkSession créé automatiquement par Databricks. C'est le point d'entrée de tout le code Spark.               |
| `.read`                        | Démarre une opération de lecture.                                                                                      |
| `.json("chemin")`              | Lit des fichiers JSON. Le `*` est un wildcard (tous les fichiers qui commencent par `competitions_`).                  |
| `.option("multiLine", "true")` | Le JSON de l'API a des objets sur plusieurs lignes (avec des retours à la ligne). Sans cette option, Spark planterait. |
| `.printSchema()`               | Affiche le schéma : nom des colonnes, types (string, int, struct...), et si nullable.                                  |


CELLULE 5 — Aperçu Bronze
# COMMAND ----------

# display() = fonction magique Databricks qui fait un joli tableau interactif
# C'est comme un Excel dans le navigateur
print("📊 Aperçu BRONZE — Competitions :")
bronze_competitions.display(5)


CELLULE 6 — Lire tous les fichiers Bronze
# COMMAND ----------

bronze_standings = (
    spark
    .read
    .option("multiLine", "true")
    .json(f"{RAW_PATH}/standings_*.json")
)

bronze_matches = (
    spark
    .read
    .option("multiLine", "true")
    .json(f"{RAW_PATH}/matches_*.json")
)

bronze_teams = (
    spark
    .read
    .option("multiLine", "true")
    .json(f"{RAW_PATH}/teams_*.json")
)

print("✅ Tous les fichiers BRONZE chargés en mémoire")
print(f"   Competitions : {bronze_competitions.count()} lignes")
print(f"   Standings    : {bronze_standings.count()} lignes")
print(f"   Matches      : {bronze_matches.count()} lignes")
print(f"   Teams        : {bronze_teams.count()} lignes")


🧠 CONCEPT — Lazy Evaluation : Quand tu fais spark.read.json(...), Spark ne lit RIEN encore. Il construit juste un "plan d'exécution". C'est quand tu fais .count() ou .display() qu'il lit réellement les données. C'est comme préparer une recette sans cuisiner — tu cuisines seulement quand quelqu'un a faim.

CELLULE 7 — Sauvegarder Bronze en Delta Lake
# COMMAND ----------

# ============================================================
# SAUVEGARDE BRONZE EN DELTA LAKE
# ============================================================

# Delta Lake = format de stockage natif de Databricks
# Avantages par rapport à JSON/CSV :
# - ACID : transactions fiables (pas de données corrompues à moitié écrites)
# - Time Travel : revenir à une version précédente des données
# - Schema Enforcement : le schéma est contrôlé et versionné
# - 10-100x plus rapide que JSON/CSV

bronze_competitions.write \
    .format("delta") \
    .mode("overwrite") \
    .save(f"{BRONZE_PATH}/competitions")

bronze_standings.write \
    .format("delta") \
    .mode("overwrite") \
    .save(f"{BRONZE_PATH}/standings")

bronze_matches.write \
    .format("delta") \
    .mode("overwrite") \
    .save(f"{BRONZE_PATH}/matches")

bronze_teams.write \
    .format("delta") \
    .mode("overwrite") \
    .save(f"{BRONZE_PATH}/teams")

print("✅ Couche BRONZE sauvegardée en Delta Lake")


🧠 CONCEPTS expliqués :

| Code                 | Explication                                                                                            |
| -------------------- | ------------------------------------------------------------------------------------------------------ |
| `.write`             | Démarre une opération d'écriture.                                                                      |
| `.format("delta")`   | Écrit au format Delta Lake (parquet + log de transactions).                                            |
| `.mode("overwrite")` | Écrase les anciennes données. Autres modes : `"append"` (ajoute), `"ignore"` (ne fait rien si existe). |
| `.save("chemin")`    | Écrit les données dans DBFS.                                                                           |

CELLULE 8 — Markdown Silver
# COMMAND ----------

# MAGIC %md
# MAGIC ---
# MAGIC # 🥈 COUCHE SILVER — Données nettoyées
# MAGIC 
# MAGIC **Principe** : On nettoie, on type, on dénormalise, on supprime les doublons.

CELLULE 9 — Silver Competitions
# COMMAND ----------

# ============================================================
# SILVER : NETTOYAGE DES COMPÉTITIONS
# ============================================================

from pyspark.sql.functions import (
    col, explode, to_date, trim, upper, current_date
)

# --- Silver Competitions ---
# explode() = "aplatit" un tableau JSON en lignes
#   Exemple : {"competitions": [{"id": 1}, {"id": 2}]} 
#   → explode crée 2 lignes : id=1 et id=2

silver_competitions = (
    bronze_competitions
    .select(explode("competitions").alias("comp"))  # comp = chaque objet du tableau
    .select(
        # col("comp.id") = accède au champ "id" de l'objet "comp"
        # .cast("int") = convertit en entier (au lieu de string)
        col("comp.id").cast("int").alias("competition_id"),
        
        # trim() = supprime les espaces au début et à la fin
        trim(col("comp.name")).alias("competition_name"),
        
        # upper() = met en majuscules
        upper(col("comp.code")).alias("competition_code"),
        
        col("comp.type").alias("competition_type"),
        col("comp.emblem").alias("emblem_url"),
        
        # to_date() = convertit une string "2024-08-16" en vraie date
        to_date(col("comp.currentSeason.startDate")).alias("season_start"),
        to_date(col("comp.currentSeason.endDate")).alias("season_end"),
        
        col("comp.currentSeason.currentMatchday").cast("int").alias("current_matchday"),
        col("comp.area.name").alias("area_name"),
        col("comp.area.code").alias("area_code"),
        
        # current_date() = date du jour (pour tracer quand on a chargé)
        current_date().alias("silver_loaded_date")
    )
    # Supprime les doublons sur competition_id (garde le premier)
    .dropDuplicates(["competition_id"])
)

print("📋 Schéma SILVER — Competitions :")
silver_competitions.printSchema()
silver_competitions.display(5)



🧠 CONCEPTS PySpark expliqués :
| Fonction                  | À quoi ça sert                                                                                    |
| ------------------------- | ------------------------------------------------------------------------------------------------- |
| `col("nom")`              | Référence une colonne du DataFrame.                                                               |
| `explode("colonne")`      | Transforme un tableau `[{a:1}, {a:2}]` en lignes séparées. Indispensable pour les JSON imbriqués. |
| `.cast("int")`            | Convertit le type (string → int, string → date...).                                               |
| `alias("nouveau_nom")`    | Renomme la colonne.                                                                               |
| `trim()`                  | Supprime les espaces parasites (très courant dans les données réelles).                           |
| `upper()`                 | Met en majuscules (standardiser les codes).                                                       |
| `to_date()`               | Convertit une chaîne de texte en objet date.                                                      |
| `dropDuplicates(["col"])` | Supprime les lignes en double selon la colonne donnée.                                            |
| `current_date()`          | Fonction Spark qui retourne la date du jour.                                                      |



CELLULE 10 — Silver Teams
# COMMAND ----------

# ============================================================
# SILVER : NETTOYAGE DES ÉQUIPES
# ============================================================

silver_teams = (
    bronze_teams
    .select(explode("teams").alias("team"))
    .select(
        col("team.id").cast("int").alias("team_id"),
        trim(col("team.name")).alias("team_name"),
        trim(col("team.shortName")).alias("short_name"),
        upper(col("team.tla")).alias("tla_code"),
        col("team.crest").alias("crest_url"),
        trim(col("team.address")).alias("address"),
        col("team.website"),
        col("team.founded").cast("int").alias("founded_year"),
        trim(col("team.clubColors")).alias("club_colors"),
        trim(col("team.venue")).alias("venue"),
        upper(col("competition.code")).alias("competition_code"),
        current_date().alias("silver_loaded_date")
    )
    .dropDuplicates(["team_id"])
)

print(f"✅ Silver Teams : {silver_teams.count()} équipes")
silver_teams.display(5)



CELLULE 11 — Silver Matches
# COMMAND ----------

# ============================================================
# SILVER : NETTOYAGE DES MATCHS
# ============================================================

silver_matches = (
    bronze_matches
    .select(explode("matches").alias("match"))
    .select(
        col("match.id").cast("bigint").alias("match_id"),
        col("match.competition.id").cast("int").alias("competition_id"),
        trim(col("match.competition.name")).alias("competition_name"),
        col("match.season").alias("season_year"),
        col("match.matchday").cast("int").alias("matchday"),
        trim(col("match.status")).alias("status"),
        col("match.utcDate").cast("timestamp").alias("match_datetime"),
        col("match.homeTeam.id").cast("int").alias("home_team_id"),
        trim(col("match.homeTeam.name")).alias("home_team_name"),
        col("match.awayTeam.id").cast("int").alias("away_team_id"),
        trim(col("match.awayTeam.name")).alias("away_team_name"),
        col("match.score.fullTime.home").cast("int").alias("home_score"),
        col("match.score.fullTime.away").cast("int").alias("away_score"),
        trim(col("match.score.winner")).alias("winner"),
        # Récupère le premier arbitre, ou NULL s'il n'y en a pas
        col("match.referees")[0]["name"].alias("main_referee"),
        trim(col("match.venue")).alias("venue"),
        current_date().alias("silver_loaded_date")
    )
    .dropDuplicates(["match_id"])
)

print(f"✅ Silver Matches : {silver_matches.count()} matchs")
silver_matches.display(5)


CELLULE 12 — Silver Standings
# COMMAND ----------

# ============================================================
# SILVER : NETTOYAGE DES CLASSEMENTS
# ============================================================

# Les standings sont plus complexes :
#   standings = [ {type: "TOTAL", table: [...]}, {type: "HOME", table: [...]} ]
# On ne garde que type = "TOTAL" (classement global)

silver_standings = (
    bronze_standings
    .select(
        upper(col("competition.code")).alias("competition_code"),
        explode("standings").alias("standing")
    )
    # filter() = équivalent SQL : WHERE type = 'TOTAL'
    .filter(col("standing.type") == "TOTAL")
    .select(
        col("competition_code"),
        explode("standing.table").alias("entry")
    )
    .select(
        col("competition_code"),
        col("entry.position").cast("int").alias("position"),
        col("entry.team.id").cast("int").alias("team_id"),
        trim(col("entry.team.name")).alias("team_name"),
        col("entry.playedGames").cast("int").alias("played_games"),
        col("entry.won").cast("int").alias("won"),
        col("entry.draw").cast("int").alias("draw"),
        col("entry.lost").cast("int").alias("lost"),
        col("entry.points").cast("int").alias("points"),
        col("entry.goalsFor").cast("int").alias("goals_for"),
        col("entry.goalsAgainst").cast("int").alias("goals_against"),
        col("entry.goalDifference").cast("int").alias("goal_difference"),
        current_date().alias("silver_loaded_date")
    )
    .dropDuplicates(["competition_code", "team_id"])
)

print(f"✅ Silver Standings : {silver_standings.count()} lignes")
silver_standings.display(5)


🧠 CONCEPT — .filter() : C'est le WHERE de PySpark. .filter(col("standing.type") == "TOTAL") garde uniquement les lignes où le type est TOTAL.

CELLULE 13 — Sauvegarder Silver
# COMMAND ----------

silver_competitions.write.format("delta").mode("overwrite").save(f"{SILVER_PATH}/competitions")
silver_teams.write.format("delta").mode("overwrite").save(f"{SILVER_PATH}/teams")
silver_matches.write.format("delta").mode("overwrite").save(f"{SILVER_PATH}/matches")
silver_standings.write.format("delta").mode("overwrite").save(f"{SILVER_PATH}/standings")

print("✅ Couche SILVER sauvegardée en Delta Lake")

CELLULE 14 — Markdown Gold
# COMMAND ----------

# MAGIC %md
# MAGIC ---
# MAGIC # 🥇 COUCHE GOLD — Données analytics-ready
# MAGIC 
# MAGIC **Principe** : Agrégations, jointures, calculs de KPIs. 
# MAGIC Optimisé pour les requêtes BI (Power BI, Tableau).


CELLULE 15 — Gold Matches enrichis
# COMMAND ----------

# ============================================================
# GOLD : TABLE DES MATCHS AVEC STATISTIQUES
# ============================================================

from pyspark.sql.functions import (
    when, abs, count, sum as spark_sum, avg, max as spark_max
)

gold_matches = (
    silver_matches
    .select(
        col("match_id"),
        col("competition_id"),
        col("competition_name"),
        col("season_year"),
        col("matchday"),
        col("status"),
        col("match_datetime"),
        col("home_team_id"),
        col("home_team_name"),
        col("away_team_id"),
        col("away_team_name"),
        col("home_score"),
        col("away_score"),
        # Calcul du résultat pour l'équipe à DOMICILE
        # when(condition, valeur_si_vrai).when(condition2, valeur2).otherwise(valeur_defaut)
        when(col("home_score") > col("away_score"), "W")   # Win
        .when(col("home_score") < col("away_score"), "L")  # Loss
        .otherwise("D")                                     # Draw
        .alias("home_result"),
        # Calcul du résultat pour l'équipe à l'EXTÉRIEUR
        when(col("away_score") > col("home_score"), "W")
        .when(col("away_score") < col("home_score"), "L")
        .otherwise("D")
        .alias("away_result"),
        # Total de buts dans le match
        (col("home_score") + col("away_score")).alias("total_goals"),
        # Différence de buts (valeur absolue)
        abs(col("home_score") - col("away_score")).alias("goal_diff"),
        col("winner"),
        col("main_referee"),
        col("venue")
    )
)

print(f"✅ Gold Matches : {gold_matches.count()} lignes")
gold_matches.display(5)



🧠 CONCEPTS expliqués :
| Fonction                  | À quoi ça sert                                                                    |
| ------------------------- | --------------------------------------------------------------------------------- |
| `when(condition, valeur)` | Équivalent du `CASE WHEN` en SQL. Chaînage : `when(...).when(...).otherwise(...)` |
| `abs()`                   | Valeur absolue (toujours positive).                                               |
| `sum as spark_sum`        | On renomme `sum` car c'est aussi un mot-clé Python.                               |
| `col("a") + col("b")`     | Opération arithmétique entre colonnes (comme en SQL : `a + b`).                   |


CELLULE 16 — Gold Team Stats (agrégation)
# COMMAND ----------

# ============================================================
# GOLD : STATISTIQUES PAR ÉQUIPE (DOMICILE + EXTÉRIEUR)
# ============================================================

# On calcule les stats à domicile ET à l'extérieur, puis on les joint

# --- Stats à DOMICILE ---
home_stats = (
    gold_matches
    .groupBy("competition_name", "home_team_id", "home_team_name")
    .agg(
        # count(*) = nombre de matchs
        count("*").alias("home_played"),
        # spark_sum(when(...)) = compte conditionnellement
        spark_sum(when(col("home_result") == "W", 1).otherwise(0)).alias("home_won"),
        spark_sum(when(col("home_result") == "D", 1).otherwise(0)).alias("home_draw"),
        spark_sum(when(col("home_result") == "L", 1).otherwise(0)).alias("home_lost"),
        spark_sum("home_score").alias("home_goals_for"),
        spark_sum("away_score").alias("home_goals_against"),
        # Points : 3 pour une victoire, 1 pour un nul, 0 sinon
        spark_sum(
            when(col("home_result") == "W", 3)
            .when(col("home_result") == "D", 1)
            .otherwise(0)
        ).alias("home_points")
    )
    .withColumnRenamed("home_team_id", "team_id")
    .withColumnRenamed("home_team_name", "team_name")
)

# --- Stats à l'EXTÉRIEUR ---
away_stats = (
    gold_matches
    .groupBy("competition_name", "away_team_id", "away_team_name")
    .agg(
        count("*").alias("away_played"),
        spark_sum(when(col("away_result") == "W", 1).otherwise(0)).alias("away_won"),
        spark_sum(when(col("away_result") == "D", 1).otherwise(0)).alias("away_draw"),
        spark_sum(when(col("away_result") == "L", 1).otherwise(0)).alias("away_lost"),
        spark_sum("away_score").alias("away_goals_for"),
        spark_sum("home_score").alias("away_goals_against"),
        spark_sum(
            when(col("away_result") == "W", 3)
            .when(col("away_result") == "D", 1)
            .otherwise(0)
        ).alias("away_points")
    )
    .withColumnRenamed("away_team_id", "team_id")
    .withColumnRenamed("away_team_name", "team_name")
)

# --- Jointure DOMICILE + EXTÉRIEUR ---
# .join(df2, ["col1", "col2"], "outer") = FULL OUTER JOIN
# "outer" = garde TOUTES les lignes, même si une équipe n'a joué que à domicile
gold_team_stats = (
    home_stats
    .join(away_stats, ["competition_name", "team_id", "team_name"], "outer")
    # fillna(0) = remplace les NULL par 0 (équipe qui n'a pas joué à l'extérieur par exemple)
    .fillna(0)
    .select(
        col("competition_name"),
        col("team_id"),
        col("team_name"),
        (col("home_played") + col("away_played")).alias("total_played"),
        (col("home_won") + col("away_won")).alias("total_won"),
        (col("home_draw") + col("away_draw")).alias("total_draw"),
        (col("home_lost") + col("away_lost")).alias("total_lost"),
        (col("home_points") + col("away_points")).alias("total_points"),
        (col("home_goals_for") + col("away_goals_for")).alias("goals_for"),
        (col("home_goals_against") + col("away_goals_against")).alias("goals_against"),
        (col("goals_for") - col("goals_against")).alias("goal_difference"),
        # Taux de victoire en pourcentage
        (col("total_won") / col("total_played") * 100).alias("win_rate_pct"),
        # Buts par match
        (col("goals_for") / col("total_played")).alias("goals_per_game")
    )
    # orderBy(desc("col")) = ORDER BY col DESC
    .orderBy(col("total_points").desc(), col("goal_difference").desc())
)

print(f"✅ Gold Team Stats : {gold_team_stats.count()} équipes")
gold_team_stats.display(10)


🧠 CONCEPTS expliqués :
| Fonction                        | À quoi ça sert                                                |
| ------------------------------- | ------------------------------------------------------------- |
| `.groupBy("col")`               | Regroupe les lignes par valeur unique (comme `GROUP BY` SQL). |
| `.agg(...)`                     | Applique des fonctions d'agrégation sur chaque groupe.        |
| `count("*")`                    | Compte le nombre de lignes dans chaque groupe.                |
| `spark_sum(when(...))`          | Somme conditionnelle : compte seulement les victoires.        |
| `.join(df2, ["cols"], "outer")` | Jointure. `"outer"` = FULL OUTER JOIN (garde tout).           |
| `.fillna(0)`                    | Remplace les valeurs NULL par 0.                              |
| `.orderBy(col("x").desc())`     | Trie par ordre décroissant.                                   |


CELLULE 17 — Gold Standings (avec rang)
# COMMAND ----------

# ============================================================
# GOLD : CLASSEMENT FINAL AVEC RANG
# ============================================================

from pyspark.sql.window import Window
from pyspark.sql.functions import dense_rank, desc

# Window = définit une "fenêtre" de calcul
# partitionBy("competition_name") = calcule le rang PAR compétition
# orderBy(desc("total_points"), ...) = ordre du classement
rank_window = Window.partitionBy("competition_name").orderBy(
    desc("total_points"), desc("goal_difference"), desc("goals_for")
)

gold_standings = (
    gold_team_stats
    # dense_rank() = attribue un rang (1, 2, 2, 3...) sans trou dans la numérotation
    .withColumn("position", dense_rank().over(rank_window))
    .select(
        col("competition_name"),
        col("position"),
        col("team_id"),
        col("team_name"),
        col("total_played"),
        col("total_won"),
        col("total_draw"),
        col("total_lost"),
        col("total_points"),
        col("goals_for"),
        col("goals_against"),
        col("goal_difference"),
        col("win_rate_pct"),
        col("goals_per_game")
    )
)

print("✅ Gold Standings (recalculé avec rang) :")
gold_standings.display(10)


🧠 CONCEPT — Window Functions :
C'est l'équivalent des fonctions fenêtre en SQL (ROW_NUMBER(), RANK(), DENSE_RANK()).

Window.partitionBy("competition_name").orderBy(desc("total_points"))

Signifie : "Pour chaque compétition, classe les équipes par points décroissants".
| Fonction       | Comportement                     |
| -------------- | -------------------------------- |
| `row_number()` | 1, 2, 3, 4 (toujours unique)     |
| `rank()`       | 1, 2, 2, 4 (trou après ex-aequo) |
| `dense_rank()` | 1, 2, 2, 3 (pas de trou)         |


CELLULE 18 — Gold Match Stats (par compétition)
# COMMAND ----------

# ============================================================
# GOLD : STATISTIQUES GLOBALES PAR COMPÉTITION
# ============================================================

gold_match_stats = (
    gold_matches
    .groupBy("competition_name")
    .agg(
        count("*").alias("total_matches"),
        spark_sum("total_goals").alias("total_goals_scored"),
        avg("total_goals").alias("avg_goals_per_match"),
        spark_max("total_goals").alias("max_goals_in_match"),
        spark_sum(when(col("home_result") == "W", 1).otherwise(0)).alias("home_wins"),
        spark_sum(when(col("away_result") == "W", 1).otherwise(0)).alias("away_wins"),
        spark_sum(when(col("home_result") == "D", 1).otherwise(0)).alias("draws"),
        (spark_sum("home_score") / count("*")).alias("avg_home_goals"),
        (spark_sum("away_score") / count("*")).alias("avg_away_goals")
    )
    .withColumn("home_win_pct", col("home_wins") / col("total_matches") * 100)
    .withColumn("away_win_pct", col("away_wins") / col("total_matches") * 100)
    .withColumn("draw_pct", col("draws") / col("total_matches") * 100)
)

print("✅ Gold Match Stats par compétition :")
gold_match_stats.display()

CELLULE 19 — Sauvegarder Gold
# COMMAND ----------

gold_matches.write.format("delta").mode("overwrite").save(f"{GOLD_PATH}/matches")
gold_team_stats.write.format("delta").mode("overwrite").save(f"{GOLD_PATH}/team_stats")
gold_standings.write.format("delta").mode("overwrite").save(f"{GOLD_PATH}/standings")
gold_match_stats.write.format("delta").mode("overwrite").save(f"{GOLD_PATH}/match_stats")

print("✅ Couche GOLD sauvegardée en Delta Lake")


CELLULE 20 — Vérification finale
# COMMAND ----------

# ============================================================
# VÉRIFICATION FINALE
# ============================================================

print("📊 RÉCAPITULATIF DU LAKEHOUSE")
print("=" * 50)

for layer, path in [("BRONZE", BRONZE_PATH), ("SILVER", SILVER_PATH), ("GOLD", GOLD_PATH)]:
    print(f"\n🏷️  {layer}:")
    # dbutils.fs.ls() = liste les fichiers/dossiers dans DBFS
    tables = dbutils.fs.ls(path)
    for t in tables:
        if t.isDir():
            df = spark.read.format("delta").load(t.path)
            print(f"   📁 {t.name:<25} : {df.count():>6} lignes")

print("\n" + "=" * 50)
print("✅ PHASE 3 TERMINÉE !")


✅ RÉCAPITULATIF — Ce que tu as construit
┌─────────────────────────────────────────────────────────────────────┐
│                    🏔️ DATABRICKS LAKEHOUSE                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  🥉 BRONZE (brut)                                                  │
│  ├── bronze/competitions     ← JSON competitions tel quel          │
│  ├── bronze/teams            ← JSON teams tel quel                 │
│  ├── bronze/matches          ← JSON matches tel quel               │
│  └── bronze/standings       ← JSON standings tel quel              │
│                                                                     │
│  🥈 SILVER (nettoyé)                                               │
│  ├── silver/competitions     ← types OK, dates, pas de doublons    │
│  ├── silver/teams           ← trim, upper, founded en int          │
│  ├── silver/matches         ← scores en int, timestamps            │
│  └── silver/standings       ← filtré TOTAL uniquement              │
│                                                                     │
│  🥇 GOLD (analytics)                                               │
│  ├── gold/matches           ← + home_result, away_result, totals   │
│  ├── gold/team_stats        ← stats domicile + extérieur agrégées │
│  ├── gold/standings         ← classement recalculé avec RANK()     │
│  └── gold/match_stats       ← stats globales par compétition       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

📊 Tableau récapitulatif des fonctions PySpark apprises
| Fonction                         | Équivalent SQL                         | À quoi ça sert               |
| -------------------------------- | -------------------------------------- | ---------------------------- |
| `spark.read.json()`              | `FROM JSON`                            | Lire des fichiers JSON       |
| `.select()`                      | `SELECT`                               | Choisir les colonnes         |
| `.filter()`                      | `WHERE`                                | Filtrer les lignes           |
| `.groupBy().agg()`               | `GROUP BY`                             | Agréger                      |
| `.join()`                        | `JOIN`                                 | Joindre 2 tables             |
| `.orderBy()`                     | `ORDER BY`                             | Trier                        |
| `explode()`                      | `UNNEST` / `CROSS APPLY`               | Aplatir un tableau JSON      |
| `col()`                          | référence colonne                      | Référencer une colonne       |
| `when().otherwise()`             | `CASE WHEN`                            | Condition                    |
| `cast("int")`                    | `CAST(int)`                            | Convertir le type            |
| `trim()`                         | `TRIM()`                               | Supprimer les espaces        |
| `upper()`                        | `UPPER()`                              | Majuscules                   |
| `to_date()`                      | `TO_DATE()`                            | String → Date                |
| `current_date()`                 | `CURRENT_DATE()`                       | Date du jour                 |
| `dropDuplicates()`               | `DISTINCT`                             | Supprimer les doublons       |
| `fillna(0)`                      | `COALESCE(col, 0)`                     | Remplacer NULL par 0         |
| `withColumn()`                   | `ALTER TABLE ADD`                      | Ajouter/modifier une colonne |
| `withColumnRenamed()`            | `ALTER TABLE RENAME`                   | Renommer une colonne         |
| `Window.partitionBy().orderBy()` | `OVER (PARTITION BY ... ORDER BY ...)` | Fonction fenêtre             |
| `dense_rank().over(window)`      | `DENSE_RANK() OVER (...)`              | Rang dans un groupe          |


🚀 Prochaine étape : Phase 4 — Snowflake + dbt
Dans la Phase 4, on va :
  Créer un compte Snowflake (free trial)
  Créer un warehouse et une base de données
  Installer dbt (data build tool)
  Écrire des modèles dbt qui transforment les données
  Ajouter des tests de qualité
Dis-moi quand tu es prêt pour la Phase 4 ! ❄️
"""
L'utilisateur a demandé la Phase 4. J'ai créé tous les fichiers. Maintenant je dois lui présenter le guide complet étape par étape avec les explications détaillées de chaque concept. Je vais structurer ma réponse comme pour les phases précédentes : explications des concepts, commandes PowerShell exactes, code complet avec explications ligne par ligne.
Je dois aussi créer un widget pour visualiser l'architecture dbt et peut-être un autre pour expliquer le flux de données.
Points clés à couvrir :
Qu'est-ce que Snowflake (cloud data warehouse)
Qu'est-ce que dbt (data build tool - transformation SQL avec version control)
Création compte Snowflake free trial
Récupération des credentials
Installation dbt
Configuration profiles.yml
Architecture des modèles dbt (sources → staging → marts)
Syntaxe Jinja dans dbt
Tests de qualité
Script Python pour charger PostgreSQL → Snowflake
Commandes dbt (run, test, docs)
Je vais créer un widget d'architecture et ensuite le guide 

"""































