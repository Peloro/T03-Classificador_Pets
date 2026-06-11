# Qual Desses é o meu Bebê?

### Trabalho de Descritores de Imagens - T03 - Processamento de Imagens (2026-1)

### Pedro Louro Fernandes - 13672446

Trabalho da disciplina de Processamento de Imagens. O objetivo é avaliar diferentes
**descritores de imagens** (e suas combinações) em duas tarefas semânticas: **classificação**
de pets e **busca** por similaridade.

Todo o desenvolvimento está no notebook **`pets_descritores.ipynb`**, que funciona
simultaneamente como código executável e relatório.

---

## Base de dados

- **367 imagens** de **43 pets** distintos, todas 256×256.
- Distribuição fortemente desbalanceada (de 1 a 20 fotos por pet).
- Para a **classificação**, pets com menos de 3 fotos foram removidos: **362 imagens, 38 classes**.
- Para a **busca**, todas as 367 imagens são usadas como consulta.

> Problema *fine-grained*: com 38 classes, o baseline aleatório de classificação é
> ≈ 1/38 ≈ **2,6%**.

---

## Descritores avaliados

| Descritor | Semântica capturada |
|---|---|
| **Histograma de cor HSV** (`color_hist`) | Distribuição global de cor da pelagem/plumagem |
| **Momentos de cor por célula** (`color_grid`) | Layout *espacial* da cor (grade 4×4) |
| **Local Binary Patterns** (`lbp`) | Micro-textura local da pelagem |
| **GLCM / Haralick** (`glcm`) *- não visto em aula* | Co-ocorrência espacial de tons (textura estatística) |
| **HOG** (`hog`) | Forma/silhueta via orientação de gradientes |
| **Momentos de Hu** (`hu`) | Invariantes globais de forma |

Além disso, foi implementado o **Bag of Visual Words (BoVW)** com SIFT + KMeans (K=200).

---

## Resultados

### Tarefa de Classificação (SVM RBF, split 80/10/10)

| Configuração | Acurácia (teste) |
|---|---|
| **`color_hist + color_grid`** *(melhor)* | **≈ 0,28** |
| `color_hist + lbp` | ≈ 0,26 |
| `color_hist` (melhor isolado) | ≈ 0,24 |
| `color_grid` | ≈ 0,22 |
| `hog` | ≈ 0,20 |
| todos os 6 descritores juntos | ≈ 0,20 |
| `lbp` | ≈ 0,13 |
| `hu` | ≈ 0,11 |
| `glcm` | ≈ 0,07 |

O melhor resultado (~28%) está **~11× acima do baseline aleatório** (2,6%).

### Tarefa de Busca (distância euclidiana par a par)

| Configuração | P@1 | mAP |
|---|---|---|
| **`color_hist + lbp + glcm + color_grid`** *(melhor)* | **≈ 0,35** | **≈ 0,11** |
| `color_grid` | ≈ 0,31 | ≈ 0,11 |
| `color_hist + color_grid` | ≈ 0,31 | ≈ 0,10 |
| `color_hist` | ≈ 0,26 | ≈ 0,09 |
| `lbp` | ≈ 0,15 | ≈ 0,08 |
| `hu` | ≈ 0,10 | ≈ 0,05 |

### Busca com Bag of Visual Words (SIFT, K=200)

| Método | P@1 | mAP |
|---|---|---|
| BoVW (histograma de visual words) | ≈ 0,15 | ≈ 0,075 |
| *(comparação)* melhor combinação de cor/textura | ≈ 0,35 | ≈ 0,11 |

O BoVW ficou **abaixo** dos descritores de cor - capta estrutura local de gradiente genérica
entre pets e **descarta a cor**, que é o sinal mais discriminante nesta base. As projeções
**t-SNE/UMAP** dos histogramas confirmam isso: as classes aparecem majoritariamente
embaralhadas, sem clusters limpos por pet.

---

## Principais conclusões

1. **Cor é o sinal mais forte** desta base - `color_hist` e `color_grid` lideram as duas tarefas.
2. **Combinar enxuto vence combinar tudo:** cor + textura leve (LBP/GLCM) supera juntar todos os
   6 descritores, que sofre com ruído de fundo/pose e dimensionalidade.
3. **HOG, Hu e BoVW-SIFT são prejudicados pela ausência de segmentação** - codificam o fundo e
   estrutura genérica, não a identidade do pet.
4. O **GLCM** (descritor novo) é fraco isolado, mas **agrega valor na busca** combinado com cor.
5. O **maior gargalo é o fundo não-segmentado**; o principal ganho futuro viria de segmentar o
   pet antes de extrair as características.

**Limitações:** conjuntos de validação/teste minúsculos (~46 imagens) tornam as acurácias
ruidosas, mas as *tendências relativas* entre descritores são consistentes entre classificação
e busca.

---

## Como executar

1. Descompacte `pets256.zip` e deixe a pasta `pets256/` e o `pets.csv` ao lado do notebook
   (ou ajuste `BASE` e `CSV` na Seção 1).
2. Abra `pets_descritores.ipynb` no Jupyter/VS Code e execute todas as células.

## Arquivos

- `pets_descritores.ipynb` - notebook principal (código + relatório).
- `pets_descritores.html` - render do notebook para leitura sem Jupyter.
- `codigo_modular/` - os mesmos passos em scripts `.py` separados.
