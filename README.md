# STGNN-Dengue

Previsão espaço-temporal de casos de dengue no Rio de Janeiro usando Spatio-Temporal Graph Neural Networks.

## Motivação

A dengue se espalha como um fenômeno em rede — municípios vizinhos compartilham vetor, clima e mobilidade humana. Modelos tradicionais tratam cada cidade isoladamente e ignoram essa dependência espacial.

## Objetivo

Modelar os 79 municípios do RJ como nós de um grafo, com arestas representando proximidade geográfica, e prever a taxa de internações 1 a 4 semanas à frente com um STGCN.

## Estrutura

    DengueDataset/
    ├── data/           CSVs por estado (Zenodo, Brazil-Dengue-Arboviral-Diseases)
    ├── features/       Variáveis explicativas (clima, emissões, socioeconômicas)
    ├── target/         Série alvo em formato wide
    ├── processed/      Tensor [T,N,F], splits, scaler, metadados
    ├── graphs/         Matrizes de adjacência, normalizações (Â, L̃), distâncias
    └── results/        Métricas, predições, checkpoints, figuras

    notebooks/
    ├── 00_analise_exploratoria.ipynb    EDA do estado escolhido
    ├── 01_preprocessamento.ipynb        Recorte, splits, tensor [T,N,F]
    ├── 02_construcao_grafo.ipynb        k-NN geográfico + kernel gaussiano
    ├── 03_baseline_lstm.ipynb           LSTM sem grafo (Tweedie vs. MSE log1p)
    ├── 04_stgcn.ipynb                   STGCN com 3 variantes de grafo
    └── 05_avaliacao_comparativa.ipynb   Comparação final

## Dataset

**Brazil-Dengue-Arboviral-Diseases** (Zenodo, DOI 10.5281/zenodo.18189193)

- 21 estados, séries semanais de 1999 a 2021
- 5.028 municípios no total
- Features: temperatura, precipitação, umidade, emissões CH₄/CO₂/NO₂, pobreza, urbanização, densidade
- Chave: `CD_MUN` + `Data`

Recorte atual: **Rio de Janeiro** (79 municípios, 1.184 semanas).

## Protocolo

- **Split cronológico:** treino 1999–2012, val 2013–2015, teste 2016–2021
- **Janela:** W=4 semanas → H=4 semanas
- **Target:** log1p(taxa de internações), avaliado em escala bruta
- **Loss:** MSE em log1p
- **Métrica principal:** MAE e RMSE por horizonte, com foco em semanas de surto
- **Seeds:** 42, 43, 44 (média ± desvio)

## Controles experimentais

Para isolar o efeito do grafo, o STGCN é treinado com 3 variantes:

- `geo` — grafo k-NN geográfico (k=5, kernel gaussiano)
- `ident` — A = I (sem mistura espacial)
- `rand` — grafo com nós embaralhados (mesma distribuição de graus)

Se `geo` não superar `ident` e `rand`, o ganho não vem do grafo.

## Como rodar

1. Baixe o dataset em [Zenodo](https://zenodo.org/records/18189193) e coloque em `DengueDataset/`.
2. Abra os notebooks no Colab com o Drive montado.
3. Ajuste `ESTADO` na célula de configuração (padrão `'33'` = RJ).
4. Rode na ordem `00` → `05`.

## Referências

- Siabi et al. (2026) — *STGCN para previsão de dengue na América Latina*, Scientific Reports
- Silva et al. (2026) — *Previsão de dengue em capitais brasileiras com GNNs*, SBCAS
- Gogovi (2026) — *Quando a passagem de mensagens espaciais não ajuda*, Wiley
- Yu et al. (2018) — *Spatio-Temporal Graph Convolutional Networks*, IJCAI

## Requisitos

Python 3.10+, PyTorch, NumPy, Pandas, SciPy, Matplotlib, Seaborn. Instalação:

    pip install torch numpy pandas scipy matplotlib seaborn


## Licença

Uso acadêmico — disciplina GBC073 (Inteligência Computacional, UFU).