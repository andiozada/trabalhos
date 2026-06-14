# 📚 Documentação do Sistema de Navegação com Dijkstra - Python

**Repositório Original:** [Grafo---AED2](https://github.com/christianps-dev/Grafo---AED2)

---

## 📖 Índice

1. [Visão Geral](#visão-geral)
2. [Módulos e Classes](#módulos-e-classes)
3. [Funções Principais](#funções-principais)
4. [Como Usar](#como-usar)

---

## 🎯 Visão Geral

Este projeto implementa um **Sistema de Navegação Integrado** que utiliza o **Algoritmo de Dijkstra** para encontrar o menor caminho entre dois pontos em um mapa. O sistema é capaz de:

- **Importar mapas** em formato OSM (OpenStreetMap)
- **Converter coordenadas** de WGS84 para UTM
- **Construir grafos** com lista de adjacência
- **Calcular rotas** usando o Algoritmo de Dijkstra com Heap Min
- **Visualizar resultados** através de interface gráfica com PyQt6

---

## 📦 Módulos e Classes

### 1️⃣ **heap.py** - Estrutura de Dados: Heap Mínimo

#### 📌 Classe: `Heap`

Uma implementação de **Heap Mínimo** (Min-Heap) usada para otimizar o Algoritmo de Dijkstra.

##### 🔧 **Métodos Privados**

```python
def __parente(self, index: int) -> int
```
**O que faz:** Calcula o índice do nó pai em um heap binário.

**Explicação fácil:** Imagine uma árvore invertida onde cada nó tem no máximo 2 filhos. Este método encontra quem é o "pai" de um elemento.

**Fórmula:** `(index - 1) // 2`

---

```python
def __esq(self, index: int) -> int
```
**O que faz:** Calcula o índice do filho esquerdo de um nó.

**Explicação fácil:** Em um heap, cada elemento pode ter um filho à esquerda. Este método encontra onde ele fica.

**Fórmula:** `(2 * index + 1)`

---

```python
def __dir(self, index: int) -> int
```
**O que faz:** Calcula o índice do filho direito de um nó.

**Explicação fácil:** Assim como o esquerdo, cada nó pode ter um filho à direita.

**Fórmula:** `(2 * index + 2)`

---

##### 🔧 **Métodos Públicos**

```python
def size(self) -> int
```
**O que faz:** Retorna a quantidade de elementos no heap.

**Explicação fácil:** Conta quantos elementos estão guardados no heap.

---

```python
def isEmpty(self) -> bool
```
**O que faz:** Verifica se o heap está vazio.

**Explicação fácil:** Retorna `True` se não há elementos, `False` caso contrário.

---

```python
def heapifyUp(self, index: int)
```
**O que faz:** Move um elemento para cima na árvore até manter a propriedade do heap.

**Explicação fácil:** Se um elemento é menor que seu pai, ele "sobe" de lugar. Continua subindo até encontrar seu lugar correto. Isto garante que o menor elemento sempre fica no topo.

**Processo:**
1. Compara o elemento com seu pai
2. Se for menor, troca de lugar
3. Repete com o novo pai
4. Para quando não conseguir mais subir

---

```python
def heapifyDown(self, index: int, tam: int)
```
**O que faz:** Move um elemento para baixo na árvore, mantendo a propriedade do heap.

**Explicação fácil:** Se um elemento é maior que seus filhos, ele "desce" de lugar. O elemento vai para baixo até encontrar seu lugar correto.

**Processo:**
1. Compara o elemento com seus filhos
2. Identifica o menor dos filhos
3. Se o elemento for maior, troca com o menor filho
4. Repete com a nova posição
5. Para quando não conseguir mais descer

---

```python
def peekMin(self) -> tuple
```
**O que faz:** Retorna o menor elemento sem removê-lo.

**Explicação fácil:** Olha qual é o elemento mais pequeno, mas não tira ele do heap.

**Exceção:** Lança erro se o heap está vazio.

---

```python
def insert(self, key: tuple)
```
**O que faz:** Adiciona um novo elemento ao heap.

**Explicação fácil:** 
1. Coloca o elemento no final da lista
2. Chama `heapifyUp` para ele subir até o lugar certo

---

```python
def extractMin(self) -> tuple
```
**O que faz:** Remove e retorna o menor elemento do heap.

**Explicação fácil:**
1. Guarda o elemento do topo (o menor)
2. Move o último elemento para o topo
3. Chama `heapifyDown` para reorganizar
4. Devolve o menor elemento que foi guardado

**Exceção:** Lança erro se o heap está vazio.

---

---

### 2️⃣ **djikstra_geometrico.py** - Algoritmo de Dijkstra

#### 📌 Classe: `Vertice`

Representa um ponto no mapa.

```python
class Vertice:
    def __init__(self, v_id: int, x: float, y: float)
```

**Atributos:**
- `id`: Identificador único do vértice
- `x`: Coordenada X (em pixels após escala)
- `y`: Coordenada Y (em pixels após escala)

**Explicação fácil:** Um vértice é um ponto no mapa, como uma encruzilhada em uma cidade. Cada um tem um ID único e uma posição (x, y).

---

#### 📌 Classe: `Aresta`

Representa uma conexão entre dois vértices.

```python
class Aresta:
    def __init__(self, orig: int, dest: int, tipo: int)
```

**Atributos:**
- `orig`: ID do vértice de origem
- `dest`: ID do vértice de destino
- `tipo`: 
  - `1` = Mão única (só vai de origem para destino)
  - `2` = Mão dupla (vai nos dois sentidos)

**Explicação fácil:** Uma aresta é uma rua que conecta dois pontos. Pode ser de mão única ou dupla.

---

#### 🔧 **Funções Globais**

```python
def calc_dist(x0: float, y0: float, x1: float, y1: float) -> float
```

**O que faz:** Calcula a distância euclidiana entre dois pontos.

**Explicação fácil:** Usa a fórmula da distância para saber quantos "passos" tem entre dois pontos em um plano.

**Fórmula:** `√((x0-x1)² + (y0-y1)²)`

**Exemplo:** Distância entre (0,0) e (3,4) é 5.

---

```python
def construir_grafo(vertices: list, arestas: list) -> list
```

**O que faz:** Constrói uma **Lista de Adjacência** a partir de vértices e arestas.

**Explicação fácil:** 
Cria uma estrutura que diz: "Do ponto A, você pode ir para B (distância 10) e C (distância 5)". Isso facilita encontrar os vizinhos de cada ponto.

**Processo:**
1. Cria uma lista vazia para cada vértice
2. Para cada aresta:
   - Calcula a distância entre origem e destino
   - Adiciona na lista do vértice de origem
   - Se for mão dupla, também adiciona a volta (destino → origem)

**Retorna:** Lista onde cada posição i contém uma lista de tuplas (distância, vizinho)

---

```python
def dijkstra(grafo: list, vertices: list, inicio: int, fim: int) -> dict
```

**O que faz:** Encontra o menor caminho entre dois vértices usando o Algoritmo de Dijkstra.

**Explicação fácil:**
Imagina que você está em uma cidade e quer ir do ponto A ao ponto B pelo caminho mais curto. O Dijkstra marca cada ponto com a menor distância encontrada até agora. Começa pelo início, e vai explorando pontos cada vez mais próximos do destino.

**Processo:**
1. Inicializa todas as distâncias como infinito, menos o início (distância 0)
2. Cria um heap com o vértice de início
3. Enquanto o heap não está vazio:
   - Retira o vértice com menor distância
   - Para cada vizinho deste vértice:
     - Se encontrar um caminho mais curto, atualiza a distância
     - Adiciona o vizinho ao heap
4. Reconstrói o caminho andando para trás do fim até o início
5. Retorna um dicionário com:
   - `caminho_ids`: Lista de vértices do caminho
   - `distancia_pixels`: Distância em pixels
   - `distancia_metros`: Distância em metros (usando FATOR_ESCALA)
   - `nos_explorados`: Quantos vértices foram visitados
   - `tempo_ms`: Tempo de processamento em milissegundos

**Retorna:** Dicionário com informações da rota, ou `None` se não há caminho.

---

```python
def carregar_mapa_poly(caminho_arquivo) -> tuple
```

**O que faz:** Lê um arquivo `.poly` e carrega vértices e arestas.

**Explicação fácil:**
O arquivo `.poly` é um formato customizado que guarda informações do mapa. Este função lê este arquivo e cria objetos `Vertice` e `Aresta` em Python.

**Formato do arquivo `.poly`:**
```
<total_vertices>  <cols>  <linhas>  <camadas>
<id>  <x>  <y>
... (total_vertices linhas)
<total_arestas>  <linhas_de_arestas>
<id>  <orig>  <dest>  <tipo>
... (total_arestas linhas)
0
```

**Retorna:** Tupla (lista_vertices, lista_arestas)

---

---

### 3️⃣ **conversor_osm.py** - Conversão de Mapas OSM

#### 📌 Classe: `Node`

Representa um nó do arquivo OSM.

```python
class Node:
    def __init__(self, id_original: int, lat: float, lon: float, id_interno: int)
```

**Atributos:**
- `id_original`: ID no arquivo OSM original
- `lat`: Latitude (WGS84)
- `lon`: Longitude (WGS84)
- `x`: Coordenada X em UTM (calculada depois)
- `y`: Coordenada Y em UTM (calculada depois)
- `id_interno`: ID sequencial (0, 1, 2, ...)

**Explicação fácil:** Um nó é um ponto no mapa vindo do OpenStreetMap. Ele tem coordenadas em latitude/longitude que depois são convertidas para um sistema de coordenadas local (UTM).

---

#### 📌 Classe: `Way`

Representa um caminho (rua) no arquivo OSM.

```python
class Way:
    def __init__(self)
```

**Atributos:**
- `node_ids`: Lista de IDs dos nós que formam este caminho
- `is_oneway`: Booleano indicando se é mão única

**Explicação fácil:** Uma forma é uma sequência de pontos que forma uma rua. Por exemplo, uma rua pode passar pelos pontos [1, 2, 3, 4].

---

#### 🔧 **Funções Globais**

```python
def converter_para_utm(lat_deg: float, lon_deg: float) -> tuple
```

**O que faz:** Converte coordenadas de WGS84 (latitude/longitude) para UTM.

**Explicação fácil:**
- **WGS84** é o sistema que GPS usa (latitude entre -90 e +90, longitude entre -180 e +180)
- **UTM** é um sistema local em metros que funciona melhor para mapas pequenos

A conversão usa matemática complexa com parâmetros elipsoidais:
- `A_WGS84`: Raio maior da Terra ≈ 6.378.137 metros
- `F_WGS84`: Achatamento da Terra
- `K0`: Fator de escala da projeção (0.9996)
- `LON0_DEG`: Meridiano central para a zona (-45° para Brasil)

**Retorna:** Tupla (x_utm, y_utm)

---

```python
def reduzir_escala(pontos: list, redutor: float)
```

**O que faz:** Normaliza e redimensiona as coordenadas para caber na tela.

**Explicação fácil:**
1. Encontra o ponto mais à esquerda e mais em cima
2. Subtrai estas coordenadas de todos os pontos (normaliza)
3. Divide por um fator redutor (escala para caber na tela)

**Efeito:** Todos os pontos começam em (0, 0) e ficam dentro de um tamanho razoável.

---

```python
def parse_osm(filename: str) -> str
```

**O que faz:** Lê um arquivo OSM completo e gera um arquivo `.poly`.

**Explicação fácil:**
1. Lê o arquivo XML do OpenStreetMap
2. Extrai todos os nós (pontos) e vias (ruas)
3. Converte coordenadas de WGS84 para UTM
4. Redimensiona para caber na tela
5. Inverte eixo Y (porque gráficos usam Y invertido)
6. Salva tudo em um arquivo `.poly`

**Processo detalhado:**
1. **Parsing dos nós:** Encontra cada `<node>` no XML e cria um objeto `Node`
2. **Parsing das vias:** Encontra cada `<way>` e verifica se é mão única
3. **Tratamento de mão única reversa:** Se `oneway=-1`, inverte a ordem dos nós
4. **Normalização:** Reduz escala para caber em uma janela gráfica
5. **Inversão de Y:** Ajusta coordenadas para sistema de gráficos
6. **Escrita:** Salva em formato `.poly` no diretório `out/`

**Retorna:** Caminho do arquivo `.poly` gerado

---

---

### 4️⃣ **main.py** - Interface de Linha de Comando

#### 🔧 **Funções de Controle**

```python
def limpar_tela()
```

**O que faz:** Limpa o terminal.

**Explicação fácil:** Funciona em Windows (`cls`) e Linux/Mac (`clear`).

---

```python
def pausar()
```

**O que faz:** Pausa o programa até o usuário pressionar Enter.

**Explicação fácil:** Espera o usuário ler a mensagem e confirmar para continuar.

---

```python
def exibir_menu() -> str
```

**O que faz:** Mostra um menu com 3 opções e retorna a escolha.

**Opções:**
1. Importar novo mapa OSM, converter para POLY e calcular rota
2. Carregar mapa POLY existente e calcular rota
3. Sair

---

```python
def executar_sistema()
```

**O que faz:** Loop principal do programa. Executa continuamente até o usuário sair.

**Fluxo:**
1. Mostra menu
2. Se opção 1: Carrega arquivo OSM, converte para POLY
3. Se opção 2: Carrega arquivo POLY existente
4. Se arquivo existir:
   - Carrega vértices e arestas
   - Constrói grafo
   - Pede ao usuário origem e destino
   - Executa Dijkstra
   - Mostra resultado
5. Volta ao menu

---

---

### 5️⃣ **interface.py** - Interface Gráfica (PyQt6)

#### 📌 Classe: `GrafoItem`

Renderiza o grafo na tela usando PyQt6.

```python
class GrafoItem(QGraphicsItem)
```

**O que faz:** Desenha vértices, arestas, rotas, e toda a visualização gráfica do mapa.

##### 🔧 **Métodos principais**

```python
def atualizar_geometria()
```

**O que faz:** Recalcula as linhas das arestas quando o mapa é carregado.

**Processo:**
1. Encontra os limites do mapa (min/max x e y)
2. Cria retângulo de renderização com margem
3. Para cada aresta:
   - Se for mão única: encurta a linha e adiciona seta
   - Se for mão dupla: cria linha simples
4. Prepara estruturas para desenho eficiente

---

```python
def boundingRect() -> QRectF
```

**O que faz:** Retorna a área retangular que engloba todo o grafo.

---

```python
def paint(painter, option, widget)
```

**O que faz:** Desenha tudo na tela.

**Elementos desenhados:**
- Arestas mão dupla (azul)
- Arestas mão única com setas (verde)
- Pesos das arestas (se ativado)
- Rota encontrada (laranja destacado)
- Vértices com cores especiais:
  - Verde: Origem
  - Vermelho: Destino
  - Amarelo: Vértice temporário
  - Cinza: Normal

---

#### 📌 Classe: `VisualizadorMapa`

Gerencia visualização e interações do mapa.

```python
class VisualizadorMapa(QGraphicsView)
```

##### 🔧 **Métodos principais**

```python
def wheelEvent(self, event)
```

**O que faz:** Permite zoom com roda do mouse.

**Funcionamento:**
- Roda para cima: Zoom in (20% maior)
- Roda para baixo: Zoom out (20% menor)

---

```python
def mousePressEvent(self, event)
```

**O que faz:** Trata cliques do mouse.

**Clique direito:** Executa ação conforme o modo selecionado:
- Buscar caminho: Seleciona origem/destino
- Adicionar vértice: Cria novo ponto
- Remover vértice: Deleta ponto clicado
- Adicionar/remover aresta: Conecta ou desconecta pontos

---

#### 📌 Classe: `Janela`

Interface principal da aplicação.

```python
class Janela(QMainWindow)
```

**O que faz:** Cria a janela principal com painéis de controle e visualização.

##### 🔧 **Atributos principais**

```
- origem_selecionada: ID do vértice de origem
- destino_selecionado: ID do vértice de destino
- caminho_resultado: Lista de vértices da rota encontrada
- vertices: Lista de todos os vértices
- arestas: Lista de todas as arestas
- grafo: Lista de adjacência
```

##### 🔧 **Métodos principais**

```python
def init_ui()
```

**O que faz:** Cria toda a interface gráfica com botões e painéis.

---

```python
def importar_mapa()
```

**O que faz:** Abre diálogo para selecionar arquivo `.poly` e carrega o mapa.

---

```python
def converter_usar_osm()
```

**O que faz:** Abre diálogo para selecionar arquivo `.osm`, converte para `.poly` e carrega.

---

```python
def calcular_caminho()
```

**O que faz:** Executa Dijkstra entre origem e destino selecionados.

**Resultado:**
- Mostra estatísticas (tempo, nós explorados, distância)
- Destaca a rota no mapa com cor laranja

---

```python
def tratar_clique_mapa(x, y)
```

**O que faz:** Processa clique direito no mapa conforme o modo.

**Modos:**
1. **Buscar Caminho**: Seleciona origem (verde) e destino (vermelho)
2. **Adicionar Vértice**: Cria novo ponto
3. **Remover Vértice**: Deleta ponto mais próximo
4. **Adicionar/Remover Aresta**: Conecta/desconecta dois pontos

---

```python
def copiar_imagem_grafo()
```

**O que faz:** Copia uma imagem do mapa para a área de transferência.

---

```python
def refatorar_indices()
```

**O que faz:** Reorganiza IDs dos vértices quando um é removido.

**Processo:**
1. Cria mapa de IDs antigos → novos
2. Renumera todos os vértices sequencialmente
3. Atualiza as arestas com novos IDs
4. Remove arestas que apontam para vértices inexistentes

**Explicação fácil:** Se você tem vértices 0, 1, 3, 5 e remove o 1, eles viram 0, 1, 2, 3.

---

```python
def atualizar_interface_status()
```

**O que faz:** Atualiza os rótulos de status na interface.

**Mostra:**
- ID da origem com cor verde
- ID do destino com cor vermelha
- Estado atual do programa

---

```python
def atualizar_estrutura_grafo()
```

**O que faz:** Reconstrói o grafo quando vértices ou arestas são adicionados/removidos.

**Processo:**
1. Reconstrói a lista de adjacência
2. Cria mapa de vértices por ID
3. Limpa caminho anterior
4. Atualiza renderização

---

```python
def limpar_selecoes()
```

**O que faz:** Reseta origem, destino e caminho encontrado.

---

### 6️⃣ **config.py** - Configurações

```python
FATOR_ESCALA = 2.0
```

**O que é:** Fator para converter distâncias de pixels para metros.

**Explicação fácil:** Se um segmento de rua tem 10 pixels na tela, ele representa 20 metros na realidade (10 × 2.0).

---

---

## 🚀 Como Usar

### **Modo Terminal (main.py)**

```bash
python src/main.py
```

**Menu:**
1. Escolha importar mapa OSM ou carregar POLY existente
2. Digite a origem e destino
3. Veja o resultado do Dijkstra

### **Modo Interface Gráfica (interface.py)**

```bash
python src/interface.py
```

**Passos:**
1. Clique em "Importar Mapa (.poly)" ou "Converter e Usar Arquivo .osm"
2. Selecione o arquivo desejado
3. Use o combo box para escolher o modo de interação
4. Clique direito no mapa para selecionar origem (1º clique) e destino (2º clique)
5. Clique em "Calcular Menor Caminho"
6. Veja a rota destacada em laranja

**Recursos extras:**
- 🔍 Roda do mouse: Zoom in/out
- ☑️ **Mostrar IDs**: Exibe número de cada vértice
- ☑️ **Mostrar Pesos**: Exibe distância de cada aresta
- ☑️ **Ocultar Elementos Fora da Rota**: Mostra só a rota encontrada
- ☑️ **Ocultar Apenas Vértices Não Selecionados**: Mostra origem, destino e rota
- 📋 **Copiar Grafo**: Copia imagem para área de transferência

---

## 📝 Exemplo de Uso

### Arquivo `mapaUFG.poly`

```
5 2 0 1
0 0.0 0.0
1 10.5 5.2
2 20.3 15.7
3 30.1 25.0
4 15.0 35.5
4 1
0 0 1 2
1 1 2 2
2 2 3 1
3 0 4 2
0
```

**Interpretação:**
- 5 vértices, cada um com (id, x, y)
- 4 arestas:
  - 0→1 (mão dupla)
  - 1→2 (mão dupla)
  - 2→3 (mão única)
  - 0→4 (mão dupla)

Se buscar de 0 para 3, Dijkstra encontrará: `0 → 1 → 2 → 3`

---

## 🎓 Resumo das Estruturas de Dados

| Estrutura | Complexidade | Uso |
|-----------|-------------|-----|
| **Heap** | O(log n) insert/extract | Otimizar Dijkstra |
| **Lista de Adjacência** | O(1) acesso | Guardar grafo |
| **Dijkstra com Heap** | O((V+E)log V) | Encontrar caminho mínimo |

---

## 📚 Referências

- **Algoritmo de Dijkstra:** Encontra o caminho mais curto em um grafo
- **Heap Min:** Estrutura otimizada para prioridade
- **Conversão UTM:** Transforma coordenadas de globo em sistema local
- **OpenStreetMap:** Base de dados de mapas geográficos

---

**Documentação atualizada:** 14 de junho de 2026
