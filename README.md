Auto-Dimensionamento de Paredes no Revit via Dynamo & Python
📋 Sobre o Projeto
Este script em Python para o Dynamo automatiza a criação de cotas lineares totais para paredes no Revit. Ele analisa a geometria das paredes selecionadas, identifica as faces extremas (pontas) com base no vetor de orientação geométrica, e desenha uma cota ortogonal a uma distância (offset) predefinida. Ideal para acelerar a documentação de projetos executivos e a rotina de compatibilização.

🚀 Funcionalidades
Leitura Geométrica Avançada: Extrai sólidos reais dos elementos, ignorando linhas de eixo simples.

Filtro de Faces Paralelas: Utiliza Produto Escalar (Dot Product) para ignorar faces laterais e focar apenas nas extremidades.

Tratamento Seguro de Origem (Fallback): Previne erros da API do Revit calculando o centro da face via BoundingBox caso a propriedade nativa Origin falhe em paredes com recortes ou uniões complexas.

Conversão Automática de Unidades: Permite o input paramétrico em milímetros (padrão de detalhamento) e converte internamente para os pés fracionais (padrão da API do Revit).

🛠️ Pré-requisitos
Revit: Testado nas versões recentes (2020+).

Dynamo: Compatível com os motores IronPython.

Pacotes: Nenhuma biblioteca customizada (Custom Node) é necessária. O script utiliza 100% da RevitAPI nativa.

📦 Como Configurar no Dynamo (Inputs)
No nó do Python Script, estruture os fios de entrada da seguinte forma:

IN[0] (Vista): A vista do Revit onde a cota será gerada. Se desconectado, o script assume a vista ativa (ActiveView).

IN[1]: Não utilizado nesta versão.

IN[2] (Elementos): Lista de elementos (Paredes) selecionados no modelo.

IN[3] (Offset): Distância de deslocamento da linha de cota em relação à parede, em milímetros (Numérico). Padrão assumido caso vazio: 600.0 mm.

🧠 Entendendo o Código (Principais Funções)
1. Sistema de Coordenadas e Vetores
Python
curva = paredes[0].Location.Curve
vetor_parede = curva.GetEndPoint(1).Subtract(curva.GetEndPoint(0)).Normalize()
vetor_perp = XYZ(-vetor_parede.Y, vetor_parede.X, 0).Normalize()
O script lê a linha base da parede para determinar sua direção longitudinal (vetor_parede). Em seguida, inverte os eixos para criar um vetor perpendicular (vetor_perp). Isso garante que a cota será sempre afastada lateralmente da parede, independentemente da rotação do elemento no projeto.

2. O Sistema de "Score" (Pontuação de Faces)
Python
score = pt_f.DotProduct(vetor_parede)
Para não depender de caixas de contorno estáticas, o script projeta o ponto central de cada face plana ao longo do vetor da parede. A face com o menor score matemático representa o extremo inicial, e a de maior score representa o extremo final. Faces internas de portas e janelas são ignoradas por não atingirem as pontuações máximas e mínimas.

3. Proteção contra Exceções Geométricas
Python
try:
    pt_f = face.Origin
except:
    bb = face.GetBoundingBox()
    u = (bb.Min.U + bb.Max.U) / 2.0
    v = (bb.Min.V + bb.Max.V) / 2.0
    pt_f = face.Evaluate(UV(u, v))
Quando paredes são unidas a pisos, lajes ou outras paredes, as faces podem ser divididas em geometrias complexas, fazendo a propriedade Origin retornar um erro (TypeError: property cannot be read). O bloco except atua como um plano de contingência, avaliando o centro da face através de coordenadas UV (Evaluate), impedindo que o script quebre.

4. Transação e Instanciação no Revit
Python
TransactionManager.Instance.EnsureInTransaction(doc)
dim = doc.Create.NewDimension(view, linha_cota, refs)
TransactionManager.Instance.TransactionTaskDone()
Com a linha geométrica da cota gerada em memória (linha_cota) e as referências das faces extremas capturadas (refs), o script interage com o banco de dados do Revit (Documento) abrindo uma transação controlada para desenhar o elemento final da cota linear na tela.

📝 Saídas (Outputs)
O script foi desenhado para facilitar o debug:

OUT[0]: Retorna o elemento da Cota (Dimension) criado com sucesso na API.

OUT[1]: Retorna um log textual detalhado, fundamental para rastrear paredes sem geometria, faces inválidas ou erros de transação (ex: linha muito curta ou Revit rejeitou).
