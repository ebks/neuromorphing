---

# CAPÍTULO 05

# ARQUITETURA E DINÂMICA DE REDES NEURAIS DE SPIKES (SNNS) EM BRIAN2

---

![imagem](neuro.png)

*Com os modelos matemáticos de neurônios individuais e sinapses estabelecidos, e munidos das ferramentas computacionais fornecidas pelo Brian2, este capítulo avança para o nível de redes neurais de spikes (SNNs). O foco desloca-se da dinâmica de componentes isolados para o comportamento emergente que surge da interação entre múltiplas unidades neuronais interconectadas. Exploraremos as principais topologias arquitetônicas utilizadas na construção de SNNs, incluindo redes feedforward, recorrentes e aleatórias, discutindo suas inspirações biológicas e propriedades computacionais. Detalharemos as funcionalidades do Brian2 para a especificação precisa da conectividade sináptica entre populações neuronais, utilizando o método `connect()` e suas diversas opções para criar padrões de conexão que vão desde mapeamentos simples até estruturas complexas e probabilísticas. Para solidificar a compreensão, desenvolveremos e simularemos dois estudos de caso práticos em Brian2: primeiro, uma rede feedforward multicamadas, ilustrando a propagação de sinais através de estágios de processamento; segundo, uma rede recorrente incorporando o motivo computacional fundamental da inibição lateral, demonstrando como a interação entre excitação e inibição pode moldar a dinâmica da rede. Finalmente, abordaremos técnicas essenciais para a visualização e análise da atividade de SNNs simuladas, como a geração de raster plots para observar padrões de disparo populacional e o cálculo de taxas de disparo médias, ferramentas indispensáveis para interpretar os resultados das simulações e inferir a função computacional da rede.*

---

**5.1. Topologias de Rede: Feedforward, Recorrente, Aleatória**

A arquitetura, ou **topologia**, de uma rede neural define o padrão geral de como seus neurônios estão organizados e interconectados. A escolha da topologia é fundamental, pois restringe e molda o fluxo de informação, a dinâmica intrínseca da rede e, consequentemente, os tipos de computações que ela pode realizar eficientemente. As SNNs, inspirando-se na diversidade estrutural do cérebro, podem ser construídas com base em diferentes arranjos topológicos (Gerstner et al., 2014; Litwin-Kumar & Doiron, 2012). As categorias mais fundamentais incluem:

1.  **Redes Feedforward (Avanço Direto):**
    *   **Estrutura:** Neurônios são organizados em camadas distintas (e.g., camada de entrada, camadas ocultas, camada de saída). As conexões sinápticas fluem estritamente em uma direção, da camada $k$ para a camada $k+1$, sem ciclos ou conexões dentro da mesma camada ou de volta para camadas anteriores.
    *   **Inspiração Biológica:** Vias sensoriais primárias no cérebro (e.g., via visual ventral inicial, via auditiva) exibem uma organização largamente feedforward, onde a informação é processada sequencialmente através de diferentes áreas ou núcleos, com cada estágio extraindo características progressivamente mais complexas.
    *   **Propriedades Computacionais:** Redes feedforward são adequadas para tarefas de mapeamento de padrões de entrada para padrões de saída, como classificação, reconhecimento de objetos ou regressão. O processamento é inerentemente direcionado, transformando a representação da informação a cada camada. A ausência de recorrência simplifica a análise e o treinamento (em comparação com redes recorrentes), mas limita sua capacidade de processar sequências temporais ou manter memória interna de estados passados de forma intrínseca. A dinâmica é largamente determinada pela entrada e pela propagação através das camadas.
    *   **Implementação em SNNs:** Em SNNs feedforward, os spikes de entrada ativam neurônios na primeira camada, que por sua vez disparam e ativam neurônios na camada seguinte, e assim por diante. O tempo de processamento é relacionado ao tempo de propagação dos spikes através das camadas.

2.  **Redes Recorrentes (Recurrent Networks):**
    *   **Estrutura:** Caracterizam-se pela presença de ciclos ou loops nas conexões sinápticas. As conexões podem ocorrer dentro da mesma camada de neurônios (recorrência local) ou entre diferentes camadas, incluindo conexões que apontam para trás (feedback). Um caso extremo é uma rede totalmente conectada, onde cada neurônio pode, em princípio, conectar-se a qualquer outro neurônio, incluindo ele mesmo (autoconexões, embora menos comuns biologicamente).
    *   **Inspiração Biológica:** A conectividade recorrente é onipresente no cérebro, especialmente no neocórtex e no hipocampo (ver Seção 2.5). Circuitos locais dentro das colunas corticais são densamente recorrentes. Projeções de feedback de áreas de ordem superior para áreas de ordem inferior também são comuns.
    *   **Propriedades Computacionais:** A recorrência introduz dinâmica temporal complexa na rede. A atividade não é determinada apenas pela entrada atual, mas também pelo estado interno da rede (sua história recente de atividade), que é mantido e propagado através dos loops de feedback. Isso confere às redes recorrentes a capacidade de:
        *   *Manter Memória de Curto Prazo:* A atividade pode persistir na rede mesmo após o fim do estímulo de entrada (atividade persistente), formando uma base para memória de trabalho.
        *   *Processar Sequências Temporais:* Podem detectar padrões que se desdobram no tempo, integrar informação ao longo de sequências e gerar saídas sequenciais (e.g., controle motor, produção de linguagem).
        *   *Gerar Dinâmica Intrínseca:* Podem exibir atividade espontânea complexa, oscilações rítmicas e estados atratores (padrões de atividade estáveis que representam memórias ou decisões).
        *   *Implementar Modelos Preditivos:* Conexões de feedback podem carregar predições ou expectativas que são comparadas com a entrada atual.
    *   **Implementação em SNNs:** A simulação de SNNs recorrentes pode ser computacionalmente mais exigente devido à dinâmica complexa e potencial instabilidade (atividade descontrolada). O controle da excitação através de circuitos inibitórios recorrentes é crucial para manter a estabilidade (ver Seção 5.4).

3.  **Redes com Conectividade Aleatória:**
    *   **Estrutura:** As conexões entre neurônios não seguem um padrão estritamente estratificado ou totalmente recorrente, mas são estabelecidas probabilisticamente, muitas vezes com uma probabilidade que depende da distância entre os neurônios ou de seus tipos celulares (e.g., conexões excitatórias-inibitórias). Podem exibir características tanto feedforward quanto recorrentes, dependendo dos parâmetros da conectividade.
    *   **Inspiração Biológica:** Embora a conectividade cerebral não seja puramente aleatória (é altamente estruturada pela genética e pela plasticidade), modelos de redes aleatórias (como os modelos de Erdős-Rényi ou de mundo pequeno) são frequentemente usados como um ponto de partida para estudar princípios gerais de dinâmica de rede ou como modelos para certas estruturas (e.g., conectividade local no córtex pode ter um componente aleatório sobreposto a uma estrutura específica).
    *   **Propriedades Computacionais:** Redes aleatórias, especialmente quando operam em um regime balanceado entre excitação e inibição, podem exibir dinâmica rica e caótica (atividade irregular e assíncrona), similar à observada em gravações corticais *in vivo* (van Vreeswijk & Sompolinsky, 1996; Brunel, 2000). Esta dinâmica caótica pode ser útil para:
        *   *Amplificação de Sinais:* Pequenos inputs podem ser amplificados pela dinâmica recorrente.
        *   *Computação em Reservatório (Reservoir Computing):* A dinâmica rica e de alta dimensionalidade de uma rede recorrente aleatória (o "reservatório") pode ser usada como uma base para mapear inputs complexos para representações que são então lidas por uma camada de saída treinável (Maass et al., 2002; Sussillo & Abbott, 2009). Apenas os pesos da camada de saída precisam ser treinados, simplificando o aprendizado.
        *   *Geração de Variabilidade:* A dinâmica intrínseca pode fornecer uma fonte de variabilidade útil para exploração em tarefas de aprendizado por reforço ou para modelar a variabilidade observada em respostas neuronais.
    *   **Implementação em SNNs:** A conectividade aleatória é facilmente implementada em Brian2 usando a opção `p` no método `connect()` (ver Seção 5.2). O balanço entre excitação e inibição é um parâmetro crítico a ser ajustado para obter dinâmicas estáveis e interessantes.

**Figura 5.1**: Topologias de Rede Neuronal. Diagramas esquemáticos: (a) Rede Feedforward: Camadas distintas (Input, Hidden, Output) com conexões direcionais para frente. (b) Rede Recorrente: Neurônios interconectados com loops de feedback (conexões dentro da camada ou para camadas anteriores). (c) Rede com Conectividade Aleatória: Conexões estabelecidas probabilisticamente entre os neurônios, resultando em uma estrutura menos regular.

Na prática, muitas arquiteturas de SNNs combinam elementos dessas topologias básicas. Por exemplo, um modelo do córtex pode incluir múltiplas camadas (feedforward) onde cada camada possui conectividade recorrente local (aleatória ou estruturada) entre neurônios excitatórios e inibitórios. A escolha da topologia deve ser guiada pela tarefa computacional a ser realizada e pela estrutura biológica que se deseja modelar ou da qual se busca inspiração. Brian2 fornece as ferramentas para construir redes com qualquer uma dessas topologias, como veremos na próxima seção.

**5.2. Conectividade Sináptica em Brian2: O Método `connect()`**

Após definir as populações neuronais usando `NeuronGroup`, o próximo passo crucial na construção de uma SNN em Brian2 é estabelecer as conexões sinápticas entre elas usando o objeto `Synapses`. Conforme introduzido na Seção 4.2, a criação do objeto `Synapses` define o modelo das sinapses (suas variáveis, dinâmica e regras `on_pre`/`on_post`), mas não especifica quais neurônios individuais estão conectados. Esta especificação é feita através do método `connect()` do objeto `Synapses`.

O método `Synapses.connect()` é extremamente versátil e oferece múltiplas maneiras de definir a conectividade, permitindo a criação de padrões de conexão simples ou altamente complexos com eficiência (Stimberg et al., 2019a). A assinatura básica é `S.connect(condition=None, i=None, j=None, p=None, n=None, skip_if_invalid=False, namespace=None, level=0)`. Os argumentos mais importantes são:

*   **`i` e `j` (Índices Explícitos):** Permitem especificar diretamente os índices dos neurônios pré-sinápticos (`i`) e pós-sinápticos (`j`) a serem conectados.
    *   `S.connect(i=k, j=l)`: Conecta o neurônio pré-sináptico de índice `k` ao neurônio pós-sináptico de índice `l`. Ambos `k` e `l` devem ser inteiros.
    *   `S.connect(i=[k1, k2, ...], j=[l1, l2, ...])`: Conecta pares explícitos $(k1, l1), (k2, l2), ...$. As listas ou arrays `i` e `j` devem ter o mesmo comprimento.
    ```python
    # Exemplo: Conexão um-para-um entre dois grupos de mesmo tamanho N
    # S.connect(i=np.arange(N), j=np.arange(N)) # ou S.connect(j='i')
    # Exemplo: Conectar neurônio pré 0 aos pós 1 e 2
    # S.connect(i=[0, 0], j=[1, 2])
    ```

*   **`condition` (Conectividade Baseada em Condição):** Permite criar conexões apenas entre os pares de neurônios $(i, j)$ que satisfazem uma determinada condição booleana. A condição é fornecida como uma string que pode envolver os índices `i` (pré) e `j` (pós), o número de neurônios nos grupos (`N_pre`, `N_post`), e outras variáveis ou funções disponíveis no *namespace*.
    *   `S.connect(condition='i != j')`: Conecta todos os pares exceto autoconexões (útil para conexões recorrentes dentro de um grupo).
    *   `S.connect(condition='abs(i - j) < 5')`: Conecta neurônios que estão próximos em índice (assumindo alguma topologia espacial implícita nos índices).
    *   `S.connect(condition='rand() < 0.1')`: Equivalente a `p=0.1` (veja abaixo).
    *   `S.connect(condition='i==j and i<10')`: Conecta os primeiros 10 neurônios a si mesmos.
    Brian2 itera sobre todos os pares $(i, j)$ possíveis e cria uma sinapse se a condição for avaliada como verdadeira.

*   **`j` (Mapeamento Baseado em String/Função):** Se `j` for fornecido como uma string (em vez de um índice ou array), ela é interpretada como uma expressão que calcula o índice pós-sináptico `j` com base no índice pré-sináptico `i`.
    *   `S.connect(j='i')`: Conexão um-para-um (assumindo `N_pre == N_post`).
    *   `S.connect(j='k for k in range(N_post)')`: Conexão um-para-todos (cada pré `i` conecta a todos os pós).
    *   `S.connect(j='i + 1')`: Conexão deslocada (requer tratamento de borda, e.g., `j='(i + 1) % N_post'`).
    *   `S.connect(j='p<0.1')`: Geração de conexões esparsas (funciona como `p` se `j` for booleano).
    *   Pode-se usar loops e lógica mais complexa dentro da string.

*   **`p` (Conectividade Probabilística):** Cria conexões aleatoriamente com uma probabilidade especificada $p$. Para cada par $(i, j)$ possível, uma sinapse é criada com probabilidade $p$.
    *   `S.connect(p=0.1)`: Cria uma rede esparsa onde, em média, cada neurônio pré-sináptico se conecta a 10% dos neurônios pós-sinápticos.
    *   Útil para modelar conectividade aleatória ou esparsa observada biologicamente.

*   **`n` (Número de Conexões):** Especifica o número desejado de sinapses a serem criadas. Pode ser usado em conjunto com `p` ou `condition` para controlar a densidade da conexão.
    *   Se `n` for um inteiro: Tenta criar exatamente `n` sinapses, selecionadas aleatoriamente entre os pares que satisfazem a condição ou a probabilidade `p`.
    *   Se `n` for uma string: Avaliada como uma expressão para determinar o número de sinapses a serem criadas a partir de cada neurônio pré-sináptico (e.g., `n='rand()*5'` para um número aleatório de conexões por neurônio pré).

*   **`skip_if_invalid` (Booleano):** Se `True`, ignora silenciosamente tentativas de criar conexões inválidas (e.g., índice fora dos limites). Se `False` (padrão), gera um erro.

*   **`namespace` (Dicionário):** Permite passar um dicionário contendo variáveis ou funções adicionais que podem ser usadas nas strings `condition` ou `j`.

**Figura 5.2**: Métodos de Conectividade em Brian2 (`Synapses.connect()`). Ilustrações esquemáticas: (a) Conexão explícita `i=[...], j=[...]`. (b) Conexão por condição `condition='...'`. (c) Conexão por mapeamento `j='f(i)'`. (d) Conexão probabilística `p=...`. Mostra como diferentes chamadas a `connect` resultam em diferentes matrizes de adjacência (padrões de conexão).

**Considerações:**

*   **Eficiência:** Brian2 é otimizado para gerar conexões eficientemente, mesmo para redes grandes e conexões probabilísticas ou baseadas em condições. A geração de código C++ acelera ainda mais este processo.
*   **Múltiplas Chamadas a `connect()`:** Pode-se chamar `connect()` múltiplas vezes no mesmo objeto `Synapses` para construir padrões de conectividade complexos em etapas. Por exemplo, conectar primeiro com probabilidade $p_1$ e depois adicionar conexões específicas com `i=..., j=...`.
*   **Acesso aos Índices:** Após a conexão, os índices pré e pós-sinápticos de cada sinapse criada podem ser acessados através dos atributos `S.i` e `S.j` (arrays de inteiros).
*   **Sinapses Múltiplas:** Por padrão, Brian2 permite a criação de múltiplas sinapses entre o mesmo par de neurônios pré e pós-sinápticos se o método de conexão resultar nisso (e.g., com `n > 1` ou múltiplas chamadas a `connect`).

A flexibilidade do método `connect()` permite aos usuários do Brian2 implementar uma vasta gama de arquiteturas de rede, desde topologias regulares e estruturadas até conectividade complexa e probabilisticamente definida, refletindo a diversidade encontrada em circuitos neurais biológicos ou explorando novas arquiteturas para computação neuromórfica.

**5.3. Estudo de Caso: Construindo uma Rede Feedforward Simples**

Vamos agora aplicar os conceitos para construir e simular uma rede neural de spikes feedforward simples de três camadas usando Brian2. A rede consistirá em:
1.  Uma camada de entrada (`Input Layer`) que gera spikes (usaremos `PoissonGroup`).
2.  Uma camada oculta (`Hidden Layer`) de neurônios LIF.
3.  Uma camada de saída (`Output Layer`) de neurônios LIF.
As conexões serão excitatórias e direcionais: Input -> Hidden e Hidden -> Output.

**Objetivo:** Simular a propagação de atividade de spikes através das camadas da rede feedforward e visualizar a atividade em cada camada.

```python
# Importar bibliotecas
import brian2 as b2
import matplotlib.pyplot as plt
import numpy as np

# --- Parâmetros Globais ---
b2.prefs.codegen.target = 'cython' # Usar Cython para desempenho
b2.defaultclock.dt = 0.1 * b2.ms  # Passo de tempo da simulação

# --- Parâmetros da Rede ---
# Tamanho das camadas
N_input = 100
N_hidden = 50
N_output = 10
# Taxa de disparo da camada de entrada
input_rate = 20 * b2.Hz
# Duração da simulação
simulation_duration = 500 * b2.ms

# --- Parâmetros dos Neurônios LIF (para camadas Oculta e Saída) ---
tau = 15 * b2.ms
Vt = -55 * b2.mV
Vr = -70 * b2.mV
El = -65 * b2.mV
Rm = 100 * b2.Mohm # R = tau / C -> C = 15ms/100MOhm = 150pF
refractory_p = 2 * b2.ms

# Equações LIF (baseadas em corrente sináptica)
lif_eqs = '''
dv/dt = (El - v + Rm * I_syn) / tau : volt (unless refractory)
I_syn : amp # Corrente sináptica total recebida
'''

# --- Criação das Camadas Neuronais ---
# Camada de Entrada: Grupo Poisson
input_layer = b2.PoissonGroup(N_input, rates=input_rate)

# Camada Oculta: Neurônios LIF
hidden_layer = b2.NeuronGroup(N_hidden, lif_eqs,
                              threshold='v > Vt',
                              reset='v = Vr',
                              refractory=refractory_p,
                              method='euler') # Euler pode ser mais rápido para redes grandes
hidden_layer.v = El
hidden_layer.I_syn = 0 * b2.nA

# Camada de Saída: Neurônios LIF
output_layer = b2.NeuronGroup(N_output, lif_eqs,
                              threshold='v > Vt',
                              reset='v = Vr',
                              refractory=refractory_p,
                              method='euler')
output_layer.v = El
output_layer.I_syn = 0 * b2.nA

# --- Criação das Sinapses Feedforward ---
# Parâmetros Sinápticos (simplificado: injeção de corrente)
tau_syn = 5 * b2.ms  # Constante de tempo da corrente sináptica
w_input_hidden = 4.0 * b2.nA # Peso Input -> Hidden
w_hidden_output = 6.0 * b2.nA # Peso Hidden -> Output
p_connect = 0.2 # Probabilidade de conexão entre camadas

# Modelo Sináptico (baseado em corrente com decaimento exponencial)
syn_model_current = '''
w : amp # Peso sináptico
dI_syn_post/dt = -I_syn_post/tau_syn : amp (summed)
'''
on_pre_current = 'I_syn_post += w'

# Sinapses Input -> Hidden
syn_IH = b2.Synapses(input_layer, hidden_layer,
                    model=syn_model_current, on_pre=on_pre_current)
syn_IH.connect(p=p_connect)
syn_IH.w = w_input_hidden
syn_IH.namespace['tau_syn'] = tau_syn # Passar tau_syn para o namespace da sinapse

# Sinapses Hidden -> Output
syn_HO = b2.Synapses(hidden_layer, output_layer,
                    model=syn_model_current, on_pre=on_pre_current)
syn_HO.connect(p=p_connect)
syn_HO.w = w_hidden_output
syn_HO.namespace['tau_syn'] = tau_syn

# --- Monitores ---
# Monitores de Spikes para cada camada
spikemon_input = b2.SpikeMonitor(input_layer)
spikemon_hidden = b2.SpikeMonitor(hidden_layer)
spikemon_output = b2.SpikeMonitor(output_layer)

# Monitor de Estado (opcional, para ver Vm de alguns neurônios ocultos)
statemon_hidden_v = b2.StateMonitor(hidden_layer, 'v', record=[0, 1, 2])

# --- Construção e Execução da Rede ---
network = b2.Network(input_layer, hidden_layer, output_layer,
                   syn_IH, syn_HO,
                   spikemon_input, spikemon_hidden, spikemon_output,
                   statemon_hidden_v)

print("Iniciando simulação da rede feedforward...")
network.run(simulation_duration, report='text', report_period=30*b2.second)
print("Simulação feedforward concluída.")

# --- Visualização ---
plt.figure(figsize=(12, 9))

# Raster Plot Camada de Entrada
plt.subplot(3, 1, 1)
plt.plot(spikemon_input.t / b2.ms, spikemon_input.i, '.k', markersize=1)
plt.title('Camada de Entrada (Poisson)')
plt.ylabel('Índice Neurônio')
plt.xlim(0, simulation_duration / b2.ms)
plt.ylim(-1, N_input)

# Raster Plot Camada Oculta
plt.subplot(3, 1, 2)
plt.plot(spikemon_hidden.t / b2.ms, spikemon_hidden.i, '.k', markersize=2)
plt.title('Camada Oculta (LIF)')
plt.ylabel('Índice Neurônio')
plt.xlim(0, simulation_duration / b2.ms)
plt.ylim(-1, N_hidden)

# Raster Plot Camada de Saída
plt.subplot(3, 1, 3)
plt.plot(spikemon_output.t / b2.ms, spikemon_output.i, '.k', markersize=3)
plt.title('Camada de Saída (LIF)')
plt.xlabel('Tempo (ms)')
plt.ylabel('Índice Neurônio')
plt.xlim(0, simulation_duration / b2.ms)
plt.ylim(-1, N_output)

plt.tight_layout()
plt.show()

# Opcional: Plotar Vm da camada oculta
plt.figure(figsize=(10, 4))
for i in range(len(statemon_hidden_v.record)):
    plt.plot(statemon_hidden_v.t/ms, statemon_hidden_v.v[i]/b2.mV, label=f'Neurônio Oculto {i}')
plt.xlabel('Tempo (ms)')
plt.ylabel('Vm (mV)')
plt.title('Potencial de Membrana (Camada Oculta - Exemplos)')
plt.legend()
plt.show()

```
Este código constrói uma SNN feedforward de três camadas. A camada de entrada (`PoissonGroup`) gera spikes aleatórios que acionam neurônios LIF na camada oculta (`hidden_layer`) através de sinapses excitatórias (`syn_IH`) conectadas probabilisticamente (`p=0.2`). A camada oculta, por sua vez, ativa neurônios LIF na camada de saída (`output_layer`) através de sinapses similares (`syn_HO`). Monitores registram os spikes em cada camada. A visualização resultante (raster plots) deve mostrar a atividade de entrada aleatória, uma atividade mais densa (ou esparsa, dependendo dos pesos e conectividade) na camada oculta, e uma atividade tipicamente mais esparsa na camada de saída, ilustrando a propagação e transformação do sinal através da rede. O plot opcional de Vm mostra a integração sub-limiar nos neurônios ocultos.

**Figura 5.3**: Atividade da Rede Feedforward Simulada. Raster plots gerados pelo código acima. Painel superior: Spikes da camada de entrada (Poisson). Painel do meio: Spikes da camada oculta (LIF), mostrando resposta à entrada. Painel inferior: Spikes da camada de saída (LIF), mostrando resposta à camada oculta. Os plots ilustram a propagação da atividade através das camadas.

**5.4. Estudo de Caso: Implementando uma Rede Recorrente com Inibição Lateral**

Agora, construiremos uma rede que incorpora conectividade recorrente e um motivo computacional comum: a **inibição lateral**. Neste padrão, neurônios excitatórios excitam interneurônios inibitórios, os quais, por sua vez, inibem outros neurônios excitatórios vizinhos (ou no mesmo grupo). Isso cria competição entre os neurônios excitatórios, onde o neurônio (ou grupo) mais ativo tende a suprimir a atividade dos outros. Usaremos uma única população de neurônios excitatórios e uma população de neurônios inibitórios.

**Objetivo:** Simular uma rede com populações excitatória (E) e inibitória (I) interconectadas (E->E, E->I, I->E) para observar dinâmicas competitivas ou oscilatórias resultantes da interação excitação-inibição.

```python
# Importar bibliotecas
import brian2 as b2
import matplotlib.pyplot as plt
import numpy as np

# --- Parâmetros Globais ---
b2.prefs.codegen.target = 'cython'
b2.defaultclock.dt = 0.1 * b2.ms

# --- Parâmetros da Rede ---
N_E = 80  # Neurônios Excitatórios
N_I = 20  # Neurônios Inibitórios
# Input Externo (para excitar a rede)
external_drive_rate = 10 * b2.Hz
# Duração da simulação
simulation_duration = 1 * b2.second # Simular por mais tempo para ver dinâmica

# --- Parâmetros dos Neurônios LIF (semelhantes para E e I, mas I pode ser mais rápido) ---
# Excitatório
tau_E = 20 * b2.ms; Vt_E = -50 * b2.mV; Vr_E = -65 * b2.mV; El_E = -65 * b2.mV; Rm_E = 100 * b2.Mohm; ref_E = 2*b2.ms
# Inibitório (pode ser mais rápido, menor tau, menor limiar)
tau_I = 10 * b2.ms; Vt_I = -52 * b2.mV; Vr_I = -65 * b2.mV; El_I = -65 * b2.mV; Rm_I = 100 * b2.Mohm; ref_I = 1*b2.ms

# Equações LIF (usaremos condutância para inibição ser mais realista)
# g_exc, g_inh representam condutâncias sinápticas
eqs_E = '''
dv/dt = (El_E - v + Rm_E*(g_exc - g_inh))/tau_E : volt (unless refractory)
dg_exc/dt = -g_exc / (5*ms) : amp # Condutância excitatória
dg_inh/dt = -g_inh / (10*ms) : amp # Condutância inibitória
'''
eqs_I = '''
dv/dt = (El_I - v + Rm_I*g_exc)/tau_I : volt (unless refractory)
dg_exc/dt = -g_exc / (5*ms) : amp # Só recebe excitação
'''
# Potenciais de reversão
E_exc = 0 * b2.mV
E_inh = -80 * b2.mV # Potencial reversão inibitório (GABA)

# --- Criação das Populações Neuronais ---
P_E = b2.NeuronGroup(N_E, eqs_E, threshold='v>Vt_E', reset='v=Vr_E', refractory=ref_E, method='euler')
P_I = b2.NeuronGroup(N_I, eqs_I, threshold='v>Vt_I', reset='v=Vr_I', refractory=ref_I, method='euler')

# Inicialização
P_E.v = El_E; P_E.g_exc = 0*b2.nA; P_E.g_inh = 0*b2.nA
P_I.v = El_I; P_I.g_exc = 0*b2.nA

# --- Input Externo (Poisson para a população E) ---
input_external = b2.PoissonGroup(N_E, rates=external_drive_rate) # Um input por neurônio E

# --- Criação das Sinapses ---
# Pesos (condutâncias máximas injetadas por spike)
w_input_E = 6 * b2.nS # Input Externo -> E
w_E_E = 0.5 * b2.nS # E -> E (recorrente)
w_E_I = 0.8 * b2.nS # E -> I
w_I_E = 1.5 * b2.nS # I -> E (inibição lateral)
p_connect_recurrent = 0.1 # Probabilidade de conexão recorrente

# Modelo Sináptico (baseado em condutância)
# A condutância pós-sináptica é incrementada pelo peso 'w'
on_pre_gexc = 'g_exc_post += w'
on_pre_ginh = 'g_inh_post += w'

# Conexões Input -> E
syn_input_E = b2.Synapses(input_external, P_E, on_pre=on_pre_gexc)
syn_input_E.connect(j='i') # Conecta input i ao neurônio E i
syn_input_E.w = w_input_E

# Conexões E -> E (Recorrente Excitatória)
syn_E_E = b2.Synapses(P_E, P_E, on_pre=on_pre_gexc)
syn_E_E.connect(condition='i!=j', p=p_connect_recurrent) # Conectar aleatoriamente, sem autoconexões
syn_E_E.w = w_E_E

# Conexões E -> I
syn_E_I = b2.Synapses(P_E, P_I, on_pre=on_pre_gexc)
syn_E_I.connect(p=p_connect_recurrent) # Conectar E aleatoriamente a I
syn_E_I.w = w_E_I

# Conexões I -> E (Inibição Lateral)
syn_I_E = b2.Synapses(P_I, P_E, on_pre=on_pre_ginh)
syn_I_E.connect(p=p_connect_recurrent * 2) # Inibição pode ser mais densa
syn_I_E.w = w_I_E

# --- Monitores ---
spikemon_E = b2.SpikeMonitor(P_E)
spikemon_I = b2.SpikeMonitor(P_I)
# Opcional: Monitorar taxas de disparo
ratemon_E = b2.PopulationRateMonitor(P_E)
ratemon_I = b2.PopulationRateMonitor(P_I)

# --- Construção e Execução da Rede ---
network = b2.Network(P_E, P_I, input_external,
                   syn_input_E, syn_E_E, syn_E_I, syn_I_E,
                   spikemon_E, spikemon_I, ratemon_E, ratemon_I)

print("Iniciando simulação da rede recorrente E-I...")
network.run(simulation_duration, report='text', report_period=30*b2.second)
print("Simulação recorrente concluída.")

# --- Visualização ---
plt.figure(figsize=(12, 7))

# Raster Plot População E
plt.subplot(2, 1, 1)
plt.plot(spikemon_E.t / b2.ms, spikemon_E.i, '.b', markersize=2, label='Excitatório')
plt.plot(spikemon_I.t / b2.ms, spikemon_I.i + N_E, '.r', markersize=2, label='Inibitório') # Deslocar índices I
plt.title('Raster Plot Rede Recorrente E-I')
plt.ylabel('Índice Neurônio (E: 0-79, I: 80-99)')
plt.xlim(0, simulation_duration / b2.ms)
plt.ylim(-1, N_E + N_I)
plt.legend(loc='upper right')

# Plot Taxas de Disparo Populacional
plt.subplot(2, 1, 2)
plt.plot(ratemon_E.t / b2.ms, ratemon_E.rate / b2.Hz, 'b', label='Taxa Pop. E')
plt.plot(ratemon_I.t / b2.ms, ratemon_I.rate / b2.Hz, 'r', label='Taxa Pop. I')
plt.title('Taxa de Disparo Populacional')
plt.xlabel('Tempo (ms)')
plt.ylabel('Taxa (Hz)')
plt.xlim(0, simulation_duration / b2.ms)
plt.legend(loc='upper right')

plt.tight_layout()
plt.show()
```
Este código configura uma rede com 80 neurônios excitatórios (`P_E`) e 20 inibitórios (`P_I`). Há conexões recorrentes dentro da população E (`syn_E_E`), conexões de E para I (`syn_E_I`), e conexões inibitórias de I de volta para E (`syn_I_E`), implementando a inibição lateral. Um input externo (`PoissonGroup`) fornece excitação inicial para a rede E. Usamos sinapses baseadas em condutância para maior realismo, especialmente para a inibição. A simulação (executada por 1 segundo) deve revelar dinâmicas interessantes resultantes da interação E-I, como atividade oscilatória (ritmos gama são frequentemente gerados por tais circuitos) ou competição, visíveis tanto nos raster plots quanto nas taxas de disparo populacionais.

**Figura 5.4**: Atividade da Rede Recorrente E-I Simulada. (a) Raster plot gerado pelo código acima, mostrando os spikes dos neurônios excitatórios (azul) e inibitórios (vermelho). Dependendo dos parâmetros, pode mostrar atividade síncrona, oscilatória ou assíncrona irregular. (b) Taxa de disparo populacional das populações E (azul) e I (vermelho) ao longo do tempo, revelando flutuações globais na atividade da rede e a relação temporal entre excitação e inibição.

Estes dois exemplos demonstram como o Brian2 pode ser usado para construir e simular SNNs com diferentes topologias (feedforward e recorrente) e motivos de circuito (inibição lateral), permitindo a exploração de como a arquitetura da rede e as interações sinápticas moldam a dinâmica neural coletiva.

**5.5. Visualização de Atividade de Rede: Raster Plots e Taxa de Disparo Populacional**

A simulação de SNNs, especialmente as recorrentes ou de larga escala, pode gerar dinâmicas complexas. Para compreender o comportamento da rede e analisar os resultados da simulação, técnicas de visualização eficazes são indispensáveis. Brian2, através de seus objetos Monitores (`SpikeMonitor`, `StateMonitor`, `PopulationRateMonitor`) e em conjunto com bibliotecas de plotagem como Matplotlib, facilita a geração de visualizações padrão da atividade da rede. Duas das visualizações mais comuns e úteis são os raster plots e os gráficos de taxa de disparo populacional.

*   **Raster Plots (Gráficos de Espalhamento de Spikes):**
    *   **O que mostram:** Um raster plot visualiza os tempos de disparo de múltiplos neurônios ao longo do tempo. Tipicamente, o eixo horizontal representa o tempo da simulação, e o eixo vertical representa o índice (ou ID) de cada neurônio na população monitorada. Um ponto (ou pequeno traço vertical) é plotado na coordenada $(t, i)$ sempre que o neurônio $i$ dispara um spike no tempo $t$.
    *   **Como gerar:** Utiliza-se os dados registrados por um `SpikeMonitor`. O array `SpikeMonitor.t` contém os tempos de todos os spikes, e o array `SpikeMonitor.i` contém os índices dos neurônios correspondentes. Uma plotagem de dispersão (`scatter`) ou de pontos (`plot` com marcador '.') de `t` versus `i` gera o raster plot.
        ```python
        # Exemplo de código Matplotlib para gerar raster plot a partir de spikemon
        plt.figure()
        plt.plot(spikemon.t / b2.ms, spikemon.i, '.k', markersize=2) # '.k' = pontos pretos
        plt.xlabel('Tempo (ms)')
        plt.ylabel('Índice do Neurônio')
        plt.title('Raster Plot da Atividade da População')
        # plt.xlim(...) # Definir limites de tempo, se necessário
        # plt.ylim(...) # Definir limites de índice, se necessário
        plt.show()
        ```
    *   **Interpretação:** Raster plots fornecem uma visão qualitativa poderosa da dinâmica populacional. Eles permitem identificar:
        *   *Níveis de Atividade:* Densidade de pontos indica a taxa de disparo geral.
        *   *Neurônios Ativos vs. Inativos:* Linhas horizontais com muitos pontos vs. pouquíssimos pontos.
        *   *Sincronia:* Alinhamentos verticais de pontos indicam disparos síncronos em múltiplos neurônios.
        *   *Oscilações:* Padrões repetitivos ou faixas verticais de atividade podem indicar oscilações na rede.
        *   *Propagação de Atividade:* Comparando raster plots de diferentes camadas (como no exemplo feedforward), pode-se ver o fluxo de atividade.
        *   *Padrões Específicos:* Sequências de disparo ou outros padrões temporais podem emergir.

*   **Taxa de Disparo Populacional (Population Firing Rate):**
    *   **O que mostra:** Fornece uma medida quantitativa da atividade geral de uma população neuronal ao longo do tempo. Representa a taxa média de disparo instantânea (ou quase instantânea) calculada sobre todos os neurônios da população.
    *   **Como calcular/gerar:** Existem duas abordagens principais:
        1.  **Usando `PopulationRateMonitor`:** Este monitor do Brian2 calcula automaticamente a taxa de disparo suavizada ao longo do tempo (usando um kernel de suavização, como uma janela deslizante).
            ```python
            # Criar e usar PopulationRateMonitor
            ratemon = b2.PopulationRateMonitor(neuron_group)
            # ... (adicionar à rede e executar) ...
            # Plotar
            plt.figure()
            plt.plot(ratemon.t / b2.ms, ratemon.rate / b2.Hz)
            plt.xlabel('Tempo (ms)')
            plt.ylabel('Taxa de Disparo Populacional (Hz)')
            plt.title('Atividade Média da População')
            plt.show()
            ```
        2.  **Calculando a partir do `SpikeMonitor`:** Pode-se calcular manualmente a taxa de disparo binando os tempos de spike (`SpikeMonitor.t`) em intervalos de tempo curtos ($\Delta t_{bin}$) e dividindo a contagem de spikes em cada bin pelo número de neurônios na população ($N$) e pela largura do bin ($\Delta t_{bin}$). Isso resulta em um histograma temporal da taxa de disparo.
            ```python
            # Exemplo de cálculo manual da taxa a partir do SpikeMonitor
            bin_width = 5 * b2.ms # Largura do bin de tempo
            # Criar bins de tempo
            time_bins = np.arange(0, (simulation_duration/b2.ms) + (bin_width/b2.ms), bin_width/b2.ms) * b2.ms
            # Contar spikes em cada bin (requer NumPy)
            spike_counts, _ = np.histogram(spikemon.t, bins=time_bins)
            # Calcular taxa (spikes / neuronio / segundo)
            population_rate = spike_counts / (N_population * bin_width) # N_population é o tamanho do grupo
            # Obter centros dos bins para plotagem
            bin_centers = (time_bins[:-1] + time_bins[1:]) / 2

            # Plotar
            plt.figure()
            plt.plot(bin_centers / b2.ms, population_rate / b2.Hz)
            plt.xlabel('Tempo (ms)')
            plt.ylabel('Taxa de Disparo Populacional (Hz)')
            plt.title('Atividade Média da População (Binada)')
            plt.show()
            ```
    *   **Interpretação:** Gráficos de taxa de disparo populacional mostram a evolução temporal da atividade global da rede. São úteis para:
        *   Verificar a estabilidade da rede (taxas muito altas ou zero podem indicar problemas).
        *   Detectar oscilações globais (picos e vales periódicos na taxa).
        *   Observar respostas transientes a estímulos.
        *   Comparar os níveis de atividade entre diferentes populações (e.g., excitatória vs. inibitória, como no exemplo recorrente).

*   **Outras Visualizações:**
    *   **Gráficos de Variáveis de Estado:** Plotar o traço temporal de variáveis contínuas (`StateMonitor`) como potencial de membrana, correntes de adaptação, condutâncias sinápticas, etc., para neurônios individuais ou selecionados. Útil para depurar modelos e entender a dinâmica sub-limiar e os mecanismos de disparo (como visto nos exemplos de código).
    *   **Histogramas de Intervalo Inter-Spike (ISI):** Plotar a distribuição dos intervalos de tempo entre spikes consecutivos de um neurônio ou população. Revela padrões de disparo (regular, irregular, bursting).
    *   **Análise de Correlação:** Calcular correlações cruzadas entre os trens de spikes de pares de neurônios para investigar relações funcionais e sincronia precisa.
    *   **Visualização de Pesos Sinápticos:** Em simulações com plasticidade, plotar a matriz de pesos ou a distribuição de pesos ao longo do tempo para visualizar o aprendizado.

**Figura 5.5**: Exemplos de Visualização de Atividade de Rede. (a) Raster plot mostrando atividade oscilatória sincronizada. (b) Gráfico de taxa de disparo populacional correspondente, mostrando picos periódicos. (c) Gráfico de potencial de membrana ($V_m$) de um neurônio individual exibindo bursting. (d) Histograma de ISI mostrando distribuição bimodal típica de bursting.

A combinação dessas técnicas de visualização permite uma exploração aprofundada do comportamento dinâmico das SNNs simuladas em Brian2, fornecendo insights cruciais sobre como a estrutura da rede e a dinâmica dos componentes individuais interagem para gerar computação em nível de sistema. Estas ferramentas serão essenciais nos capítulos subsequentes ao investigarmos o aprendizado e a plasticidade em SNNs.

---

## REFERÊNCIAS BIBLIOGRÁFICAS

Aubon, C., & Rotter, S. (2023). Network specialization for reliability and redundancy of information transmission in parallel sensory pathways. *PLoS Computational Biology*, *19*(3), e1010945. https://doi.org/10.1371/journal.pcbi.1010945
    *   *Resumo:* Modela e simula (usando princípios aplicáveis em Brian2) como diferentes topologias de rede em vias sensoriais paralelas podem ser otimizadas para diferentes aspectos da transmissão de informação, como confiabilidade ou representação redundante. Relevante para a discussão de topologias feedforward e suas propriedades.

Cavallari, S., Hayashida, K., & Falotico, E. (2021). Benchmarking trade-offs between biological plausibility, computational cost, and accuracy of spiking neuron models on neuromorphic hardware. *Frontiers in Neuroscience*, *15*, 661110. https://doi.org/10.3389/fnins.2021.661110
    *   *Resumo:* Embora foque em modelos, a avaliação em tarefas de classificação implica a construção de redes (provavelmente feedforward) com esses modelos. Contextualiza a aplicação das redes construídas em Brian2 para tarefas funcionais e hardware.

Galán, R. F. (2021). Open-source software tools for the simulation of neuronal dynamics and large-scale brain networks. *Journal of Experimental Neuroscience*, *15*, 11790695211032484. https://doi.org/10.1177/11790695211032484
    *   *Resumo:* Revisa ferramentas de software livre para simulação neural, incluindo Brian2. Discute a importância dessas ferramentas para construir e simular redes de larga escala, o foco deste capítulo. Fornece um panorama das capacidades dessas ferramentas.

Isbister, J. B., Ruth, L. H., & Tripathy, S. J. (2021). Reconciling functional diversity and transcriptomic identity of cortical GABAergic interneurons. *eLife*, *10*, e68667. https://doi.org/10.7554/eLife.68667
    *   *Resumo:* Discute a diversidade funcional de interneurônios inibitórios no córtex, crucial para a dinâmica de redes recorrentes como a do exemplo E-I. Embora focado na biologia, informa a necessidade de modelar diferentes tipos de neurônios e suas conectividades específicas, algo facilitado por Brian2.

Javadzadeh, A., & Towfighian, S. (2022). Dynamics and energy consumption analysis of coupled bursting neurons considering electromagnetic induction. *Communications in Nonlinear Science and Numerical Simulation*, *113*, 106560. https://doi.org/10.1016/j.cnsns.2022.106560
    *   *Resumo:* Analisa a dinâmica de redes de neurônios (usando modelos que poderiam ser implementados em Brian2), focando em sincronização e bursting. Relevante para a compreensão da dinâmica complexa que pode emergir em redes recorrentes.

Knight, J. C., & Nowotny, T. (2021). Larger networks learn better pathways in reservoir computing. *Nature Communications*, *12*(1), 3750. https://doi.org/10.1038/s41467-021-23799-8
    *   *Resumo:* Investiga propriedades de redes recorrentes aleatórias (reservatórios), um tipo de topologia mencionado na Seção 5.1. Mostra como o tamanho e a estrutura da rede (simulável em Brian2) afetam sua capacidade computacional.

Petrovici, M. A., Schmitt, S., & Bellec, G. (2020). Neuromorphic computation: Challenges and solutions. *Frontiers in Neuroscience*, *14*, 801. https://doi.org/10.3389/fnins.2020.00801
    *   *Resumo:* Discute desafios em computação neuromórfica, incluindo a construção e simulação de redes eficientes. Contextualiza a necessidade de ferramentas como Brian2 e as técnicas de construção de rede aqui descritas para abordar esses desafios.

Stimberg, M., Brette, R., & Goodman, D. F. M. (2019a). Brian 2, an intuitive and efficient neural simulator. *eLife*, *8*, e47314. https://doi.org/10.7554/eLife.47314
    *   *Resumo:* O artigo principal do Brian2, descrevendo os objetos `NeuronGroup` e `Synapses`, incluindo o método `connect()`, que são centrais para este capítulo. Valida a eficiência na criação de conexões e simulação de redes.

Sun, C., Wu, D., Ma, J., & Yao, Z. (2021). Pattern transition and its mechanism for Newman–Watts small-world networks coupled by gap junctions. *Applied Mathematics and Computation*, *404*, 126218. https://doi.org/10.1016/j.amc.2021.126218
    *   *Resumo:* Estuda a dinâmica (transições de padrões de atividade) em redes com topologias específicas (mundo pequeno) e um tipo de conexão (junções comunicantes/elétricas). Embora o tipo de sinapse seja diferente, ilustra como a topologia da rede (além da aleatória ou totalmente regular) influencia a dinâmica populacional, um tema central do capítulo.

Voegtlin, T. (2021). Computing with spiking neural networks. *Biological Cybernetics*, *115*(4-5), 387-399. https://doi.org/10.1007/s00422-021-00885-w
    *   *Resumo:* Revisa diferentes paradigmas para realizar computação com SNNs, incluindo computação em reservatório e redes feedforward treinadas. Contextualiza as arquiteturas de rede discutidas neste capítulo em termos de suas potenciais aplicações computacionais.
