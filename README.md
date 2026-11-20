# AI Memory Booster

# AI Memory Booster — Édition ARM64 (Ubuntu 26.04 LTS / Kernel 7)

## 1. À quoi sert ce dépôt ?

**AI Memory Booster** est un module open-source léger, autonome et embarquable permettant de doter un modèle de langage local (LLM via **Ollama**) d'une **mémoire persistante à long terme**.

Il combine :
- **ChromaDB** comme base de données vectorielle locale (stockage des embeddings et recherche par similarité sémantique).
- **Ollama** pour l'inférence LLM et la génération d'embeddings sans dépendance cloud.
- **Un moteur Node.js + API REST + Interface Web** gérant l'injection (`storeMemory`), la récupération (`retrieveMemory`), la consolidation/résumé automatique des conversations, l'oubli ciblé (`forget`) et le chiffrement des données persistantes.

---

## 2. Dans quelles circonstances utiliser le binaire / service ?

Le binaire wrapper `/usr/bin/ai-memory-booster` (et son service `systemd` `ai-memory-booster.service`) est particulièrement pertinent dans les scénarios suivants :

- **Assistant personnel ou métier 100 % hors-ligne (air-gapped / souverain)** : aucune donnée ne quitte la carte SBC ; les échanges et souvenirs vectorisés restent chiffrés localement.
- **Déploiement sur SBC ARM64 à faible consommation** : fonctionne en continu (24/7) sur une **Orange Pi 5 (RK3588, 32 Go RAM)** ou une **Orange Pi 6 Plus (CIX CD8180)** en serveur de mémoire dédié sur le réseau local.
- **Couche mémoire découplée via API REST (`http://127.0.0.1:4000`)** : permet à plusieurs clients hétérogènes (scripts Python, bots CLI, interfaces web, agents locaux) de partager la même mémoire long terme sans réimplémenter la logique RAG.
- **Prototypage rapide** : remplace une stack RAG lourde lorsque le besoin se concentre sur la mémoire conversationnelle persistante et l'apprentissage continu (`learnFromChat: true`).

---

## 3. Pré-requis pour la cross-compilation (Ubuntu 26.04 LTS `x86_64` → `aarch64`)

Cibles matérielles visées : **Rockchip RK3588** (ARMv8.2-A) et **CIX CD8180 / P1** (ARMv9.2-A) sous **Ubuntu 26.04 LTS `aarch64` (Kernel 7)**.

### 3.1 Paquets nécessaires (obligatoires)

| Catégorie | Paquets APT (sur l'hôte `x86_64`) | Rôle |
|---|---|---|
| Chaîne de compilation | `gcc-aarch64-linux-gnu`, `g++-aarch64-linux-gnu`, `binutils-aarch64-linux-gnu`, `cpp-aarch64-linux-gnu` | Compilateurs C/C++ et éditeur de liens ciblant `aarch64` |
| Outils de build | `build-essential`, `make`, `cmake`, `ninja-build`, `pkg-config`, `autoconf`, `automake`, `libtool` | Orchestration des builds C/C++ et addons natifs |
| Packaging Debian | `dpkg-dev`, `debhelper`, `fakeroot`, `devscripts`, `dh-make` | Création, signature et vérification des paquets `.deb` |
| Librairies cibles (`arm64`) | `libc6-dev:arm64`, `libstdc++-14-dev:arm64`, `libssl-dev:arm64`, `libsqlite3-dev:arm64`, `zlib1g-dev:arm64` | Headers et bibliothèques de linkage pour `aarch64` |
| Runtime Node / Python | `nodejs`, `npm`, `python3`, `python3-venv`, `python3-dev` | Build des dépendances JS (`node-gyp`) et scripts |

### 3.2 Paquets utiles (fortement recommandés)

| Paquets APT | Rôle |
|---|---|
| `qemu-user-static`, `binfmt-support` | Exécuter de manière transparente des binaires `aarch64` sur l'hôte `x86_64` (tests unitaires Jest, `npm`, vérification post-build) |
| `ccache` | Accélère drastiquement les recompilations C/C++ successives |
| `crossbuild-essential-arm64` | Méta-paquet regroupant les outils de cross-build Debian/Ubuntu pour `arm64` |
| `lintian` | Audit de conformité des paquets `.deb` générés |
| `patchelf`, `file`, `readelf` (`binutils`) | Vérification de l'architecture ELF (`ARM aarch64`) et des `RPATH`/dépendances dynamiques |

### 3.3 Paquets optionnels (avancés / accélération matérielle)

| Paquets / Outils | Rôle |
|---|---|
| `clang`, `lld`, `llvm` | Alternative à GCC avec meilleur support natif ARMv9.2-A / SVE2 (CIX CD8180) |
| `sbuild`, `mmdebstrap`, `schroot` | Construction reproductible en chroot propre `ubuntu-26.04-arm64` |
| `rustc`, `cargo` (+ cible `aarch64-unknown-linux-gnu`) | Requis uniquement si vous recompilez depuis les sources les bindings Rust de ChromaDB (`tokenizers`, `chroma-hnswlib`) |
| `libvulkan-dev:arm64`, `glslang-tools` | Utile pour cross-compiler `llama.cpp` / backend Vulkan sur Mali-G610 (RK3588) ou Immortalis-G720 (CD8180) |

---

## 4. Installation de l'environnement de cross-compilation (`x86_64` → `aarch64`)

### Étape 1 : Activer l'architecture `arm64` (Multiarch)

```bash
sudo dpkg --add-architecture arm64


**AI Memory Booster** is an AI-powered memory enhancement module that enables **long-term AI memory** and retrieval of conversations using embeddings and vector databases.

**Online demo** [https://aimemorybooster.com](https://aimemorybooster.com)

## 📚 Table of Contents

- [🚀 Features](#-features)
- [📦 Installation](#-installation)
  - [🔤 Quick Setup for MacOS/Linux Users](#-quick-setup-for-macoslinux-users)
  - [Manual Install Instruction](#manual-install-instruction)
    - [Pre-requisites](#pre-requisites)
    - [Install Ollama](#install-ollama)
    - [Install ChromaDB](#install-chromadb)
    - [Install AI Memory Booster via npm](#install-ai-memory-booster-via-npm)
- [🎯 Usage](#-usage)
- [🔧 Configuration](#-configuration)
  - [Example config.json](#example-configjson)
  - [ConfigManager Usage](#configmanager-usage)
- [🔧 Development](#-development)
- [📜 License](#-license)
- [📩 Contact](#-contact)

## 🚀 Features

- **Fully deployable on-premise**: Run locally to maintain full control, meet strict compliance, and ensure enhanced data privacy.
- **Self-learning capability**: Continuously adapts to user environments for more personalized performance over time.
- **Permanent long-term memory**: Retains information across sessions and system restarts.
- **Offline functionality**: Operates without internet access, improving performance and reducing external dependencies.
- **LLM-agnostic**: Integrates with any LLM based on your infrastructure or budget.
- **Simple to manage**: Configure easily using a JSON file or via the built-in admin portal.
- **Seamless integration**: Provides an NPM package, REST API, and a web interface for plug-and-play deployment.
- **Fully user-owned**: Avoids reliance on third-party AI services, lowering costs and boosting independence.

## 📦 Installation

### 🔤 Quick Setup for MacOS/Linux Users

For MacOS or Linux users, we provide a script to automate the setup process:

```sh
./install.sh
```

This script will:
- Install dependencies
- Setup environment variables (`.env.local`)
- Launch required services (e.g., ChromaDB)

Make sure the script has executable permission:
```sh
chmod +x install.sh
```

### Manual Install Instruction
#### Pre-requisites
Before running AI Memory Booster, ensure that you have **Ollama** and **ChromaDB** installed.

##### Install Ollama
Follow the official installation guide at: [https://ollama.ai/](https://ollama.ai/)

##### Install ChromaDB
Ensure ChromaDB is installed and running:
```sh
pip install chromadb
chroma run --path ./chroma_db
```

#### Install AI Memory Booster via npm:
```sh
npm install ai-memory-booster
```

## 🎯 Usage

AI Memory Booster provides the following methods:

### Store Memory
Injects a memory entry into AI Memory Booster.
```js
await AI_Memory.storeMemory("Today is Wednesday and it is raining", "Today is Wednesday, how's the weather?", "It is raining");
```

### Retrieve Memory
Fetches relevant stored memories based on a user's message.
```js
const memory = await AI_Memory.retrieveMemory("How's the weather today?");
console.log(memory);
```
**Response Example:**
```js
{
  "id": "99c326e8-e234-4049-8661-3d9427944071",
  "distance": 0.1, // The distance between the query and result (smaller is better)
  "summary": "Today is Wednesday and it is raining",
  "userMessage": "Today is Wednesday, how's the weather?",
  "aiMessage": "It is raining",
  "timestamp": 1740999657717
}
```

### Forget Memory
Deletes a memory entry by ID.
```js
const success = await AI_Memory.forget("99c326e8-e234-4049-8661-3d9427944071");
console.log(success); // true if deleted, false if ID is null
```

### Forget All Memories
Deletes all stored memory entries.
```js
const success = await AI_Memory.forgetAll();
console.log(success); // true if deleted, false if no memories exist
```

### Get LLM Specification
Retrieves details about the current LLM module.
```js
const llmSpec = await AI_Memory.getLlmSpec();
console.log(llmSpec);
```

### Chat with AI Memory Booster
Interacts with the AI while using stored memory (For chat).
```js
const response = await AI_Memory.chat("How's the weather today?");
console.log(response); // "It is raining today!"
```

### Generate content with AI Memory Booster
Interacts with the AI while using stored memory (For generating text content).
```js
const response = await AI_Memory.generate("How's the weather today?");
console.log(response); // "It is raining."
```

## 🔧 Configuration

AI Memory Booster allows full customization through `config.json`. Users can modify AI model settings, server configurations, memory management, and ChromaDB parameters. The configuration can be updated in two ways:

- **Directly modifying `config.json`**
- **Using AI Memory Booster UI for easy configuration updates**

### Example config.json:

```js
{
  "aiModel": "llama3.2", // The primary AI model (must be downloaded first)
  "learnFromChat": true, // Determines if AI should learn from conversations
  "host": "localhost", // The host address when running AI Memory Booster as an API service
  "port": 4000, // The port for the AI Memory Booster API service
  "baseKeepAlive": 3000, // Duration (ms) the LLM module stays active after each call
  "extendedKeepAlive": 10000, // Extended duration (ms) the LLM remains active if no other requests arrive
  "similarityResultCount": 3, // Number of similar records retrieved from the database
  "categorySureThreshold": 49, // Threshold for AI to confidently classify a response
  "maxUserMessageCharacterLimit": 10000, //Max user message length allowed
  "summaryCharacterLimit": 256, // Maximum character length for conversation summaries
  "dimension": 768, // Dimensionality of vector embeddings
  "similarityThreshold": 0.7, // Threshold for similarity-based searches
  "consolidateConversationThreshold": 256, // Threshold for summarizing conversations
  "chromaDBHost": "http://localhost:8000", // ChromaDB service URL (host and port)
  "tenant": "default_tenant", // ChromaDB tenant name
  "collection": "ai_memory_booster", // ChromaDB collection name used by AI Memory Booster
  "rolePrompt": "You are a personal assistant. 'AI' represents you, and 'User' represents the person currently talking to you. When the user says 'I', 'mine', 'me' or 'my', it refers to the user, not you ('AI'). Do not fabricate responses.", // Prompt for how AI should respond based on past memory
  "debug": false, // If turn on the debug message
  "archive": false, // If turn on the memory archive mode
}
```

### ConfigManager Usage:

Users can dynamically get and set configurations via `configManager`:

```js
const port = AI_Memory.configManager.getPort();
console.log(`Running on port: ${port}`);

AI_Memory.configManager.setPort(8080);
console.log(`Updated port: ${AI_Memory.configManager.getPort()}`);
```

## 🔧 Development

Clone and install dependencies:
```sh
git clone https://github.com/aotol/ai-memory-booster.git
cd ai-memory-booster
npm install
```

To start AI Memory Booster as a standalone API service:
```sh
npx ai-memory-booster start
```

To launch AI Memory Booster UI, go to:
```sh
cd ai-memory-booster/web/ai-memory-ui/
npm run dev
```
Make sure you have set the environment variable: `AI_MEMORY_BOOSTER_API_URL` to identify where AI Memory Booster API service is running.

e.g.:
```sh
export AI_MEMORY_BOOSTER_API_URL="http://localhost:4000"
```
Alternatively, create a ```.env.local``` file under ```ai-memory-ui``` directory and set the content to:
```
AI_MEMORY_BOOSTER_API_URL=http://localhost:4000
```

## 📜 License

**MIT License** - AI Memory Booster versions **1.x.x** are free and open-source under the MIT License.

> **Note:** Starting from version **2.0.0**, AI Memory Booster will be available as a **commercial version** with multi-tenant (user) support and advanced features. For commercial licensing and inquiries about **2.0.0** and above, please contact **zhan@aotol.com**.

## 📩 Contact

Author: **Aotol Pty Ltd**\
Email: **[zhan@aotol.com](mailto:zhan@aotol.com)**\
Website: **[https://github.com/aotol/ai-memory-booster](https://github.com/aotol/ai-memory-booster)**

---

🚀 **Start building with AI Memory Booster today!**
