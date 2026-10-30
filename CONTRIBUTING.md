# 📊 ENEM Data Exploration

Projeto de análise dos microdados do ENEM (2023–2024), com pipeline automatizado via **DVC**, controle de versão em **Git** e experimentos em **Jupyter Notebooks**.

---

## 🧱 Estrutura do Projeto

```
enem-data-exploration/
├── data/
│   ├── raw/                ← dados originais (.zip)
│   ├── interim/            ← dados intermediários (unzip, parquet)
│   └── processed/          ← datasets prontos p/ análise
├── notebooks/
│   ├── 01_preprocessamento_enem_2023.ipynb
│   ├── 02_bivariada_e_figuras.ipynb
│   └── 03_rotulos_e_paletas.ipynb
├── src/
│   ├── prepare_enem.py     ← transforma CSV em Parquet
│   └── read_enem_dictionary.py (opcional)
├── reports/
│   ├── figures/            ← gráficos gerados
│   └── tabelas/            ← tabelas exportadas
├── .dvc/                   ← metadados do DVC
├── dvc.yaml                ← definição do pipeline
├── requirements.txt
└── README.md
```

---

## ⚙️ Configuração do Ambiente

### 1) Clonar o repositório
```bash
git clone https://github.com/fontineli/enem-data-exploration.git
cd enem-data-exploration
```

### 2) Criar ambiente virtual e instalar dependências
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

> Se não existir, crie um `requirements.txt` com:
> ```
> pandas
> pyarrow
> dvc[gdrive]
> matplotlib
> jupyterlab
> ```

---

## 💾 DVC + Google Drive

O DVC versiona e sincroniza os datasets (ZIPs, Parquet) num **Google Drive** compartilhado.

### 3) Configurar credenciais **locais** (não versionadas)
> **Nunca** commite credenciais. Elas ficam somente em `.dvc/config.local`.

**OAuth pessoal:**
```bash
dvc remote modify storage --local gdrive_client_id "<SEU_CLIENT_ID>"
dvc remote modify storage --local gdrive_client_secret "<SEU_CLIENT_SECRET>"
```

**OU Service Account (recomendado p/ times):**
```bash
dvc remote modify storage --local gdrive_service_account_json ~/Downloads/service-account.json
```

### 4) Baixar os dados
```bash
dvc pull
```

### 5) Reproduzir o pipeline
```bash
dvc repro
```

Isso executa as *stages* (unzip → preparo → Parquet) definidas no `dvc.yaml`.

---

## 📓 Notebooks

Abra os notebooks com o venv ativo:
```bash
source .venv/bin/activate
jupyter lab
```

Notebooks principais:
- `01_preprocessamento_enem_2023.ipynb` — checagens, missing, outliers, overview das notas.
- `02_bivariada_e_figuras.ipynb` — gráficos prontos (matplotlib) por tipo de escola, renda, UF, correlações.
- `03_rotulos_e_paletas.ipynb` — aplica rótulos do dicionário ENEM (xlsx) e exporta `enem_2023_rotulado.parquet`.

> Se estiver abrindo os notebooks de dentro de `notebooks/`, os caminhos para dados costumam começar com `../data/...`

---

## 🔒 Boas Práticas (Git + DVC)

- **NÃO commitar dados** (`data/raw/`, `data/interim/`, `.zip`, `.parquet`, `.csv`).  
- Use **DVC** para baixar/subir dados: `dvc pull` / `dvc push`.
- Commits devem incluir:
  - scripts em `src/`,
  - notebooks em `notebooks/`,
  - `dvc.yaml`, `dvc.lock`, `.gitignore`,
  - sem credenciais.

### Exemplo de fluxo
```bash
# altera script ou stage
dvc repro
dvc push

git add src/ dvc.yaml dvc.lock notebooks/
git commit -m "ajusta preparo e gera novas figuras"
git push
```

---

## 🧯 Segurança de Credenciais

- Deixe **apenas** configurações públicas em `.dvc/config` (ex.: `url` do remote).
- Coloque credenciais **somente** em `.dvc/config.local` (já ignorado no `.gitignore`).
- Se um commit vazar credenciais por engano:
  ```bash
  # remove credenciais do arquivo e emenda o commit
  git add .dvc/config
  git commit --amend --no-edit
  git push --force-with-lease
  ```
  Em seguida, **rotacione** o secret no Google Cloud.

---

## 🧩 Troubleshooting Rápido

- **`ModuleNotFoundError: pandas`**
  → ative o venv e `pip install -r requirements.txt`.

- **DVC – `403 quotaExceeded` com Service Account**
  → compartilhe a pasta do Drive com a Service Account **ou** use OAuth pessoal.

- **Problemas com unzip no macOS (acentos)**
  → usar `ditto` com locale:
  ```bash
  LANG=pt_BR.UTF-8 LC_ALL=pt_BR.UTF-8 ditto -x -k data/raw/microdados_enem_2023.zip data/interim/unzipped_2023
  ```

- **Notebook não acha `data/interim/...`**
  → ajuste caminho relativo: use `../data/...` se estiver dentro de `notebooks/`, ou:
  ```python
  from pathlib import Path
  PARQUET_PATH = Path("../data/interim/enem_2023.parquet").resolve()
  ```

---

## 👥 Onboarding de Colaboradores

1. Clonar o repo e criar venv.  
2. `pip install -r requirements.txt`  
3. Configurar `.dvc/config.local` (OAuth ou Service Account).  
4. `dvc pull` para obter os dados.  
5. `dvc repro` (se quiser refazer o pipeline).  
6. Abrir notebooks no Jupyter Lab.

