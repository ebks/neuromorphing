
---

# APÊNDICE D

# DERIVAÇÕES MATEMÁTICAS DETALHADAS DE MODELOS SELECIONADOS

---

![imagem](neuro.png)

*Este apêndice fornece derivações matemáticas mais explícitas para alguns dos modelos neuronais e sinápticos fundamentais apresentados no corpo principal deste livro, particularmente no Capítulo 3. O objetivo é oferecer ao leitor interessado uma compreensão mais profunda das bases matemáticas subjacentes a esses modelos, complementando a descrição conceitual e as implementações práticas fornecidas anteriormente. Serão abordados o potencial de Nernst, a solução analítica do modelo Leaky Integrate-and-Fire (LIF) para entrada constante, a estrutura matemática do modelo de Hodgkin-Huxley, a dinâmica temporal de sinapses simples e a derivação da janela de STDP a partir de um modelo baseado em traços.*

---

**D.1 Potencial de Equilíbrio de Nernst**

O potencial de equilíbrio para um íon específico, $E_{ion}$, também conhecido como potencial de Nernst, representa a diferença de potencial elétrico através de uma membrana permeável àquele íon que equilibra exatamente a força motriz devida ao gradiente de concentração desse íon. Neste potencial, não há fluxo líquido do íon através da membrana. A derivação baseia-se em princípios termodinâmicos e eletroquímicos.

O fluxo de um íon $X$ com valência $z$ através da membrana é impulsionado pelo gradiente de potencial eletroquímico, $\Delta \mu_X$. O equilíbrio é atingido quando $\Delta \mu_X = 0$. O potencial eletroquímico $\mu_X$ tem dois componentes: um químico, relacionado à concentração, e um elétrico, relacionado ao potencial elétrico $V_m$:

$\mu_X = \mu_X^0 + RT \ln [X] + zFV_m$

onde:
*   $\mu_X^0$ é o potencial químico padrão.
*   $R$ é a constante dos gases ideais.
*   $T$ é a temperatura absoluta (em Kelvin).
*   $[X]$ é a concentração molar do íon.
*   $z$ é a valência do íon (e.g., +1 para K⁺, Na⁺; +2 para Ca²⁺; -1 para Cl⁻).
*   $F$ é a constante de Faraday (carga por mol de elétrons).
*   $V_m$ é o potencial elétrico da membrana (potencial interno menos externo).

A diferença de potencial eletroquímico entre o interior (in) e o exterior (out) da célula é:
$\Delta \mu_X = \mu_{X,in} - \mu_{X,out}$
$\Delta \mu_X = (\mu_X^0 + RT \ln [X]_{in} + zFV_{m,in}) - (\mu_X^0 + RT \ln [X]_{out} + zFV_{m,out})$

Assumindo que o potencial externo $V_{m,out}$ é zero (referência), temos $V_{m,in} = V_m$. A equação simplifica para:
$\Delta \mu_X = RT \ln \frac{[X]_{in}}{[X]_{out}} + zFV_m$

No equilíbrio, $\Delta \mu_X = 0$. O potencial de membrana $V_m$ neste ponto é o potencial de Nernst, $E_X$:
$0 = RT \ln \frac{[X]_{in}}{[X]_{out}} + zFE_X$

Resolvendo para $E_X$:
$zFE_X = -RT \ln \frac{[X]_{in}}{[X]_{out}} = RT \ln \left( \frac{[X]_{in}}{[X]_{out}} \right)^{-1}$
$E_X = \frac{RT}{zF} \ln \frac{[X]_{out}}{[X]_{in}}$

Esta é a **Equação de Nernst**. Ela define o potencial de equilíbrio para um único tipo de íon baseado em suas concentrações intra e extracelular e sua valência, sob uma dada temperatura. É um conceito fundamental para entender as forças motrizes iônicas e o potencial de repouso da membrana (que depende das permeabilidades relativas a múltiplos íons, conforme descrito pela equação GHK no Capítulo 2).

**D.2 Solução Analítica do Modelo Leaky Integrate-and-Fire (LIF) para Entrada Constante**

O modelo LIF (ver Seção 3.2.1) descreve a dinâmica sub-limiar do potencial de membrana $V(t)$ pela equação diferencial linear de primeira ordem:
$\tau_m \frac{dV}{dt} = -(V - E_L) + R I(t)$

Consideremos o caso onde a corrente de entrada $I(t)$ é constante, $I(t) = I_0$. A equação torna-se:
$\tau_m \frac{dV}{dt} = -V + E_L + R I_0$

Podemos reescrevê-la na forma padrão $\frac{dy}{dx} + P(x)y = Q(x)$:
$\frac{dV}{dt} + \frac{1}{\tau_m}V = \frac{E_L + R I_0}{\tau_m}$

Esta é uma ODE linear de primeira ordem não homogênea. Pode ser resolvida usando um fator integrante $e^{\int (1/\tau_m) dt} = e^{t/\tau_m}$. Multiplicando ambos os lados pelo fator integrante:
$e^{t/\tau_m}\frac{dV}{dt} + \frac{1}{\tau_m}e^{t/\tau_m}V = \frac{E_L + R I_0}{\tau_m}e^{t/\tau_m}$

O lado esquerdo é a derivada do produto $V(t) e^{t/\tau_m}$:
$\frac{d}{dt} \left( V(t) e^{t/\tau_m} \right) = \frac{E_L + R I_0}{\tau_m}e^{t/\tau_m}$

Integrando ambos os lados em relação a $t$:
$V(t) e^{t/\tau_m} = \int \frac{E_L + R I_0}{\tau_m}e^{t/\tau_m} dt$
$V(t) e^{t/\tau_m} = (E_L + R I_0) e^{t/\tau_m} + K$
onde $K$ é a constante de integração.

Dividindo por $e^{t/\tau_m}$:
$V(t) = (E_L + R I_0) + K e^{-t/\tau_m}$

Podemos definir o **potencial de estado estacionário** (ou assintótico) $V_\infty = E_L + R I_0$. Este é o valor para o qual $V(t)$ tenderia se não houvesse limiar de disparo. A equação torna-se:
$V(t) = V_\infty + K e^{-t/\tau_m}$

Para determinar $K$, usamos a condição inicial. Se no tempo $t=0$, o potencial é $V(0) = V_0$, então:
$V_0 = V_\infty + K e^0 \implies K = V_0 - V_\infty$

Portanto, a solução geral para a dinâmica sub-limiar do LIF com entrada constante $I_0$, começando de $V_0$ em $t=0$, é:
$V(t) = V_\infty + (V_0 - V_\infty) e^{-t/\tau_m}$

**Tempo para o Primeiro Spike:**
Se o neurônio começa em $V(0) = V_{reset}$ (imediatamente após um spike anterior ou no início da simulação) e $V_\infty > V_{th}$ (ou seja, a corrente $I_0$ é suficiente para eventualmente causar um disparo), podemos calcular o tempo $t_{spike}$ que leva para $V(t)$ atingir o limiar $V_{th}$:
$V_{th} = V_\infty + (V_{reset} - V_\infty) e^{-t_{spike}/\tau_m}$

Resolvendo para $t_{spike}$:
$V_{th} - V_\infty = (V_{reset} - V_\infty) e^{-t_{spike}/\tau_m}$
$\frac{V_{th} - V_\infty}{V_{reset} - V_\infty} = e^{-t_{spike}/\tau_m}$

Tomando o logaritmo natural de ambos os lados:
$\ln\left(\frac{V_{th} - V_\infty}{V_{reset} - V_\infty}\right) = -\frac{t_{spike}}{\tau_m}$

$t_{spike} = -\tau_m \ln\left(\frac{V_{th} - V_\infty}{V_{reset} - V_\infty}\right)$
$t_{spike} = \tau_m \ln\left(\frac{V_{reset} - V_\infty}{V_{th} - V_\infty}\right)$

Esta expressão fornece o intervalo inter-spike (ISI) para um neurônio LIF sob corrente constante $I_0$, assumindo $V_\infty = E_L + RI_0 > V_{th}$.

**Frequência de Disparo:**
Se o período refratário absoluto for $t_{ref}$, a frequência de disparo $f$ é o inverso do tempo total entre spikes:
$f = \frac{1}{t_{spike} + t_{ref}} = \frac{1}{\tau_m \ln\left(\frac{V_{reset} - V_\infty}{V_{th} - V_\infty}\right) + t_{ref}}$
Esta é a curva F-I (frequência vs. corrente de entrada $I_0$) para o modelo LIF. Note que $f=0$ se $V_\infty \le V_{th}$.

**D.3 Estrutura Matemática do Modelo de Hodgkin-Huxley (HH)**

O modelo HH (Hodgkin & Huxley, 1952) descreve a dinâmica do potencial de membrana $V_m$ e das variáveis de gating $m, h, n$ através de um sistema de quatro ODEs acopladas não lineares.

**Equação da Membrana (Baseada na Lei de Kirchhoff para Correntes):**
A corrente total que atravessa a membrana capacitiva $C_m$ é igual à soma negativa das correntes iônicas mais qualquer corrente externa aplicada $I_{ext}$:
$C_m \frac{dV_m}{dt} = -I_{Na} - I_K - I_L + I_{ext}$

As correntes iônicas são modeladas usando condutâncias ($g$) e potenciais de Nernst ($E$):
*   $I_{Na} = g_{Na}(V_m, t) (V_m - E_{Na})$
*   $I_K = g_{K}(V_m, t) (V_m - E_{K})$
*   $I_L = \bar{g}_L (V_m - E_L)$ (condutância de vazamento $\bar{g}_L$ é constante)

**Equações das Condutâncias Dependentes de Voltagem:**
As condutâncias de Sódio e Potássio dependem de variáveis de gating ($m, h, n$) que representam a probabilidade de "portões" hipotéticos estarem abertos:
*   $g_{Na}(V_m, t) = \bar{g}_{Na} m^3 h$
*   $g_{K}(V_m, t) = \bar{g}_{K} n^4$
onde $\bar{g}_{Na}$ e $\bar{g}_{K}$ são as condutâncias máximas.

**Equações das Variáveis de Gating:**
A dinâmica de cada variável de gating $x \in \{m, h, n\}$ segue uma cinética de primeira ordem, dependendo das taxas de transição $\alpha_x(V_m)$ (taxa de abertura) e $\beta_x(V_m)$ (taxa de fechamento):
$\frac{dx}{dt} = \alpha_x(V_m)(1 - x) - \beta_x(V_m)x$

Esta equação descreve a transição entre um estado fechado (probabilidade $1-x$) e um estado aberto (probabilidade $x$). As funções $\alpha_x(V_m)$ e $\beta_x(V_m)$ são funções não lineares da voltagem, determinadas empiricamente por Hodgkin e Huxley para o axônio gigante de lula. Elas tipicamente têm formas exponenciais ou sigmoides. Por exemplo:
$\alpha_m(V_m) = \frac{0.1(V_m - V_{th,m})}{\exp(1 - (V_m - V_{th,m})/10) - 1}$ (Forma típica, com $V_{th,m}$ como um potencial de referência)
$\beta_m(V_m) = 4 \exp(-(V_m - V_{th,m})/18)$

**Variáveis de Estado Estacionário e Constantes de Tempo:**
Para uma voltagem constante $V$, a variável de gating $x$ relaxará para um valor de estado estacionário $x_\infty(V)$ com uma constante de tempo $\tau_x(V)$. Definindo $dx/dt = 0$:
$\alpha_x(V)(1 - x_\infty) = \beta_x(V)x_\infty$
$\alpha_x(V) = (\alpha_x(V) + \beta_x(V))x_\infty$
$x_\infty(V) = \frac{\alpha_x(V)}{\alpha_x(V) + \beta_x(V)}$

A equação dinâmica pode ser reescrita como:
$\frac{dx}{dt} = \frac{x_\infty(V) - x}{\tau_x(V)}$
onde a constante de tempo dependente de voltagem é:
$\tau_x(V) = \frac{1}{\alpha_x(V) + \beta_x(V)}$

As funções $x_\infty(V)$ (tipicamente sigmoidais) descrevem a fração de portões abertos no equilíbrio para cada voltagem, enquanto $\tau_x(V)$ (tipicamente em forma de sino) descreve quão rapidamente eles atingem esse equilíbrio. O modelo HH completo requer a especificação dessas seis funções de taxa ($\alpha_m, \beta_m, \alpha_h, \beta_h, \alpha_n, \beta_n$) ou equivalentemente das funções de estado estacionário e constante de tempo ($m_\infty, \tau_m, h_\infty, \tau_h, n_\infty, \tau_n$), além dos parâmetros $C_m, \bar{g}_{Na}, \bar{g}_{K}, \bar{g}_L, E_{Na}, E_K, E_L$. A solução do sistema acoplado de 4 ODEs requer métodos numéricos.

**D.4 Dinâmica Temporal de Sinapses Simples**

Modelos sinápticos descrevem como a chegada de um spike pré-sináptico causa uma mudança transitória na corrente ou condutância pós-sináptica.

**D.4.1 Sinapse com Decaimento Exponencial:**
Assume-se que a chegada de um spike em $t=0$ causa um aumento instantâneo na condutância (ou corrente) para um valor $A$, seguido por um decaimento exponencial passivo com constante de tempo $\tau_{syn}$. A dinâmica pós-spike é:
$\tau_{syn} \frac{dg}{dt} = -g \quad (\text{ou } \tau_{syn} \frac{dI}{dt} = -I)$
com $g(0^+) = A$ (ou $I(0^+) = A$).

A solução desta ODE linear de primeira ordem homogênea é:
$g(t) = A e^{-t/\tau_{syn}}$ para $t > 0$.

Este modelo captura um decaimento simples, mas não tem uma fase de subida realista.

**D.4.2 Sinapse com Função Alfa:**
Para obter uma subida e descida mais graduais, pode-se modelar a sinapse como um processo de duas etapas em série com a mesma constante de tempo $\tau_{syn}$. Uma variável intermediária $x$ é ativada instantaneamente pelo spike e decai, enquanto $g$ é ativada por $x$ e também decai:
$\frac{dx}{dt} = A \delta(t) - \frac{x}{\tau_{syn}}$
$\frac{dg}{dt} = x - \frac{g}{\tau_{syn}}$ (A constante de proporcionalidade foi absorvida em A)

Resolvendo a primeira equação, $x(t) = A e^{-t/\tau_{syn}}$ para $t > 0$. Substituindo na segunda:
$\frac{dg}{dt} + \frac{1}{\tau_{syn}}g = A e^{-t/\tau_{syn}}$

Esta é uma ODE linear de primeira ordem não homogênea. Usando o fator integrante $e^{t/\tau_{syn}}$:
$\frac{d}{dt} (g(t)e^{t/\tau_{syn}}) = A e^{-t/\tau_{syn}} e^{t/\tau_{syn}} = A$

Integrando:
$g(t)e^{t/\tau_{syn}} = \int A dt = At + K$
$g(t) = (At + K)e^{-t/\tau_{syn}}$

Usando a condição inicial $g(0)=0$, temos $K=0$. Portanto:
$g(t) = A t e^{-t/\tau_{syn}}$

Esta é a **função alfa**, que sobe para um pico em $t=\tau_{syn}$ e depois decai.

**D.4.3 Sinapse com Função Bi-exponencial:**
Se as constantes de tempo da fase de subida ($\tau_{rise}$) e de descida ($\tau_{fall}$) forem diferentes ($\tau_{fall} > \tau_{rise}$), a dinâmica da condutância pode ser modelada como a diferença de duas exponenciais:
$g(t) = A' (e^{-t/\tau_{fall}} - e^{-t/\tau_{rise}})$
onde $A'$ é uma constante de normalização. Esta forma também exibe subida e descida graduais e é frequentemente usada para modelar respostas sinápticas de forma mais realista do que a exponencial simples ou a função alfa. Pode ser derivada de um esquema cinético com estados aberto e fechado.

**D.5 Derivação da Janela STDP a partir de Traços de Spike**

Conforme descrito na Seção 6.1.1, uma implementação computacional comum da STDP utiliza traços pré-sinápticos ($P$) e pós-sinápticos ($O$) que decaem exponencialmente e são incrementados a cada spike.

Equações dos Traços:
$\tau_{pre} \frac{dP}{dt} = -P$; se $t = t_{pre}$, $P \rightarrow P + A_{pre}$
$\tau_{post} \frac{dO}{dt} = -O$; se $t = t_{post}$, $O \rightarrow O + A_{post}$

Regras de Atualização do Peso $w$:
*   Em $t_{post}$ (spike pós): $\Delta w = \eta_+ P(t_{post}^-)$ (LTP)
*   Em $t_{pre}$ (spike pré): $\Delta w = -\eta_- O(t_{pre}^-)$ (LTD)

Vamos derivar a mudança de peso $\Delta w$ para um único par de spikes: um pré-sináptico em $t_{pre}$ e um pós-sináptico em $t_{post}$.

**Caso 1: $\Delta t = t_{post} - t_{pre} > 0$ (Pré antes de Pós - LTP)**
O spike pré-sináptico ocorre primeiro em $t_{pre}$. O traço $P$ é incrementado para $A_{pre}$ (assumindo que estava em 0 antes). Ele então decai exponencialmente até o momento imediatamente antes do spike pós-sináptico, $t_{post}^-$.
$P(t) = A_{pre} e^{-(t-t_{pre})/\tau_{pre}}$ para $t > t_{pre}$.
Portanto, em $t_{post}^-$, o valor do traço pré é:
$P(t_{post}^-) = A_{pre} e^{-(t_{post}-t_{pre})/\tau_{pre}} = A_{pre} e^{-\Delta t/\tau_{pre}}$

A atualização do peso ocorre no momento do spike pós-sináptico ($t_{post}$) e depende deste valor do traço pré:
$\Delta w_{LTP} = \eta_+ P(t_{post}^-) = \eta_+ A_{pre} e^{-\Delta t/\tau_{pre}}$
Isso corresponde exatamente à parte de LTP da janela STDP exponencial, com $A_+ = \eta_+ A_{pre}$ e $\tau_+ = \tau_{pre}$.

**Caso 2: $\Delta t = t_{post} - t_{pre} < 0$ (Pós antes de Pré - LTD)**
O spike pós-sináptico ocorre primeiro em $t_{post}$. O traço $O$ é incrementado para $A_{post}$ e decai exponencialmente. A atualização do peso relevante ocorre no momento do spike pré-sináptico ($t_{pre}$) e depende do valor do traço pós imediatamente antes, $O(t_{pre}^-)$.
$O(t) = A_{post} e^{-(t-t_{post})/\tau_{post}}$ para $t > t_{post}$.
Portanto, em $t_{pre}^-$, o valor do traço pós é:
$O(t_{pre}^-) = A_{post} e^{-(t_{pre}-t_{post})/\tau_{post}}$

A atualização do peso em $t_{pre}$ é:
$\Delta w_{LTD} = -\eta_- O(t_{pre}^-) = -\eta_- A_{post} e^{-(t_{pre}-t_{post})/\tau_{post}}$
Como $\Delta t = t_{post} - t_{pre}$, então $t_{pre} - t_{post} = -\Delta t$. Substituindo:
$\Delta w_{LTD} = -\eta_- A_{post} e^{\Delta t/\tau_{post}}$
Isso corresponde à parte de LTD da janela STDP exponencial, com $A_- = \eta_- A_{post}$ e $\tau_- = \tau_{post}$.

Combinando ambos os casos, a implementação baseada em traços reproduz fielmente a janela de STDP exponencial padrão para pares de spikes isolados. Para trens de spikes múltiplos, ela aproxima a regra de "todos os pares" (all-pairs) ou regras similares, dependendo da linearidade assumida nas atualizações dos traços e pesos.

Estas derivações matemáticas fornecem um embasamento mais rigoroso para os modelos computacionais utilizados extensivamente na simulação de SNNs com ferramentas como Brian2.
