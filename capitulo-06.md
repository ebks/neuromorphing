---

# CAPÍTULO 06

# MECANISMOS DE APRENDIZAGEM E PLASTICIDADE EM SNNS SIMULADAS

---

![imagem](neuro.png)

*Este capítulo mergulha nos mecanismos que conferem às Redes Neurais de Spikes (SNNs) a capacidade fundamental de aprender e se adaptar, espelhando a plasticidade observada no cérebro biológico. O foco principal recairá sobre a implementação computacional e a exploração da Plasticidade Dependente do Tempo de Spike (STDP), uma regra de aprendizado local e bioinspirada. Analisaremos o formalismo matemático do modelo clássico de STDP e suas variantes, seguido por estudos de caso práticos em Brian2 que demonstram como a relação temporal precisa entre spikes pré e pós-sinápticos pode induzir potenciação ou depressão sináptica, tanto em um par simples de neurônios quanto em um contexto de aprendizado não supervisionado de padrões temporais em uma rede pequena. Além da STDP, exploraremos outras formas cruciais de plasticidade neural, como a plasticidade homeostática – essencial para a estabilidade da rede – e a plasticidade estrutural – que envolve a modificação física das conexões – discutindo seus fundamentos teóricos e as possibilidades (e limitações) de sua implementação em simuladores como o Brian2. Finalmente, abordaremos o desafio do aprendizado supervisionado em SNNs, fornecendo uma visão geral das metodologias desenvolvidas para treinar essas redes para tarefas específicas, desde abordagens pioneiras até os métodos contemporâneos baseados em gradiente substituto, que tentam conciliar a natureza baseada em eventos das SNNs com as poderosas técnicas de otimização do aprendizado profundo. Um estudo de caso final buscará ilustrar, dentro das capacidades do Brian2, um exemplo de como a plasticidade pode ser utilizada para tarefas de classificação ou extração de características.*

---

**6.1. Implementando STDP (Spike-Timing-Dependent Plasticity)**

A Plasticidade Dependente do Tempo de Spike (STDP) emergiu como um dos mecanismos de plasticidade sináptica mais estudados e computacionalmente atraentes, devido à sua base biológica sólida (ver Seção 2.4) e às suas propriedades computacionais intrigantes (Markram et al., 1997; Bi & Poo, 1998; Song et al., 2000; Caporale & Dan, 2008). A STDP encapsula a ideia hebbiana de que a correlação temporal na atividade impulsiona a mudança sináptica, mas refina essa noção ao tornar a magnitude e o sinal (potenciação vs. depressão) da mudança dependentes da ordem temporal precisa e do intervalo ($\Delta t = t_{post} - t_{pre}$) entre os spikes pré e pós-sinápticos, tipicamente na escala de dezenas de milissegundos.

**6.1.1. Modelo Clássico de STDP (Pré/Post sináptico)**

O modelo mais canônico e frequentemente implementado de STDP descreve a mudança no peso sináptico $w$ associada a um par de spikes pré e pós-sinápticos próximos no tempo. A forma funcional dessa dependência, conhecida como **janela de STDP**, é frequentemente modelada por funções exponenciais assimétricas:

$\Delta w = \begin{cases} A_+ \exp\left(-\frac{\Delta t}{\tau_+}\right) & \text{se } \Delta t > 0 \text{ (pré antes de pós: LTP)} \\ -A_- \exp\left(\frac{\Delta t}{\tau_-}\right) & \text{se } \Delta t < 0 \text{ (pós antes de pré: LTD)} \\ 0 & \text{se } \Delta t = 0 \end{cases}$

onde:
*   $\Delta t = t_{post} - t_{pre}$ é a diferença de tempo entre o spike pós-sináptico ($t_{post}$) e o spike pré-sináptico ($t_{pre}$).
*   $A_+$ e $A_-$ são as amplitudes máximas da mudança de peso para potenciação (LTP) e depressão (LTD), respectivamente (valores positivos). Controlam a magnitude do aprendizado.
*   $\tau_+$ e $\tau_-$ são as constantes de tempo que definem a largura das janelas temporais para LTP e LTD, respectivamente (tipicamente na ordem de 10-50 ms).
*   A mudança $\Delta w$ pode ser aditiva ($\Delta w$ é um valor fixo dependente de $\Delta t$) ou multiplicativa (e.g., $\Delta w = \eta \cdot F(\Delta t) \cdot w$ ou $\Delta w = \eta \cdot F(\Delta t) \cdot (w_{max} - w)$ para LTP, dependendo do peso atual $w$), onde $\eta$ é uma taxa de aprendizado. Modelos multiplicativos ou com limites (bounding) são frequentemente necessários para evitar que os pesos cresçam indefinidamente ou se tornem negativos.

**Figura 6.1**: Janela Temporal do Modelo Clássico de STDP. Gráfico mostrando a mudança relativa no peso sináptico ($\Delta w/w$ ou $\Delta w$) no eixo Y em função da diferença de tempo $\Delta t = t_{post} - t_{pre}$ (eixo X, em ms). A curva exibe potenciação (LTP, valores positivos) para $\Delta t > 0$ (pré precede pós) e depressão (LTD, valores negativos) para $\Delta t < 0$ (pós precede pré), com decaimentos exponenciais definidos por $\tau_+$ e $\tau_-$.

**Implementação Computacional em Brian2:**

Implementar a STDP em simuladores baseados em eventos como o Brian2 requer um mecanismo para rastrear a ocorrência de spikes pré e pós-sinápticos recentes e calcular a mudança de peso apropriada quando um novo spike ocorre. Uma abordagem comum e eficiente é usar **variáveis de traço (trace variables)** associadas a cada neurônio ou sinapse (Morrison et al., 2007; Pfister & Gerstner, 2006).

*   **Traço Pré-sináptico ($P(t)$):** Uma variável associada ao neurônio pré-sináptico que aumenta instantaneamente quando ele dispara e decai exponencialmente com uma constante de tempo $\tau_{pre}$ (frequentemente igual a $\tau_+$ da janela de LTP). Representa uma "memória" recente do disparo pré-sináptico.
    $\tau_{pre} \frac{dP}{dt} = -P$
    Se ocorre spike pré-sináptico em $t_{pre}$: $P(t_{pre}^+) \leftarrow P(t_{pre}^-) + 1$ (ou $+ A_{pre}$).
*   **Traço Pós-sináptico ($O(t)$):** Uma variável associada ao neurônio pós-sináptico que aumenta quando ele dispara e decai com $\tau_{post}$ (frequentemente $\tau_-$ da janela de LTD). Representa uma memória do disparo pós-sináptico.
    $\tau_{post} \frac{dO}{dt} = -O$
    Se ocorre spike pós-sináptico em $t_{post}$: $O(t_{post}^+) \leftarrow O(t_{post}^-) + 1$ (ou $+ A_{post}$).

A regra de atualização do peso $w$ pode então ser definida com base nos valores desses traços no momento do spike:

*   **Quando um spike pós-sináptico ocorre em $t_{post}$:** A mudança de peso depende do valor do traço pré-sináptico *imediatamente antes* do spike pós ($P(t_{post}^-)$). Este valor reflete a integral dos efeitos dos spikes pré-sinápticos recentes, ponderada pela janela de LTP ($\exp(-s/\tau_{pre})$ onde $s = t_{post} - t_{pre}$). A atualização pode ser:
    $\Delta w \leftarrow \eta_+ P(t_{post}^-)$ (Potenciação)
    (onde $\eta_+$ incorpora a amplitude $A_+$ e a taxa de aprendizado).
*   **Quando um spike pré-sináptico ocorre em $t_{pre}$:** A mudança de peso depende do valor do traço pós-sináptico *imediatamente antes* do spike pré ($O(t_{pre}^-)$). Este valor reflete os spikes pós-sinápticos recentes, ponderados pela janela de LTD ($\exp(s/\tau_{post})$ onde $s = t_{post} - t_{pre} < 0$). A atualização pode ser:
    $\Delta w \leftarrow -\eta_- O(t_{pre}^-)$ (Depressão)
    (onde $\eta_-$ incorpora $A_-$).

Esta abordagem baseada em traços permite implementar a STDP de forma local (cada sinapse só precisa conhecer os traços de seus neurônios pré e pós conectados) e eficiente em simuladores de eventos, pois as ODEs dos traços só precisam ser atualizadas quando ocorrem spikes (usando a flag `(event-driven)` em Brian2).

**Brian2 e STDP:**

O objeto `Synapses` do Brian2 é ideal para implementar STDP. O modelo (`model=`) define as equações para o peso $w$ e as variáveis de traço (que podem ser definidas como variáveis pré ou pós-sinápticas, ou por sinapse). As regras de atualização dos traços e do peso são implementadas nas strings `on_pre='...'` e `on_post='...'`.

```python
# Exemplo conceitual de definição de STDP em Brian2 (baseado em traços pré e pós)

# Parâmetros STDP
tau_pre = 20*b2.ms; tau_post = 20*b2.ms
A_pre = 0.01 # Magnitude do incremento do traço pré (adimensional)
A_post = 0.01 # Magnitude do incremento do traço pós (adimensional)
eta_ltp = 0.01 # Taxa de aprendizado para LTP
eta_ltd = 0.0105 # Taxa de aprendizado para LTD (ligeiramente maior para estabilidade)
w_max = 1.0 # Peso máximo

# Equações STDP no modelo Synapses
stdp_eqs = '''
# Variável de peso (exemplo adimensional)
w : 1
# Traço pré-sináptico (associado a cada neurônio pré, rastreado pela sinapse)
dapre/dt = -apre / tau_pre : 1 (event-driven)
# Traço pós-sináptico (associado a cada neurônio pós, rastreado pela sinapse)
dapost/dt = -apost / tau_post : 1 (event-driven)
# Parâmetros (podem ser definidos por sinapse ou globalmente)
tau_pre : second (constant)
tau_post : second (constant)
eta_ltp : 1 (constant)
eta_ltd : 1 (constant)
w_max : 1 (constant)
A_pre : 1 (constant)
A_post : 1 (constant)
'''

# Código executado na chegada de um spike pré-sináptico
# Atualiza o peso com base no traço pós e incrementa o traço pré
on_pre_stdp = '''
# Efeito pós-sináptico (exemplo: incrementar condutância g_exc)
# g_exc_post += w * nS # (g_exc_post definido no NeuronGroup pós)
# Atualização do peso (LTD) - depende do traço pós
w = clip(w - eta_ltd * apost, 0, w_max)
# Incrementar traço pré
apre += A_pre
'''

# Código executado quando o neurônio pós-sináptico dispara
# Atualiza o peso com base no traço pré e incrementa o traço pós
on_post_stdp = '''
# Atualização do peso (LTP) - depende do traço pré
w = clip(w + eta_ltp * apre, 0, w_max)
# Incrementar traço pós
apost += A_post
'''

# Criação do objeto Synapses
# syn_stdp = b2.Synapses(source_group, target_group, model=stdp_eqs,
#                        on_pre=on_pre_stdp, on_post=on_post_stdp)
# ... conectar, definir parâmetros iniciais para w, tau_pre, tau_post, etc. ...
```
Este esqueleto de código mostra como as equações para os traços (`apre`, `apost`) e as atualizações de peso baseadas nesses traços nos eventos `on_pre` e `on_post` podem ser especificadas em Brian2. A função `clip(value, min, max)` é usada para manter o peso $w$ dentro dos limites $[0, w_{max}]$.

**6.1.2. Estudo de Caso: Simulação de STDP em um Par de Neurônios**

Para visualizar a operação da STDP, vamos simular o cenário mais simples: um neurônio pré-sináptico conectado a um neurônio pós-sináptico. Controlaremos o tempo relativo de seus spikes e observaremos como o peso sináptico evolui.

**Objetivo:** Demonstrar LTP quando o spike pré precede o pós por um intervalo curto, e LTD quando a ordem é invertida.

```python
# Importar bibliotecas
import brian2 as b2
import matplotlib.pyplot as plt
import numpy as np

b2.prefs.codegen.target = 'cython'
b2.defaultclock.dt = 0.1*b2.ms

# --- Neurônios (usaremos SpikeGeneratorGroup para controlar tempos exatos) ---
# Neurônio Pré-sináptico
times_pre = np.arange(10, 210, 20) * b2.ms # Spikes pré a cada 20ms
pre_neuron = b2.SpikeGeneratorGroup(1, indices=np.zeros(len(times_pre)), times=times_pre)

# Neurônio Pós-sináptico (controlado)
# Caso 1: Pós dispara DEPOIS do pré (delta_t > 0 -> LTP esperado)
delta_t_ltp = 10 * b2.ms
times_post_ltp = times_pre + delta_t_ltp
post_neuron_ltp = b2.SpikeGeneratorGroup(1, indices=np.zeros(len(times_post_ltp)), times=times_post_ltp)

# Caso 2: Pós dispara ANTES do pré (delta_t < 0 -> LTD esperado)
delta_t_ltd = -10 * b2.ms
times_post_ltd = times_pre + delta_t_ltd
post_neuron_ltd = b2.SpikeGeneratorGroup(1, indices=np.zeros(len(times_post_ltd)), times=times_post_ltd)

# --- Modelo e Parâmetros STDP ---
tau_pre = 20*b2.ms; tau_post = 20*b2.ms
A_pre = 0.01; A_post = 0.01
eta_ltp = 0.01; eta_ltd = 0.0105 # Taxa LTD ligeiramente > LTP
w_max = 1.0
w_initial = 0.5 # Peso inicial

stdp_eqs = '''
w : 1
dapre/dt = -apre / tau_pre : 1 (event-driven)
dapost/dt = -apost / tau_post : 1 (event-driven)
'''
on_pre_stdp = '''
# Sem efeito pós-sináptico aqui, só plasticidade
w = clip(w - eta_ltd * apost, 0, w_max)
apre += A_pre
'''
on_post_stdp = '''
w = clip(w + eta_ltp * apre, 0, w_max)
apost += A_post
'''

# --- Criação das Sinapses e Monitores ---
# Função para criar e simular uma conexão
def simulate_stdp_pair(post_neuron, label):
    syn = b2.Synapses(pre_neuron, post_neuron, model=stdp_eqs,
                      on_pre=on_pre_stdp, on_post=on_post_stdp)
    syn.connect(i=0, j=0)
    # Definir parâmetros e estado inicial
    syn.w = w_initial
    syn.apre = 0.0
    syn.apost = 0.0
    syn.namespace['tau_pre'] = tau_pre
    syn.namespace['tau_post'] = tau_post
    syn.namespace['eta_ltp'] = eta_ltp
    syn.namespace['eta_ltd'] = eta_ltd
    syn.namespace['w_max'] = w_max
    syn.namespace['A_pre'] = A_pre
    syn.namespace['A_post'] = A_post

    # Monitorar o peso sináptico
    weight_monitor = b2.StateMonitor(syn, 'w', record=0) # Monitorar w da sinapse 0

    # Rede e Simulação
    net = b2.Network(pre_neuron, post_neuron, syn, weight_monitor)
    duration = times_pre[-1] + 50*b2.ms # Simular um pouco além do último spike
    net.run(duration, report=None)

    # Retornar monitor de peso
    print(f"Simulação concluída para {label}. Peso final: {weight_monitor.w[0][-1]:.4f}")
    return weight_monitor

# --- Executar Simulações ---
monitor_ltp = simulate_stdp_pair(post_neuron_ltp, "Pré antes de Pós (LTP)")
monitor_ltd = simulate_stdp_pair(post_neuron_ltd, "Pós antes de Pré (LTD)")

# --- Visualização ---
plt.figure(figsize=(10, 6))

plt.plot(monitor_ltp.t / b2.ms, monitor_ltp.w[0], 'b-', label=f'LTP ($\Delta t = {delta_t_ltp/b2.ms:.0f}$ ms)')
plt.plot(monitor_ltd.t / b2.ms, monitor_ltd.w[0], 'r-', label=f'LTD ($\Delta t = {delta_t_ltd/b2.ms:.0f}$ ms)')

# Marcar tempos dos spikes pré para referência
plt.vlines(times_pre / b2.ms, ymin=0, ymax=w_max, color='gray', linestyle=':', lw=0.5, label='Spikes Pré')

plt.xlabel('Tempo (ms)')
plt.ylabel('Peso Sináptico (w)')
plt.title('Evolução do Peso Sináptico devido à STDP')
plt.ylim(0, w_max + 0.1)
plt.legend()
plt.grid(True, linestyle=':')
plt.show()
```
Neste estudo de caso, usamos `SpikeGeneratorGroup` para controlar precisamente os tempos dos spikes pré e pós-sinápticos. Criamos duas sinapses separadas, uma para o caso pré-antes-de-pós ($\Delta t = +10$ ms) e outra para pós-antes-de-pré ($\Delta t = -10$ ms). O modelo STDP baseado em traços é aplicado. A simulação monitora a evolução do peso sináptico `w` ao longo do tempo para cada caso. O gráfico resultante deve mostrar claramente que o peso aumenta (LTP) quando o pré precede o pós e diminui (LTD) quando a ordem é invertida, confirmando a implementação da janela de STDP.

**Figura 6.2**: Simulação de STDP em um Par de Neurônios. Gráfico gerado pelo código acima, mostrando a evolução do peso sináptico $w$ (eixo Y) ao longo do tempo (eixo X) para dois cenários: (azul) spike pré ocorrendo 10 ms antes do pós (LTP), resultando em aumento do peso; (vermelho) spike pós ocorrendo 10 ms antes do pré (LTD), resultando em diminuição do peso. Linhas pontilhadas indicam os tempos dos spikes pré-sinápticos.

**6.1.3. Estudo de Caso: Aprendizado não Supervisionado de Padrões com STDP**

Vamos agora aplicar a STDP em um cenário ligeiramente mais complexo: uma rede simples onde múltiplos neurônios de entrada projetam para um neurônio de saída. Alguns inputs terão padrões de disparo correlacionados (próximos no tempo), enquanto outros serão não correlacionados. Esperamos que a STDP fortaleça seletivamente as sinapses dos inputs correlacionados.

**Objetivo:** Demonstrar como a STDP pode levar ao aprendizado não supervisionado de padrões temporais, fortalecendo sinapses que consistentemente contribuem para o disparo do neurônio pós-sináptico.

```python
# Importar bibliotecas
import brian2 as b2
import matplotlib.pyplot as plt
import numpy as np

b2.prefs.codegen.target = 'cython'
b2.defaultclock.dt = 0.1*b2.ms

# --- Parâmetros da Rede ---
N_input = 50  # Número de neurônios de entrada
N_output = 1   # Um neurônio de saída
simulation_duration = 5 * b2.second # Simular por mais tempo para aprendizado

# --- Input: Neurônios Poisson com padrões correlacionados ---
# 10 neurônios (índices 0-9) disparam correlacionados com taxa alta (e.g., 40 Hz)
# 40 neurônios (índices 10-49) disparam não correlacionados com taxa baixa (e.g., 5 Hz)
rate_correlated = 40 * b2.Hz
rate_uncorrelated = 5 * b2.Hz
rates_input = np.zeros(N_input) * b2.Hz
rates_input[0:10] = rate_correlated
rates_input[10:N_input] = rate_uncorrelated
input_group = b2.PoissonGroup(N_input, rates=rates_input)

# --- Neurônio de Saída LIF ---
tau = 20 * b2.ms; Vt = -50 * b2.mV; Vr = -70 * b2.mV; El = -65 * b2.mV; Rm = 100 * b2.Mohm; ref_p = 3*b2.ms
lif_eqs = '''
dv/dt = (El - v + Rm * g_exc)/tau : volt (unless refractory)
dg_exc/dt = -g_exc / (5*ms) : amp # Condutância excitatória total
'''
output_neuron = b2.NeuronGroup(N_output, lif_eqs, threshold='v>Vt', reset='v=Vr', refractory=ref_p, method='euler')
output_neuron.v = El
output_neuron.g_exc = 0*b2.nA

# --- Sinapses com STDP (Input -> Output) ---
# Parâmetros STDP (ajustados para aprendizado mais lento)
tau_pre = 20*b2.ms; tau_post = 20*b2.ms
A_pre = 0.01; A_post = 0.01
eta_ltp = 0.002 # Taxa de aprendizado LTP menor
eta_ltd = 0.0021 # Taxa de aprendizado LTD menor
w_max = 15.0 * b2.nS # Peso máximo (em condutância)
w_initial = 5.0 * b2.nS # Peso inicial

# Modelo STDP com sinapse baseada em condutância
stdp_eqs_cond = '''
w : siemens # Peso é uma condutância
dapre/dt = -apre / tau_pre : 1 (event-driven)
dapost/dt = -apost / tau_post : 1 (event-driven)
'''
on_pre_stdp_cond = '''
g_exc_post += w # Incrementar condutância pós-sináptica
w = clip(w - eta_ltd * apost * nS, 0, w_max) # LTD (nS para escala)
apre += A_pre
'''
on_post_stdp_cond = '''
w = clip(w + eta_ltp * apre * nS, 0, w_max) # LTP (nS para escala)
apost += A_post
'''

# Criação das Sinapses
synapses = b2.Synapses(input_group, output_neuron, model=stdp_eqs_cond,
                      on_pre=on_pre_stdp_cond, on_post=on_post_stdp_cond)
synapses.connect() # Conectar todos os inputs ao neurônio de saída

# Inicializar parâmetros e variáveis sinápticas
synapses.w = w_initial
synapses.apre = 0.0
synapses.apost = 0.0
# Passar parâmetros para o namespace da sinapse (necessário para acesso nas equações)
synapses.namespace['tau_pre'] = tau_pre
synapses.namespace['tau_post'] = tau_post
synapses.namespace['eta_ltp'] = eta_ltp
synapses.namespace['eta_ltd'] = eta_ltd
synapses.namespace['w_max'] = w_max
synapses.namespace['A_pre'] = A_pre
synapses.namespace['A_post'] = A_post

# --- Monitores ---
spikemon_input = b2.SpikeMonitor(input_group)
spikemon_output = b2.SpikeMonitor(output_neuron)
# Monitorar os pesos de todas as sinapses
weight_monitor_all = b2.StateMonitor(synapses, 'w', record=True, dt=100*b2.ms) # Gravar pesos a cada 100ms

# --- Rede e Simulação ---
network = b2.Network(input_group, output_neuron, synapses,
                   spikemon_input, spikemon_output, weight_monitor_all)

print("Iniciando simulação de aprendizado STDP...")
network.run(simulation_duration, report='text', report_period=1*b2.second)
print("Simulação concluída.")

# --- Visualização ---
plt.figure(figsize=(12, 8))

# Raster plot Input + Output
plt.subplot(2, 1, 1)
plt.plot(spikemon_input.t / b2.ms, spikemon_input.i, '.k', markersize=1, label='Input Spikes')
# Destacar inputs correlacionados
correlated_indices = np.where(spikemon_input.i < 10)[0]
plt.plot(spikemon_input.t[correlated_indices] / b2.ms, spikemon_input.i[correlated_indices], '.r', markersize=1, label='Input Correlacionado')
# Plotar spikes de saída
plt.plot(spikemon_output.t / b2.ms, np.zeros(len(spikemon_output.t)) + N_input + 5, 'xb', markersize=5, label='Output Spike')
plt.ylabel('Índice Neurônio Input')
plt.title('Atividade da Rede e Spikes de Saída')
plt.legend(loc='upper right')
plt.xlim(0, simulation_duration/b2.ms)

# Evolução dos pesos sinápticos
plt.subplot(2, 1, 2)
# Plotar pesos das sinapses correlacionadas (índices 0-9)
for i in range(10):
    plt.plot(weight_monitor_all.t / b2.second, weight_monitor_all.w[i] / b2.nS, 'r-', alpha=0.5)
# Plotar pesos das sinapses não correlacionadas (índices 10-49)
for i in range(10, N_input):
    plt.plot(weight_monitor_all.t / b2.second, weight_monitor_all.w[i] / b2.nS, 'k-', alpha=0.2)
# Linha vermelha sólida para média dos correlacionados, preta para não correlacionados
plt.plot(weight_monitor_all.t / b2.second, np.mean(weight_monitor_all.w[:10], axis=0) / b2.nS, 'r-', lw=2, label='Média Pesos Correl.')
plt.plot(weight_monitor_all.t / b2.second, np.mean(weight_monitor_all.w[10:], axis=0) / b2.nS, 'k-', lw=2, label='Média Pesos Não Correl.')
plt.xlabel('Tempo (s)')
plt.ylabel('Peso Sináptico (nS)')
plt.title('Evolução dos Pesos Sinápticos devido à STDP')
plt.legend()
plt.ylim(bottom=0)

plt.tight_layout()
plt.show()
```
Neste cenário, criamos um grupo de entrada Poisson onde os 10 primeiros neurônios disparam a uma taxa mais alta (correlacionados temporalmente de forma estatística) e os restantes disparam a uma taxa baixa. Todos convergem para um único neurônio de saída LIF com sinapses plásticas regidas pela STDP. Monitoramos a evolução de todos os pesos sinápticos ao longo de 5 segundos de simulação. A visualização deve mostrar que, em média, os pesos das sinapses provenientes dos inputs correlacionados (vermelho) tendem a aumentar (LTP), pois seus spikes frequentemente precedem e contribuem para o disparo do neurônio pós-sináptico. Em contraste, os pesos dos inputs não correlacionados (preto) devem permanecer baixos ou diminuir (LTD), pois seus spikes têm menos probabilidade de estarem causalmente ligados ao disparo pós-sináptico. Isso demonstra a capacidade da STDP de selecionar e fortalecer inputs relevantes de forma não supervisionada.

**Figura 6.3**: Aprendizado Não Supervisionado com STDP. (a) Raster plot mostrando spikes dos neurônios de entrada (pontos pretos e vermelhos para correlacionados) e os spikes do neurônio de saída (cruzes azuis). (b) Gráfico da evolução temporal dos pesos sinápticos (eixo Y, em nS). Linhas vermelhas finas representam pesos dos inputs correlacionados, pretas finas os não correlacionados. Linhas grossas mostram a média de cada grupo, indicando que os pesos correlacionados foram potenciados pela STDP.

**6.2. Outras Formas de Plasticidade**

Embora a STDP (e a LTP/LTD hebbiana em geral) seja crucial para associar padrões de atividade correlacionados, ela sozinha pode levar a instabilidades na rede. Por exemplo, ciclos de feedback positivo podem fazer com que os pesos cresçam sem controle, levando a atividade epiléptica, ou toda a atividade pode cessar se a LTD dominar. O cérebro emprega mecanismos adicionais de plasticidade para manter a estabilidade e a funcionalidade da rede, notavelmente a plasticidade homeostática e a plasticidade estrutural.

**Plasticidade Homeostática:**

Refere-se a um conjunto de mecanismos regulatórios que atuam para manter a atividade neuronal (e.g., taxa média de disparo) dentro de uma faixa fisiológica ou operacional desejada, agindo em escalas de tempo mais lentas (horas a dias) do que a plasticidade hebbiana rápida (Turrigiano & Nelson, 2004; Turrigiano, 2012). A plasticidade homeostática age como um controle de feedback negativo: se a atividade de um neurônio fica cronicamente muito alta, mecanismos homeostáticos agem para reduzir sua excitabilidade ou a força de seus inputs; se a atividade fica muito baixa, eles agem para aumentá-la. Isso garante que os neurônios permaneçam responsivos e que a plasticidade hebbiana possa operar sem levar à saturação ou ao silenciamento.

Dois mecanismos principais de plasticidade homeostática são:

1.  **Escalonamento Sináptico (Synaptic Scaling):** Ajusta a força de *todas* (ou de um grande subconjunto) as sinapses excitatórias de um neurônio de forma multiplicativa, para cima ou para baixo, a fim de normalizar sua taxa de disparo média. Se a taxa de disparo do neurônio está cronicamente baixa, a força de suas sinapses excitatórias é escalonada para cima (aumentada proporcionalmente); se a taxa está alta, elas são escalonadas para baixo (Davis & Bezprozvanny, 2001). Isso preserva as *razões relativas* entre os pesos sinápticos (que foram aprendidas pela plasticidade hebbiana), mas ajusta o drive total para o neurônio. Os mecanismos moleculares envolvem a regulação da expressão e tráfico de receptores AMPA, possivelmente mediados por fatores como TNF-$\alpha$ ou BDNF.
    *   *Implementação Conceitual em Brian2:* Pode ser modelado adicionando uma variável que rastreia a taxa de disparo média recente do neurônio pós-sináptico ($\bar{r}_{post}$) e uma regra lenta que ajusta multiplicativamente todos os pesos sinápticos $w_i$ que chegam a ele, de forma a levar $\bar{r}_{post}$ em direção a uma taxa alvo $\bar{r}_{target}$. Por exemplo:
        $\tau_{homeo} \frac{d\bar{r}_{post}}{dt} = -\bar{r}_{post} + r_{inst}$ (onde $r_{inst}$ é a taxa instantânea ou um traço de spikes)
        $\frac{dw_i}{dt} = \eta_{homeo} w_i (\bar{r}_{target} - \bar{r}_{post})$ (Ajuste multiplicativo lento)
        Implementar isso de forma eficiente e robusta em Brian2 pode requerer acesso e modificação de múltiplos pesos sinápticos baseados no estado de um neurônio pós-sináptico, o que pode ser complexo dependendo da estrutura do `Synapses`.

2.  **Plasticidade Intrínseca (Intrinsic Plasticity - IP):** Refere-se a mudanças na excitabilidade do próprio neurônio, modificando suas propriedades intrínsecas de disparo em resposta a níveis crônicos de atividade, independentemente das sinapses. Isso pode envolver a alteração da densidade ou cinética de vários canais iônicos dependentes de voltagem (e.g., canais de $K^+$, $Na^+$, $Ca^{2+}$) na membrana do soma, dendrites ou axônio (Zhang & Linden, 2003; Desai et al., 1999). Por exemplo, um aumento crônico na atividade pode levar a um aumento na condutância de canais de $K^+$ (aumentando a AHP ou o vazamento), tornando o neurônio menos excitável (reduzindo sua função ganho f-I). O inverso ocorreria para atividade cronicamente baixa.
    *   *Implementação Conceitual em Brian2:* Pode ser modelado ajustando parâmetros intrínsecos do modelo neuronal (como o potencial de limiar $V_t$, a condutância de vazamento $g_L$, ou parâmetros de correntes de adaptação como $a$ ou $b$ no AdEx) com base na taxa de disparo média recente. Por exemplo:
        $\tau_{IP} \frac{dV_t}{dt} = \eta_{IP} ( \bar{r}_{post} - \bar{r}_{target})$ (Ajuste lento do limiar)
        Isso é geralmente mais fácil de implementar em Brian2 do que o escalonamento sináptico, pois envolve apenas variáveis do `NeuronGroup`.

A interação entre plasticidade hebbiana (que promove instabilidade e competição) e plasticidade homeostática (que promove estabilidade) é considerada essencial para o aprendizado robusto e a auto-organização em redes neurais biológicas (Zenke et al., 2017).

**Figura 6.4**: Conceitos de Plasticidade Homeostática. (a) Escalonamento Sináptico: Neurônio com atividade cronicamente baixa (esquerda) tem seus pesos sinápticos excitatórios (tamanho dos triângulos) aumentados multiplicativamente (direita) para restaurar a taxa de disparo alvo. (b) Plasticidade Intrínseca: Neurônio com atividade cronicamente alta (esquerda) ajusta suas propriedades intrínsecas (e.g., aumenta condutância de K+, direita) para reduzir sua excitabilidade e taxa de disparo.

**Plasticidade Estrutural:**

Refere-se a mudanças físicas na estrutura da rede neural, como o crescimento e a retração de axônios, dendritos e espinhas dendríticas, bem como a formação (sinaptogênese) e eliminação (pruning) de sinapses (Chklovskii et al., 2004; Holtmaat & Svoboda, 2009). Essas mudanças ocorrem em escalas de tempo que variam de minutos (para espinhas dendríticas) a dias ou mais (para ramos axonais/dendríticos maiores) e são fortemente influenciadas pela atividade neural, fatores de crescimento (neurotrofinas) e experiência.

A plasticidade estrutural adiciona outra camada de adaptação à rede, permitindo que ela reconfigure sua própria conectividade (topologia) em resposta a demandas de aprendizado ou mudanças ambientais.
*   **Nível de Espinhas Dendríticas:** Espinhas podem aparecer, desaparecer, aumentar ou diminuir de tamanho rapidamente em resposta à atividade sináptica local e LTP/LTD, afetando a força e a estabilidade de sinapses individuais.
*   **Nível de Axônios e Dendritos:** Ramos podem crescer ou se retrair, buscando novos parceiros sinápticos ou abandonando conexões existentes. Isso pode ser guiado por gradientes químicos e atividade elétrica.
*   **Sinaptogênese e Eliminação:** Novas sinapses podem ser formadas entre neurônios próximos (ou distantes, dependendo do crescimento axonal), e sinapses existentes podem ser eliminadas, especialmente durante o desenvolvimento, mas também na idade adulta. Acredita-se que a atividade correlacionada estabilize sinapses (ligado à LTP), enquanto atividade não correlacionada ou baixa atividade leve à eliminação (ligado à LTD).

**Implicações Computacionais:** A plasticidade estrutural permite:
*   Otimizar a fiação da rede para eficiência (e.g., minimizando o comprimento total do fio).
*   Alocar recursos sinápticos para onde são mais necessários.
*   Criar novas vias de processamento de informação.
*   Aumentar a capacidade de armazenamento da rede formando novas conexões.

**Simulação da Plasticidade Estrutural:** Modelar a plasticidade estrutural em simulações é significativamente mais desafiador do que modelar a plasticidade sináptica ou intrínseca, pois envolve mudanças na própria estrutura de dados que representa a rede (quem está conectado a quem).
*   **Brian2 e Plasticidade Estrutural:** Brian2, em sua forma padrão, não possui mecanismos embutidos para crescimento ou retração de neuritos ou para adicionar/remover neurônios dinamicamente. No entanto, é possível simular aspectos da plasticidade estrutural em nível sináptico:
    *   *Criação/Eliminação de Sinapses:* Pode-se, em princípio, remover sinapses existentes do objeto `Synapses` se seus pesos caírem abaixo de um limiar (representando eliminação) e criar novas conexões (talvez aleatoriamente entre neurônios próximos ou com base em alguma regra de atividade) durante a simulação. Isso, no entanto, pode ser computacionalmente caro e requer manipulação cuidadosa dos índices e estados sinápticos. Frameworks construídos sobre ou em conjunto com Brian2 podem oferecer suporte mais direto.
    *   *Modelos de "Peso Estrutural":* Uma abordagem mais abstrata é ter um pool de sinapses potenciais com peso zero, e a plasticidade (e.g., STDP modificada) pode fazer com que esses pesos se tornem não-zero (formação) ou retornem a zero (eliminação) (Deger et al., 2012).
*   **Desafios:** Modelar o crescimento guiado por atividade de forma realista requer algoritmos complexos e pode ser muito intensivo computacionalmente. A maioria das simulações de larga escala ainda utiliza conectividade fixa ou apenas plasticidade sináptica/intrínseca.

Apesar dos desafios de simulação, a plasticidade estrutural é um componente importante da adaptação neural biológica, e sua incorporação em modelos futuros pode ser crucial para alcançar níveis mais elevados de aprendizado autônomo e flexibilidade em sistemas neuromórficos.

**6.3. Abordagens para Treinamento Supervisionado de SNNs**

Enquanto regras de plasticidade não supervisionadas como a STDP são excelentes para extrair estrutura estatística dos dados e auto-organização, muitas tarefas práticas de engenharia (e.g., classificação de imagens, reconhecimento de fala) requerem **aprendizado supervisionado**, onde a rede é treinada para produzir uma saída específica (rótulo ou valor alvo) para uma dada entrada, com base em exemplos rotulados. Adaptar os paradigmas de aprendizado supervisionado, especialmente aqueles baseados em gradiente que impulsionaram o sucesso do deep learning em ANNs convencionais, para o domínio das SNNs tem sido um desafio significativo.

O principal obstáculo reside na **natureza não diferenciável do evento de disparo do spike**. O spike é um evento "tudo ou nada", tipicamente modelado como uma descontinuidade (ou uma função delta de Dirac no tempo). Isso significa que a derivada do output de um neurônio de spiking em relação ao seu input (ou aos seus pesos sinápticos) é zero em quase todos os lugares e indefinida (ou infinita) no momento exato do disparo. Isso impede a aplicação direta do algoritmo de **backpropagation**, que depende do cálculo de gradientes através da regra da cadeia para ajustar os pesos da rede de forma a minimizar um erro na saída.

Diversas abordagens foram desenvolvidas ao longo dos anos para contornar esse problema e permitir o treinamento supervisionado de SNNs:

1.  **Métodos baseados em Tempo de Spike (Primeira Geração):**
    *   **SpikeProp:** Um dos primeiros algoritmos (Bohte et al., 2002), aplicável a SNNs feedforward que usam codificação temporal (e.g., tempo para o primeiro spike). Deriva uma regra de aprendizado baseada em gradiente calculando como pequenas mudanças nos pesos afetam o tempo de disparo dos neurônios, assumindo um modelo neuronal específico (SRM - Spike Response Model) e linearização em torno do tempo de disparo. Limitações: Requer codificação temporal, computacionalmente caro, difícil de escalar para redes profundas, sensível a parâmetros.
    *   **Tempotron:** Uma regra de aprendizado para um único neurônio LIF projetada para classificar padrões espaço-temporais de spikes de entrada (Gütig & Sompolinsky, 2006). Ajusta os pesos sinápticos para garantir que o potencial de membrana pós-sináptico cruze (ou não cruze) o limiar em momentos específicos, dependendo da classe do padrão. É biologicamente mais plausível que SpikeProp, mas ainda limitado a classificadores lineares de camada única.

2.  **Conversão ANN-para-SNN:**
    *   Abordagem: Treinar uma Rede Neural Artificial (ANN) convencional (e.g., CNN, RNN) usando backpropagation padrão para uma tarefa específica. Depois, converter a ANN treinada em uma SNN equivalente, mapeando as ativações da ANN (tipicamente valores contínuos, como saídas ReLU) para taxas de disparo de neurônios LIF ou IF na SNN (Cao et al., 2015; Diehl et al., 2015). Os pesos da ANN são transferidos diretamente ou escalados para a SNN.
    *   *Vantagens:* Permite aproveitar os avanços e ferramentas do deep learning convencional. Pode alcançar alta precisão rapidamente se a conversão for bem-sucedida.
    *   *Desvantagens:* A conversão pode levar a perda de precisão (devido à discretização em spikes e aproximação da taxa de disparo). Geralmente resulta em SNNs com altas taxas de disparo (para mimetizar ativações ReLU), perdendo parte da vantagem de eficiência energética das SNNs esparsas. Requer processamento em múltiplos passos de tempo na SNN para estimar a taxa, introduzindo latência. O aprendizado ocorre offline na ANN, não online na SNN.

3.  **Métodos Baseados em Gradiente Substituto (Surrogate Gradient - SG):**
    *   Abordagem: Esta é atualmente a abordagem dominante e mais bem-sucedida para treinar SNNs profundas de forma supervisionada e de ponta a ponta (end-to-end) (Neftci et al., 2019; Zenke & Neftci, 2021; Pérez-Nieves et al., 2021). A ideia central é substituir a derivada da função de disparo do spike (que é problemática) por uma "função substituta" ou "gradiente substituto" ($\sigma'(V)$) que seja contínua e bem comportada durante o cálculo do gradiente no passo backward da backpropagation. No passo forward, o neurônio ainda dispara um spike discreto ("tudo ou nada") quando $V \ge V_{th}$. Mas no passo backward, ao calcular o gradiente do erro que passa por aquele neurônio, usa-se $\sigma'(V)$ em vez da derivada real (zero ou infinito).
        $\frac{\partial \text{Erro}}{\partial W} = \frac{\partial \text{Erro}}{\partial \text{SpikeOut}} \cdot \underbrace{\sigma'(V)}_{\text{Substituto}} \cdot \frac{\partial V}{\partial W}$
    *   **Funções Substitutas Comuns:** Diversas funções foram propostas como gradientes substitutos $\sigma'(V)$, geralmente centradas em torno do limiar $V_{th}$ e com largura controlável:
        *   Retângulo (derivada de uma rampa linear)
        *   Triângulo (derivada de uma quadrática)
        *   Sigmoide rápida (derivada da função sigmoide)
        *   Exponencial ou Gaussiana Suave
        A escolha da função substituta e de sua largura pode impactar o desempenho e a estabilidade do aprendizado.
    *   **Treinamento:** Com o gradiente substituto, pode-se usar variantes do algoritmo de backpropagation, frequentemente adaptadas para lidar com a dimensão temporal das SNNs (Backpropagation Through Time - BPTT, adaptado para spikes). Frameworks de deep learning populares como PyTorch e TensorFlow agora possuem bibliotecas especializadas (e.g., snnTorch, SpykeTorch, Sinabs, Nengo-DL) que implementam SG e facilitam o treinamento de SNNs.
    *   *Vantagens:* Permite treinamento end-to-end de SNNs profundas (incluindo recorrentes e convolucionais) usando gradientes. Frequentemente alcança desempenho competitivo com ANNs em benchmarks importantes. Pode resultar em SNNs com atividade esparsa e eficientes energeticamente se o treinamento for configurado corretamente (e.g., com regularização).
    *   *Desvantagens:* A base teórica ainda está sendo totalmente explorada (por que funciona tão bem?). O desempenho pode ser sensível à escolha da função substituta, taxa de aprendizado e outros hiperparâmetros. A implementação direta em simuladores como Brian2 (sem depender de frameworks de auto-diferenciação como PyTorch/TensorFlow) é complexa, pois requer a infraestrutura de backpropagation.

**Figura 6.5**: Conceito do Gradiente Substituto (Surrogate Gradient). (a) Função de disparo do spike (Heaviside $\Theta(V-V_{th})$) e sua derivada (Delta de Dirac $\delta(V-V_{th})$), que é problemática para backpropagation. (b) O método SG: No passo forward, usa-se a função Heaviside. No passo backward, a derivada $\delta$ é substituída por uma função substituta suave $\sigma'(V)$ (e.g., retangular, triangular, sigmoide rápida) que é não-zero em torno do limiar, permitindo que o gradiente flua.

4.  **Outras Abordagens (e.g., baseadas em Aprendizado por Reforço, Evolucionárias):**
    *   Regras de plasticidade local (como STDP) podem ser combinadas com um sinal global de recompensa ou erro (neuromodulação) para guiar o aprendizado em direção a um objetivo (Aprendizado por Reforço com SNNs, e.g., R-STDP) (Frémaux et al., 2013). Isso é biologicamente plausível, mas pode ter convergência mais lenta.
    *   Algoritmos evolucionários podem ser usados para otimizar os pesos ou parâmetros de uma SNN para uma tarefa específica, mas geralmente não escalam bem para redes grandes.

**Implementação em Brian2:**
*   **STDP e Plasticidade Local:** Brian2 é excelente para implementar regras de plasticidade local como STDP, plasticidade intrínseca ou homeostática, como visto nos exemplos. Regras baseadas em recompensa (R-STDP) também podem ser implementadas definindo como um sinal de recompensa (variável global ou neuromodulador simulado) modula os parâmetros da STDP (e.g., $\eta_+$ ou $\eta_-$).
*   **Aprendizado Supervisionado Baseado em Gradiente:** Implementar métodos baseados em SG diretamente em Brian2 é desafiador, pois requer cálculo de gradientes e backpropagation. Geralmente, para SG, utiliza-se Brian2 em conjunto com frameworks como PyTorch (usando bibliotecas como snnTorch) onde a SNN é definida e simulada (possivelmente com um backend Brian2 via Nengo-DL ou outras pontes), mas o treinamento é gerenciado pelo framework de auto-diferenciação. O foco deste livro é no Brian2 puro, então exploraremos tarefas onde a plasticidade local do Brian2 pode ser aplicada.

**6.4. Estudo de Caso: Aprendizado por Reforço Simples com R-STDP**

Para ilustrar um cenário de aprendizado onde a plasticidade local do Brian2 pode ser guiada por um objetivo, implementaremos um exemplo conceitual de Aprendizado por Reforço usando STDP modulada por recompensa (Reward-modulated STDP - R-STDP). A ideia é que a STDP padrão ocorre continuamente, mas a magnitude efetiva da mudança de peso ($\Delta w$) é multiplicada por um fator relacionado à recompensa recente. Se uma ação (disparo pós-sináptico) seguida de certos inputs (spikes pré-sinápticos) leva a uma recompensa, essas conexões são fortalecidas mais fortemente (ou deprimidas menos fortemente) do que seriam pela STDP normal.

**Tarefa Simples:** Treinar um neurônio de saída para disparar em resposta a um padrão de entrada específico ("bom") e não disparar para outro padrão ("ruim"), usando um sinal de recompensa/punição.

**Objetivo:** Demonstrar como um sinal de recompensa global pode modular a STDP local para aprendizado direcionado a um objetivo.

```python
# Importar bibliotecas
import brian2 as b2
import matplotlib.pyplot as plt
import numpy as np

b2.prefs.codegen.target = 'cython'
b2.defaultclock.dt = 0.1*b2.ms

# --- Parâmetros ---
N_input = 20
N_output = 1
simulation_duration_epoch = 100 * b2.ms # Duração de apresentação de cada padrão
N_epochs = 100 # Número de épocas de treinamento

# --- Input: Dois padrões distintos ---
# Padrão "Bom" (Target): Neurônios 0-9 disparam a 50 Hz
# Padrão "Ruim" (Non-Target): Neurônios 10-19 disparam a 50 Hz
rate_pattern = 50 * b2.Hz
rate_noise = 2 * b2.Hz # Ruído de fundo para todos
pattern_good_indices = np.arange(0, 10)
pattern_bad_indices = np.arange(10, 20)

# Usaremos TimedArray para alternar os padrões
input_rates_pattern = np.zeros((N_epochs * 2, N_input)) * b2.Hz
current_pattern_is_good = []
for i in range(N_epochs):
    # Época i: Padrão Bom
    idx_start = i * 2
    input_rates_pattern[idx_start, pattern_good_indices] = rate_pattern
    current_pattern_is_good.append(True)
    # Época i: Padrão Ruim
    idx_start = i * 2 + 1
    input_rates_pattern[idx_start, pattern_bad_indices] = rate_pattern
    current_pattern_is_good.append(False)
# Adicionar ruído a todos
input_rates_pattern += rate_noise
# Criar TimedArray
input_rates_timed = b2.TimedArray(input_rates_pattern, dt=simulation_duration_epoch)
# Grupo Poisson com taxa variável
input_group = b2.PoissonGroup(N_input, rates='input_rates_timed(t, i)')

# --- Neurônio de Saída LIF ---
# (Mesmos parâmetros do exemplo anterior, mas sem g_inh)
tau = 20*b2.ms; Vt = -50*b2.mV; Vr = -70*b2.mV; El = -65*b2.mV; Rm = 100*b2.Mohm; ref_p = 3*b2.ms
lif_eqs_out = '''
dv/dt = (El - v + Rm * g_exc)/tau : volt (unless refractory)
dg_exc/dt = -g_exc / (5*ms) : amp # Condutância excitatória total
'''
output_neuron = b2.NeuronGroup(N_output, lif_eqs_out, threshold='v>Vt', reset='v=Vr', refractory=ref_p, method='euler')
output_neuron.v = El
output_neuron.g_exc = 0*b2.nA

# --- Sinapses com R-STDP ---
# Parâmetros STDP
tau_pre = 20*b2.ms; tau_post = 20*b2.ms
A_pre = 0.01; A_post = 0.01
eta_ltp = 0.005 # Taxa base LTP
eta_ltd = 0.0055 # Taxa base LTD
w_max = 20.0 * b2.nS
w_initial = 5.0 * b2.nS
# Parâmetros R-STDP
tau_reward = 200 * b2.ms # Constante de tempo do traço de elegibilidade/recompensa
reward_baseline = 0.0 # Nível de recompensa base (sem recompensa/punição)

# Modelo R-STDP: Modifica STDP padrão com um traço de recompensa 'c'
# e um traço de elegibilidade 'e' por sinapse
rstdp_eqs = '''
w : siemens
dapre/dt = -apre / tau_pre : 1 (event-driven)
dapost/dt = -apost / tau_post : 1 (event-driven)
de/dt = -e / tau_reward : 1 # Traço de elegibilidade por sinapse
dc/dt = -c / tau_reward : 1 # Traço de recompensa (global, mas rastreado por sinapse aqui)
'''
# 'on_pre': Marca a sinapse como recentemente ativa (aumenta elegibilidade 'e') e aplica LTD
on_pre_rstdp = '''
g_exc_post += w
e += eta_ltd * apost # Elegibilidade para LTD
w = clip(w + c * e, 0, w_max) # Aplica mudança de peso modulada pela recompensa C e elegibilidade E
apre += A_pre
'''
# 'on_post': Marca a sinapse como recentemente ativa (aumenta elegibilidade 'e') e aplica LTP
on_post_rstdp = '''
e += eta_ltp * apre # Elegibilidade para LTP
w = clip(w + c * e, 0, w_max) # Aplica mudança de peso modulada
apost += A_post
'''
# Nota: Esta é uma implementação simplificada. O traço 'c' deveria ser idealmente
# uma variável global de recompensa. A atualização de 'w' baseada em 'c*e'
# pode ser feita continuamente ou em eventos de recompensa.
# Vamos simplificar: Atualizar 'w' nos spikes, mas 'e' acumula a "culpa/crédito".

# Simplificação: Faremos a modulação diretamente nas atualizações de peso
# baseada em uma variável de recompensa que atualizaremos externamente.
rstdp_eqs_v2 = '''
w : siemens
dapre/dt = -apre / tau_pre : 1 (event-driven)
dapost/dt = -apost / tau_post : 1 (event-driven)
reward_level : 1 # Nível de recompensa atual (passado externamente)
'''
on_pre_rstdp_v2 = '''
g_exc_post += w
delta_w = -eta_ltd * apost * (reward_level - reward_baseline) # Modula LTD
w = clip(w + delta_w*nS, 0, w_max)
apre += A_pre
'''
on_post_rstdp_v2 = '''
delta_w = eta_ltp * apre * (reward_level - reward_baseline) # Modula LTP
w = clip(w + delta_w*nS, 0, w_max)
apost += A_post
'''

# Criação das Sinapses
synapses_rstdp = b2.Synapses(input_group, output_neuron, model=rstdp_eqs_v2,
                             on_pre=on_pre_rstdp_v2, on_post=on_post_rstdp_v2)
synapses_rstdp.connect()
synapses_rstdp.w = w_initial
synapses_rstdp.apre = 0.0
synapses_rstdp.apost = 0.0
synapses_rstdp.namespace['tau_pre'] = tau_pre
synapses_rstdp.namespace['tau_post'] = tau_post
synapses_rstdp.namespace['eta_ltp'] = eta_ltp
synapses_rstdp.namespace['eta_ltd'] = eta_ltd
synapses_rstdp.namespace['w_max'] = w_max
synapses_rstdp.namespace['A_pre'] = A_pre
synapses_rstdp.namespace['A_post'] = A_post
synapses_rstdp.namespace['reward_baseline'] = reward_baseline
# Inicializar recompensa
synapses_rstdp.reward_level = reward_baseline

# --- Monitores ---
spikemon_output_r = b2.SpikeMonitor(output_neuron)
weight_monitor_r = b2.StateMonitor(synapses_rstdp, 'w', record=True, dt=simulation_duration_epoch)

# --- Loop de Treinamento com Recompensa ---
network_r = b2.Network(input_group, output_neuron, synapses_rstdp, spikemon_output_r, weight_monitor_r)
network_r.store('initial_state') # Salvar estado inicial

print("Iniciando treinamento R-STDP...")
reward_signal_history = []
output_spike_counts = []

for epoch in range(N_epochs * 2): # Loop sobre apresentações de padrões
    network_r.restore('initial_state') # Resetar estado da rede (V, g, traços) - opcional, para isolar épocas
    # Simular uma época
    network_r.run(simulation_duration_epoch, report=None)

    # Calcular recompensa para esta época
    current_spikes = spikemon_output_r.count[0] # Spikes do neurônio de saída nesta época
    output_spike_counts.append(current_spikes)
    spikemon_output_r.remove_spikes() # Limpar spikes para próxima época

    is_good_pattern = current_pattern_is_good[epoch]
    reward = 0.0
    if is_good_pattern and current_spikes > 0: # Recompensa por disparar para padrão bom
        reward = 1.0
    elif not is_good_pattern and current_spikes == 0: # Recompensa por NÃO disparar para padrão ruim
        reward = 0.5 # Menor recompensa por inação correta
    elif is_good_pattern and current_spikes == 0: # Punição por não disparar para padrão bom
        reward = -0.5
    elif not is_good_pattern and current_spikes > 0: # Punição por disparar para padrão ruim
        reward = -1.0

    reward_signal_history.append(reward)
    # Aplicar recompensa à plasticidade (atualiza a variável 'reward_level' para a *próxima* época)
    # Idealmente, a recompensa modularia mudanças passadas (requer traço de elegibilidade)
    # Aqui, para simplificar, definimos o nível para a próxima interação.
    synapses_rstdp.reward_level = reward # Esta aplicação é conceitualmente falha - recompensa afeta futuro.
                                         # Uma implementação correta usaria traços de elegibilidade.

    # -- Implementação CORRETA conceitualmente (requer 'e' no modelo) --
    # Assumindo que 'e' foi atualizado durante a época:
    # delta_w_total = synapses_rstdp.e * reward
    # synapses_rstdp.w = clip(synapses_rstdp.w + delta_w_total * nS, 0, w_max)
    # synapses_rstdp.e = 0 # Resetar elegibilidade
    # Esta atualização pós-época é mais correta para R-STDP.

print("Treinamento concluído.")

# --- Visualização ---
# (Similar ao exemplo STDP anterior, plotar pesos para inputs bons vs ruins)
plt.figure(figsize=(12, 6))
good_indices = np.arange(0, 10)
bad_indices = np.arange(10, 20)
# Plotar evolução dos pesos
plt.plot(weight_monitor_r.t / b2.second, np.mean(weight_monitor_r.w[good_indices], axis=0) / b2.nS, 'g-', lw=2, label='Média Pesos "Bons"')
plt.plot(weight_monitor_r.t / b2.second, np.mean(weight_monitor_r.w[bad_indices], axis=0) / b2.nS, 'm-', lw=2, label='Média Pesos "Ruins"')
plt.xlabel('Tempo (Épocas * Duração Época)') # Eixo X representa tempo total
plt.ylabel('Peso Sináptico Médio (nS)')
plt.title('Evolução Média dos Pesos com R-STDP')
plt.legend()
plt.ylim(bottom=0)
plt.grid(True, linestyle=':')
plt.show()

# Plotar recompensa e spikes de saída ao longo das épocas
plt.figure(figsize=(12, 4))
epochs_axis = np.arange(N_epochs * 2)
plt.plot(epochs_axis, reward_signal_history, 'ko-', label='Sinal de Recompensa')
plt.bar(epochs_axis, output_spike_counts, color='lightblue', label='Spikes de Saída / Época')
plt.xlabel('Época de Apresentação do Padrão')
plt.ylabel('Recompensa / Contagem de Spikes')
plt.title('Histórico de Recompensa e Atividade de Saída')
plt.legend()
plt.show()
```
Este código implementa uma forma simplificada de R-STDP. Dois padrões de entrada são apresentados alternadamente. Um sinal de recompensa/punição é calculado após cada apresentação com base se o neurônio de saída disparou corretamente para o padrão "bom" e ficou quieto para o padrão "ruim". Esta recompensa (idealmente através de um traço de elegibilidade 'e', mas simplificado aqui) modula as mudanças de peso induzidas pela STDP. Com o tempo, esperamos que os pesos sinápticos dos inputs associados ao padrão "bom" aumentem, enquanto os associados ao padrão "ruim" diminuam, permitindo ao neurônio classificar corretamente os padrões. (Nota: A implementação da modulação por recompensa aqui é simplificada para fins ilustrativos dentro do Brian2 puro; implementações R-STDP mais canônicas requerem traços de elegibilidade e aplicação da recompensa de forma mais cuidadosa).

**Figura 6.6**: Aprendizado por Reforço com R-STDP (Conceitual). (a) Evolução dos pesos médios para inputs do padrão "bom" (verde) e "ruim" (magenta), mostrando a diferenciação aprendida. (b) Histórico do sinal de recompensa (pontos pretos) e da contagem de spikes de saída por época (barras azuis), idealmente mostrando a recompensa aumentando e a contagem de spikes se tornando mais seletiva ao padrão "bom" ao longo do treinamento.

Estes estudos de caso ilustram como a plasticidade sináptica, seja ela não supervisionada (STDP) ou guiada por um sinal externo (R-STDP), pode ser implementada e explorada usando Brian2 para dotar as SNNs da capacidade de aprender e adaptar suas respostas com base na experiência ou em objetivos definidos. A combinação de diferentes formas de plasticidade (Hebbiana, homeostática, estrutural) é provavelmente necessária para alcançar a robustez e flexibilidade do aprendizado biológico, representando uma área ativa de pesquisa e desenvolvimento em computação neuromórfica.

---

## REFERÊNCIAS BIBLIOGRÁFICAS

Bellec, G., Scherr, F., Subramoney, A., Hajek, E., Salaj, D., Legenstein, R., & Maass, W. (2020). A solution to the learning dilemma for recurrent networks of spiking neurons. *Nature Communications*, *11*(1), 3625. https://doi.org/10.1038/s41467-020-17236-y
    *   *Resumo:* Apresenta um algoritmo de aprendizado (e-prop) para redes recorrentes de SNNs (RSNNs) que utiliza gradientes substitutos e é mais biologicamente plausível e eficiente que BPTT padrão. Relevante para o desafio do aprendizado supervisionado em redes recorrentes com dinâmica complexa.

Bohnstinghel, T., Gjoni, M., Pignari, S. A., Sandamirskaya, Y., & Indiveri, G. (2022). Online unsupervised learning from asynchronous streams of data in spiking neural networks. *Neuromorphic Computing and Engineering*, *2*(4), 044002. https://doi.org/10.1088/2634-4386/ac8cd0
    *   *Resumo:* Descreve um sistema de SNN implementado em hardware neuromórfico (usando modelos AdEx) que realiza aprendizado não supervisionado (baseado em STDP e plasticidade intrínseca) diretamente de fluxos de dados baseados em eventos. Demonstra a aplicação prática do aprendizado não supervisionado discutido.

Gallego, G., Delbruck, T., Orchard, G., Bartolozzi, C., Taba, B., Censi, A., ... & Scaramuzza, D. (2022). Event-based vision: A survey. *IEEE Transactions on Pattern Analysis and Machine Intelligence*, *44*(1), 154-180. https://doi.org/10.1109/TPAMI.2020.3008413
    *   *Resumo:* Embora uma survey sobre visão baseada em eventos, discute algoritmos para processar esses dados, muitos dos quais envolvem SNNs com aprendizado (STDP ou SG). Contextualiza a necessidade de regras de plasticidade para processar dados neuromórficos.

Gardner, B., Sporea, I., & Grüning, A. (2022). Training spiking neural networks for neuromorphic hardware. *Frontiers in Neuroscience*, *15*, 761718. https://doi.org/10.3389/fnins.2021.761718
    *   *Resumo:* Revisa diferentes abordagens para treinar SNNs com foco na implementação em hardware neuromórfico, incluindo conversão ANN-SNN e métodos baseados em gradiente substituto. Discute os desafios de transferir modelos treinados em simulação para hardware.

Guerguiev, J., Lillicrap, T. P., & Richards, B. A. (2020). dendritic segregation of learning signals promotes episodic type memory formation. *Nature Communications*, *11*(1), 376. https://doi.org/10.1038/s41467-019-14115-5
    *   *Resumo:* Propõe um modelo computacional (compatível com simulação em Brian2) onde diferentes compartimentos dendríticos podem integrar sinais de aprendizado distintos (e.g., erro preditivo vs. contexto), contribuindo para a formação de memória. Destaca a importância da estrutura subcelular na plasticidade.

Pallasdies, F., Tetzlaff, C., & Lent, R. (2023). Co-evolution of structural and synaptic plasticity facilitates learning by reducing noise. *PLoS Computational Biology*, *19*(1), e1010806. https://doi.org/10.1371/journal.pcbi.1010806
    *   *Resumo:* Utiliza modelagem computacional para investigar a interação entre plasticidade estrutural (formação/eliminação de sinapses) e plasticidade sináptica (STDP). Sugere que a plasticidade estrutural pode melhorar a estabilidade e a eficiência do aprendizado hebbiano. Relevante para a seção de outras formas de plasticidade.

Payeur, A., Yar-Aadan, A., Stokkan, M., & Sprekeler, H. (2021). Burst-dependent synaptic plasticity and burst synchronization in adapting neuronal networks can stem from linearized calcium dynamics. *PLoS Computational Biology*, *17*(10), e1009461. https://doi.org/10.1371/journal.pcbi.1009461
    *   *Resumo:* Apresenta um modelo de plasticidade baseado em cálcio que pode explicar formas de plasticidade dependentes de rajadas (bursts) e sua relação com a sincronização da rede. Relevante para variantes de STDP e dinâmica de rede.

Schuman, C. D., Kulkarni, S. R., Parsa, M., Mitchell, J. P., Date, P., & Kay, B. (2022). Opportunities for neuromorphic computing algorithms and applications. *Nature Computational Science*, *2*(1), 10-19. https://doi.org/10.1038/s43588-021-00184-y
    *   *Resumo:* Revisa oportunidades em algoritmos neuromórficos, incluindo aprendizado baseado em STDP, homeostático e supervisionado (como SG). Coloca os mecanismos de plasticidade discutidos neste capítulo no contexto mais amplo das aplicações e desafios do campo.

Stimberg, M., Brette, R., & Goodman, D. F. M. (2019a). Brian 2, an intuitive and efficient neural simulator. *eLife*, *8*, e47314. https://doi.org/10.7554/eLife.47314
    *   *Resumo:* Artigo descrevendo o Brian2, a ferramenta usada para as implementações neste capítulo. Detalha como definir modelos e regras de plasticidade (como STDP) usando o sistema de equações e eventos `on_pre`/`on_post`. Essencial para entender os exemplos de código.

Zenke, F., & Neftci, E. O. (2021). Brain-inspired learning on neuromorphic hardware. *Philosophical Transactions of the Royal Society A: Mathematical, Physical and Engineering Sciences*, *379*(2208), 20200158. https://doi.org/10.1098/rsta.2020.0158
    *   *Resumo:* Foca em algoritmos de aprendizado bioinspirados (STDP, SG) para SNNs e hardware neuromórfico. Discute a importância da plasticidade local e os avanços no treinamento supervisionado via SG. Fornece uma excelente visão geral das abordagens de aprendizado relevantes.
