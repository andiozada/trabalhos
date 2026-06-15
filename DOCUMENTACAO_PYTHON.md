# Documentação do Projeto Grafo---AED2

## Sobre Este Projeto

Este é um sistema de cálculo de rotas que utiliza o algoritmo de Dijkstra para encontrar o caminho mais curto entre dois pontos em um mapa. O projeto integra processamento de dados geográficos (conversão de coordenadas, parsing de arquivos OSM) com visualização gráfica interativa.

**Linguagem:** Python (100%)

**Alunos (Grupo 8):**
- Andriel Pereira e Silva
- Guilherme Oliveira
- Christian Pereira Da Silva
- Lucas Rocha Rodrigues

---

## Estrutura Geral

```
src/
├── main.py                  # Ponto de entrada - interface gráfica
├── interface.py             # GUI com PyQt6
├── djikstra_geometrico.py   # Algoritmo de Dijkstra e estruturas de grafo
├── conversor_osm.py         # Processamento de dados de mapas
├── heap.py                  # Heap mínimo (estrutura de dados)
├── config.py                # Configurações globais
└── __init__.py
```

---

## Módulos

### heap.py - Min-Heap para Dijkstra

Implementa uma fila de prioridade usando heap mínimo. Estrutura essencial para a eficiência do algoritmo de Dijkstra.

#### Classe Heap

**Atributos privados:**
- `__heap`: Lista interna que armazena os elementos

**Métodos auxiliares privados:**

- `__parente(index)` → índice do pai: `(index - 1) // 2`
- `__esq(index)` → índice do filho esquerdo: `2 * index + 1`  
- `__dir(index)` → índice do filho direito: `2 * index + 2`

**Métodos públicos:**

| Método | Retorno | O que faz |
|--------|---------|----------|
| `size()` | int | Quantidade de elementos no heap |
| `isEmpty()` | bool | Verifica se está vazio |
| `peekMin()` | tuple | Retorna o menor sem remover |
| `insert(key)` | - | Adiciona elemento e reorganiza |
| `extractMin()` | tuple | Remove e retorna o menor |
| `heapifyUp(index)` | - | Move elemento para cima mantendo ordem |
| `heapifyDown(index, tam)` | - | Move elemento para baixo mantendo ordem |

**Como funciona na prática:**

O heap mantém o **menor elemento sempre no topo** (índice 0). Quando inserimos algo novo:
1. Coloca no final
2. Compara com o pai
3. Se for menor, troca e repete com o novo pai

Na extração:
1. Guarda o elemento do topo (o mínimo)
2. Move o último para o topo
3. O deixa "cair" até o lugar certo

Lança `RuntimeError` se tentar acessar heap vazio.

---

### djikstra_geometrico.py - Algoritmo e Estruturas

Contém o núcleo do sistema: o algoritmo de Dijkstra, estruturas de vértices/arestas e funções para manipular grafos.

#### Classes

**Vertice**
```python
Vertice(v_id: int, x: float, y: float)
```
Representa um ponto no mapa com ID único e coordenadas (x, y) em pixels.

**Aresta**
```python
Aresta(orig: int, dest: int, tipo: int)
```
Conexão entre dois vértices.
- `tipo=1`: mão única (sentido único)
- `tipo=2`: mão dupla (ambos os sentidos)

#### Funções

**calc_dist(x0, y0, x1, y1) → float**

Distância euclidiana entre dois pontos: `√((x0-x1)² + (y0-y1)²)`

Retorna um valor em pixels (será convertido para metros depois se necessário).

---

**construir_grafo(vertices, arestas) → list**

Monta a lista de adjacência do grafo.

Para cada aresta:
1. Calcula distância entre os dois vértices
2. Adiciona à lista do vértice de origem
3. Se mão dupla, também adiciona a volta

Retorna: lista onde `grafo[i]` contém tuplas `(distância, id_vizinho)`

Exemplo: Se há uma aresta de 0→1 com distância 10 e tipo 2:
```
grafo[0] = [..., (10.0, 1)]
grafo[1] = [..., (10.0, 0)]
```

---

**dijkstra(grafo, vertices, inicio, fim) → dict ou None**

O algoritmo em si. Retorna o caminho mais curto e estatísticas.

**Processo:**

1. Inicializa distâncias como "infinito" (1e9), menos a origem (0)
2. Cria heap com o vértice inicial
3. Enquanto há vértices no heap:
   - Remove o com menor distância
   - Para cada vizinho, tenta atualizar a distância
   - Se achar caminho mais curto, adiciona vizinho ao heap
4. Reconstrói o caminho andando para trás do destino até origem
5. Calcula estatísticas e formata resultado

**Retorno - dicionário com:**
```python
{
    "caminho_ids": [0, 2, 5, 7],          # Sequência de vértices
    "distancia_pixels": 45.3,              # Em pixels
    "distancia_metros": 90.6,              # Pixels * FATOR_ESCALA
    "nos_explorados": 12,                  # Quantos vértices visitou
    "tempo_ms": 2.45                       # Tempo de execução
}
```

Se não houver caminho: retorna `None` e imprime mensagem no console.

**Imprime no console:**
- Tempo total de processamento
- Quantidade de nós visitados
- Distâncias em pixels e metros

---

**carregar_mapa_poly(caminho_arquivo) → (list, list)**

Lê arquivo `.poly` e reconstrói vértices e arestas.

**Formato esperado do arquivo:**
```
5 2 0 1                    # total_verts, colunas, linhas, camadas
0 0.0 0.0                  # id, x, y (vértices)
1 10.5 5.2
...
4 1                        # total_arestas, tipo
0 0 1 2                    # id, origem, destino, tipo (arestas)
1 1 2 2
...
0                          # marcador de fim
```

Retorna tupla: `(lista_de_Vertice, lista_de_Aresta)`

---

### conversor_osm.py - Processamento de Mapas

Converte dados brutos do OpenStreetMap para um formato interno processável.

#### Classes

**Node**

Nó do arquivo OSM antes de ser convertido.
```python
Node(id_original, lat, lon, id_interno)
```
Armazena coordenadas em latitude/longitude e posteriormente em UTM (x, y).

**Way**

Sequência de nós que forma uma rua.
```python
Way()
```
- `node_ids`: lista de IDs dos nós que formam a rua
- `is_oneway`: booleano para rua de mão única

#### Funções

**converter_para_utm(lat_deg, lon_deg) → (float, float)**

Converte GPS (latitude/longitude do WGS84) para UTM (sistema local em metros).

Usa fórmulas de projeção com parâmetros:
- Raio terrestre (WGS84): 6,378,137 m
- Meridiano central para Brasil: -45°
- Fator de escala: 0.9996

Retorna tupla `(x_utm, y_utm)` em metros.

**Não precisa saber:** As fórmulas complexas implementadas aqui. O importante é que transforma "latitude/longitude" em "metros locais".

---

**reduzir_escala(pontos, redutor)**

Normaliza e redimensiona coordenadas para caber na tela.

Operação:
1. Encontra o mínimo x e y dos pontos
2. Subtrai esses valores (move tudo para origem 0,0)
3. Divide por `redutor` (escala para tamanho razoável)

Modificação **in-place**: altera os objetos diretamente.

---

**parse_osm(filename) → str**

Função principal do módulo. Lê arquivo OSM e gera `.poly`.

**O que realmente faz:**

1. **Lê arquivo XML** do OpenStreetMap
2. **Extrai nós e vias** (roads/highways)
3. **Filtra vias válidas** - apenas tipos "highway" reconhecidos (motorway, residential, etc.)
4. **Trata mão única** - se `oneway=-1`, inverte direção
5. **Converte coordenadas** WGS84 → UTM
6. **Redimensiona** para caber na tela
7. **Inverte eixo Y** - porque gráficos usam Y invertido
8. **Salva resultado** em arquivo `.poly`

**Importante:** Não mantém TODOS os nós do OSM, apenas os que são parte de vias válidas. Isso reduz dados significativamente.

Retorna caminho do arquivo gerado ou string vazia em caso de erro.

---

### main.py - Interface Gráfica

Ponto de entrada da aplicação.

```python
def main():
    app = QApplication(sys.argv)      # Cria aplicação Qt
    janela = Janela()                 # Cria janela principal
    janela.show()                     # Mostra na tela
    sys.exit(app.exec())              # Loop de eventos
```

A interface de CLI foi **removida** em favor da GUI com PyQt6.

---

### interface.py - GUI com PyQt6

Interface gráfica completa para visualização e interação com o mapa.

#### Classe GrafoItem

Renderiza o grafo na tela (herda de `QGraphicsItem`).

**Atributos:**
- `janela`: referência à janela principal
- `_bounding_rect`: retângulo que engloba todo o grafo
- `linhas_unica`: linhas das arestas de mão única
- `linhas_dupla`: linhas das arestas de mão dupla
- `path_setas_unica`: setas desenhadas nas arestas

**Métodos:**

`atualizar_geometria()`

Recalcula geometria quando mapa é carregado. Encontra limites, cria linhas e setas. Chamado sempre que mapa muda.

`boundingRect() → QRectF`

Retorna a caixa que engloba todo o grafo.

`paint(painter, option, widget)`

Desenha na tela:
- Arestas de mão dupla (azul)
- Arestas de mão única com setas (verde)
- Pesos das arestas (se ativado)
- Rota encontrada (laranja)
- Vértices com cores: verde (origem), vermelho (destino), amarelo (temp), cinza (normal)

---

#### Classe VisualizadorMapa

Visualizador interativo do mapa (herda de `QGraphicsView`).

**Interações:**
- **Roda do mouse**: zoom in/out (±20%)
- **Clique direito**: executa ação conforme modo selecionado

Detecta vértices próximos ao clique (raio de tolerância ajustado por zoom).

---

#### Classe Janela

Janela principal da aplicação.

**Atributos principais:**
- `origem_selecionada`, `destino_selecionado`: IDs dos vértices
- `caminho_resultado`: lista de vértices da rota
- `vertices`, `arestas`: dados do grafo
- `grafo`: lista de adjacência

**Métodos importantes:**

`importar_mapa()`

Abre diálogo para escolher arquivo `.poly` e carrega.

`converter_usar_osm()`

Abre diálogo para arquivo `.osm`, converte via `parse_osm()` e carrega resultado.

`calcular_caminho()`

Executa Dijkstra com origem/destino selecionados. Mostra estatísticas na interface.

`tratar_clique_mapa(x, y)`

Processa cliques direitos conforme modo:
1. **Buscar Caminho**: seleciona origem (1º clique) e destino (2º clique)
2. **Adicionar Vértice**: cria novo ponto no local
3. **Remover Vértice**: deleta ponto mais próximo
4. **Adicionar Aresta (Mão Única/Dupla)**: conecta dois pontos
5. **Remover Aresta**: desconecta dois pontos

`refatorar_indices()`

Quando um vértice é removido, renumera todos os IDs sequencialmente (0, 1, 2...). Atualiza arestas para novos IDs.

`atualizar_interface_status()`

Atualiza rótulos mostrando origem, destino e estado atual.

`atualizar_estrutura_grafo()`

Reconstrói grafo quando vértices/arestas mudam.

`limpar_selecoes()`

Reseta origem, destino, caminho e estatísticas.

`copiar_imagem_grafo()`

Renderiza grafo como imagem e copia para área de transferência.

---

### config.py - Configurações

```python
FATOR_ESCALA = 2.0
```

Multiplier para converter distâncias de pixels para metros.

Se `FATOR_ESCALA = 2.0` e a distância calculada é 50 pixels, então a distância real é 100 metros.

---

## Como Usar

### Via Interface Gráfica (Recomendado)

```bash
cd src
python main.py
```

**Workflow:**

1. Clique em **"Importar Mapa (.poly)"** ou **"Converter e Usar Arquivo .osm"**
2. Selecione arquivo
3. Escolha modo no combo box (padrão: "Buscar Caminho")
4. Clique direito no mapa 2 vezes para marcar origem (verde) e destino (vermelho)
5. Clique **"Calcular Menor Caminho"**
6. Rota aparece em laranja

**Checkboxes úteis:**
- "Mostrar IDs dos Vértices" - mostra número de cada ponto
- "Mostrar Pesos das Arestas" - distância em metros
- "Ocultar Elementos Fora da Rota" - mostra apenas a rota
- "Ocultar Apenas Vértices Não Selecionados" - modo focado

**Controles:**
- Roda do mouse: zoom
- Arrastar: mover visualização
- Copiar Grafo: copia imagem para clipboard

---

## Fluxo de Dados

```
Arquivo OSM (XML)
    ↓
parse_osm() → converter_para_utm() + reduzir_escala()
    ↓
Arquivo .poly
    ↓
carregar_mapa_poly()
    ↓
Vertice[] + Aresta[]
    ↓
construir_grafo()
    ↓
Lista de Adjacência
    ↓
dijkstra()
    ↓
Caminho + Estatísticas
    ↓
GrafoItem.paint() → Visualização
```

---

## Detalhes Técnicos

### Algoritmo de Dijkstra

Greedy algorithm que encontra caminho mais curto em grafo com pesos positivos.

**Complexidade:** O((V + E) log V) com heap

Onde V = vértices, E = arestas

### Conversão de Coordenadas

Os dados do OSM vêm em latitude/longitude (WGS84). Para visualizar em 2D na tela, usamos:

1. **Projeção UTM** (Universal Transverse Mercator) - converte para metros locais
2. **Redimensionamento** - divide por fator para caber em pixels
3. **Inversão de Y** - sistemas gráficos têm Y invertido

### Estrutura de Dados

- **Heap mínimo**: O(log n) para insert/extract → otimiza Dijkstra
- **Lista de adjacência**: O(1) para acessar vizinhos
- **Dicionário (mapa)**: O(1) para encontrar vértice por ID

---

## Possíveis Melhorias

- [ ] Implementar A* (mais rápido que Dijkstra para navegação)
- [ ] Salvar/carregar grafos em formato binário (mais rápido)
- [ ] Suporte a múltiplas rotas alternativas
- [ ] Calcular distância de ruas a pé vs. carro
- [ ] Interface 3D (altura das ruas)
- [ ] Busca por nome de local

---

## Notas de Desenvolvimento

**Por que Dijkstra e não Bellman-Ford?**
- Dijkstra é mais rápido (O(V log V) vs. O(VE))
- Funciona com pesos positivos (distâncias sempre são)

**Por que heap mínimo?**
- Sem heap: O(V²)
- Com heap: O((V+E) log V)
- Grande diferença em grafos grandes

**Por que converter OSM?**
- OSM tem dados desnecessários
- Filtrar apenas highways reduz tamanho
- Converter para UTM deixa cálculos em metros

---

**Documentação atualizada:** Junho 2026
