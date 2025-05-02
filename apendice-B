
---

# APÊNDICE B

# GLOSSÁRIO TERMINOLÓGICO

---

![imagem](neuro.png)

*Este glossário fornece definições concisas para termos técnicos e conceitos chave utilizados ao longo deste livro, abrangendo neurobiologia, neurociência computacional, engenharia neuromórfica *in silico* e *in vitro*, e a plataforma Brian2. O objetivo é servir como uma referência rápida para o leitor, clarificando a terminologia específica do campo e facilitando a compreensão do texto principal. Os termos estão organizados em ordem alfabética.*

---

*   **Adaptação (Neuronal):** Fenômeno pelo qual a taxa de disparo de um neurônio diminui ao longo do tempo em resposta a um estímulo constante ou repetitivo. Frequentemente mediada por correntes iônicas lentas (e.g., $K^+$ ativadas por $Ca^{2+}$ ou $Na^+$). Modelos como AdEx e Izhikevich podem exibir adaptação.

*   **Address-Event Representation (AER):** Esquema de comunicação assíncrono usado em hardware neuromórfico, onde a ocorrência de um spike é codificada por um "evento" digital contendo o endereço (identificador) do neurônio que disparou. Permite comunicação esparsa e eficiente em sistemas distribuídos.

*   **AdEx (Adaptive Exponential Integrate-and-Fire):** Modelo neuronal de duas variáveis que combina a dinâmica exponencial de disparo do EIF (Exponential Integrate-and-Fire) com uma variável de adaptação linear. É computacionalmente eficiente e capaz de reproduzir diversos padrões de disparo, incluindo adaptação e bursting.

*   **Aprendizado Hebbiano:** Princípio fundamental de plasticidade sináptica postulado por Donald Hebb, geralmente resumido como "neurônios que disparam juntos, conectam-se". Sugere que a força de uma sinapse é aumentada se a atividade pré e pós-sináptica estiverem correlacionadas temporalmente.

*   **Aprendizado por Reforço (Reinforcement Learning - RL):** Paradigma de aprendizado onde um agente aprende a tomar ações em um ambiente para maximizar um sinal de recompensa cumulativo, geralmente através de tentativa e erro. Pode ser implementado em SNNs usando plasticidade modulada por recompensa (e.g., R-STDP).

*   **Aprendizado Supervisionado:** Paradigma de aprendizado onde a rede é treinada com exemplos de entrada e suas saídas corretas correspondentes (rótulos). O objetivo é aprender um mapeamento da entrada para a saída. Backpropagation é o método dominante em ANNs; métodos como Gradiente Substituto são usados para SNNs.

*   **Aprendizado Não Supervisionado:** Paradigma de aprendizado onde a rede aprende a encontrar estrutura ou padrões nos dados de entrada sem rótulos explícitos. STDP é um exemplo de regra que pode mediar aprendizado não supervisionado em SNNs (e.g., extração de características, clustering).

*   **Arquitetura de Von Neumann:** Modelo computacional dominante caracterizado pela separação física entre a unidade central de processamento (CPU) e a memória principal, conectadas por um barramento. Sofre do "gargalo de Von Neumann" devido à necessidade de transferir dados e instruções.

*   **Arranjo de Microeletrodos (Microelectrode Array - MEA):** Dispositivo contendo uma grade de microeletrodos usado para registrar a atividade elétrica extracelular (spikes, LFPs) de culturas neurais *in vitro* (incluindo organoides) ou tecido cerebral *in vivo*, e também para aplicar estimulação elétrica.

*   **Assembloide:** Estrutura *in vitro* criada pela fusão ou co-cultura de dois ou mais organoides de diferentes identidades regionais ou tipos celulares (e.g., cortical dorsal e ventral). Permite modelar interações e migrações inter-regionais que não ocorrem em organoides únicos.

*   **Avalanches Neuronais:** Cascatas de atividade neural (spikes ou eventos LFP) cuja distribuição de tamanho e duração segue uma lei de potência. São consideradas uma assinatura de operação em regime de criticalidade em redes neurais.

*   **Axônio:** Prolongamento único e tipicamente longo de um neurônio, especializado na condução rápida e regenerativa de potenciais de ação do soma para os terminais sinápticos. Pode ser mielinizado para aumentar a velocidade de condução.

*   **Backpropagation:** Algoritmo de aprendizado supervisionado fundamental para treinar redes neurais artificiais (ANNs) profundas. Utiliza a regra da cadeia para calcular o gradiente da função de erro em relação aos pesos da rede e ajustá-los via descida de gradiente. Sua aplicação direta a SNNs é dificultada pela natureza não diferenciável do spike.

*   **Barra Cruzada (Crossbar Array):** Arquitetura de interconexão densa onde um conjunto de fios paralelos (e.g., horizontais) cruza perpendicularmente outro conjunto de fios paralelos (e.g., verticais). Dispositivos (como memristores) são colocados nas interseções, permitindo alta densidade e implementação de computação em memória (VMM).

*   **Benchmark:** Uma tarefa, dataset e conjunto de métricas padronizados usados para avaliar e comparar o desempenho de diferentes sistemas computacionais (hardware ou algoritmos) de forma justa e objetiva.

*   **Brian2:** Biblioteca Python de software livre para simulação de redes neurais de spikes (SNNs). Caracterizada pela flexibilidade na definição de modelos via equações e pelo suporte integrado a unidades físicas. Ferramenta central utilizada nos exemplos *in silico* deste livro.

*   **Bursting:** Padrão de disparo neuronal caracterizado por rajadas (bursts) de múltiplos potenciais de ação emitidos em alta frequência, intercaladas por períodos de quiescência ou disparos tônicos de baixa frequência. Modelos como Izhikevich e AdEx podem exibir bursting.

*   **Células-Tronco Pluripotentes (CTPs):** Células indiferenciadas que têm a capacidade de se diferenciar em qualquer tipo celular dos três folhetos germinativos (ectoderma, mesoderma, endoderma). Incluem Células-Tronco Embrionárias (CTEs) e Células-Tronco Pluripotentes Induzidas (iPSCs). São a base para a geração de organoides.

*   **Células-Tronco Pluripotentes Induzidas (iPSCs):** Células-tronco pluripotentes geradas artificialmente a partir de células somáticas adultas (e.g., pele, sangue) através da reprogramação genética (introdução de fatores de transcrição específicos). Permitem criar modelos celulares específicos do paciente.

*   **Computação em Memória (Compute-in-Memory - CIM / Processing-in-Memory - PIM):** Paradigma computacional que busca superar o gargalo de Von Neumann realizando operações computacionais (e.g., multiplicação vetor-matriz) diretamente dentro da matriz de memória (geralmente usando NVMs), onde os dados (pesos) estão armazenados, minimizando a movimentação de dados.

*   **Computação Neuromórfica:** Abordagem de computação e design de hardware inspirada na estrutura e função do cérebro biológico, visando emular seus princípios de processamento (paralelismo, baseado em eventos, co-localização memória-processamento) para alcançar maior eficiência energética, velocidade (em certas tarefas) e adaptabilidade do que a arquitetura de Von Neumann.

*   **Conectividade Efetiva (Effective Connectivity - EC):** Descreve a influência causal direcionada que uma unidade neural exerce sobre outra. Métodos para inferi-la (e.g., GC, TE, GLM) tentam ir além das correlações estatísticas para identificar interações direcionadas.

*   **Conectividade Funcional (Functional Connectivity - FC):** Descreve as dependências estatísticas (correlações) entre a atividade de diferentes unidades neurais, sem implicar causalidade direta. Frequentemente inferida usando correlação cruzada ou teoria da informação.

*   **Correlação Cruzada (Cross-Correlation):** Medida estatística da similaridade entre dois sinais (ou trens de spikes) em função de um atraso temporal aplicado a um deles. O histograma de correlação cruzada (CCH) é usado para analisar relações temporais entre disparos neuronais.

*   **Criticalidade:** Hipótese de que redes neurais podem operar em um estado dinâmico balanceado na fronteira entre ordem (atividade quiescente ou regular) e desordem (atividade caótica ou epiléptica). Operar na criticalidade é teoricamente ótimo para processamento de informação e é frequentemente associado a avalanches neuronais com distribuições de tamanho/duração em lei de potência.

*   **Dendrites:** Extensões ramificadas do corpo celular de um neurônio que recebem a maioria dos inputs sinápticos de outros neurônios. Desempenham um papel crucial na integração de sinais, que pode ser passiva (teoria do cabo) ou ativa (com canais dependentes de voltagem).

*   **Depressão de Longo Prazo (Long-Term Depression - LTD):** Diminuição persistente (horas ou mais) na força de uma sinapse, geralmente induzida por padrões específicos de atividade de baixa frequência ou por temporização pós-antes-de-pré na STDP. Mecanismo complementar à LTP para aprendizado e refinamento de circuitos.

*   **Dynamic Vision Sensor (DVS):** Tipo de sensor de visão neuromórfico baseado em eventos cujos pixels respondem assincronamente a mudanças na intensidade luminosa, gerando um fluxo esparso de eventos ON/OFF.

*   **Eficiência Energética:** Métrica de desempenho computacional que relaciona a quantidade de computação realizada (e.g., operações, inferências) à energia consumida. Um objetivo chave da computação neuromórfica é alcançar alta eficiência energética (e.g., medida em SOPs/Joule).

*   **Entropia de Transferência (Transfer Entropy - TE):** Medida de teoria da informação que quantifica o fluxo de informação direcionado de um processo para outro. Usada para inferir conectividade efetiva, é não-paramétrica, mas exige muitos dados.

*   **Esparsidade (Sparsity):** Propriedade de sinais ou representações onde apenas uma pequena fração dos componentes (e.g., neurônios ativos, coeficientes não-zero) carrega informação significativa em um dado momento. A atividade esparsa em SNNs contribui para a eficiência energética.

*   **Espinhas Dendríticas (Dendritic Spines):** Pequenas protrusões na superfície das dendrites de muitos neurônios (especialmente piramidais) que são os principais locais de recepção de sinapses excitatórias. São estruturas plásticas importantes para aprendizado e memória.

*   **Estado da Arte (State of the Art - SOTA):** Refere-se às tecnologias, métodos ou níveis de desempenho mais avançados e recentes em um determinado campo de pesquisa ou desenvolvimento.

*   **Estimulação Elétrica:** Método de fornecer input a tecido neural aplicando pulsos de corrente ou voltagem através de microeletrodos (e.g., em um MEA). É tecnicamente estabelecido, mas sofre de baixa especificidade celular e artefatos.

*   **Estimulação Óptica (Optogenética):** Método de controlar a atividade neural usando luz para ativar opsinas fotossensíveis (como ChR2, NpHR) expressas geneticamente em neurônios específicos. Permite alta precisão temporal, espacial e celular, mas requer modificação genética e sistemas ópticos.

*   **Fenda Sináptica:** O pequeno espaço extracelular (~20-40 nm) entre a membrana pré-sináptica e a membrana pós-sináptica em uma sinapse química, através do qual os neurotransmissores se difundem.

*   **Field-Programmable Gate Array (FPGA):** Circuito integrado que pode ser reprogramado pelo usuário após a fabricação para implementar funções lógicas digitais customizadas. Usado para prototipar hardware digital, incluindo arquiteturas neuromórficas, com maior flexibilidade que ASICs, mas geralmente menor desempenho/eficiência.

*   **Gargalo de Von Neumann (Von Neumann Bottleneck):** A limitação de desempenho em arquiteturas de Von Neumann causada pela largura de banda finita do barramento que conecta a CPU e a memória principal, restringindo a taxa de transferência de dados e instruções.

*   **Gerador Central de Padrões (Central Pattern Generator - CPG):** Circuito neural (biológico ou artificial) capaz de produzir padrões de atividade rítmica e coordenada de forma autônoma, sem input sensorial rítmico. Usado para controlar comportamentos motores como locomoção.

*   **Glia:** População de células não-neuronais no sistema nervoso (incluindo astrócitos, oligodendrócitos, micróglia) que desempenham papéis cruciais de suporte, metabólicos, imunológicos e na modulação da função sináptica e plasticidade. Sua presença e maturação são importantes em organoides.

*   **Gradiente Substituto (Surrogate Gradient - SG):** Técnica usada para treinar SNNs com métodos baseados em gradiente (como backpropagation). Substitui a derivada da função de disparo do spike (que é zero ou indefinida) por uma função "substituta" suave e diferenciável durante o passo backward do treinamento, permitindo que o gradiente flua através da rede.

*   **Hardware Neuromórfico:** Circuitos integrados (chips) projetados especificamente para implementar modelos de neurônios e sinapses e executar computação inspirada nos princípios do cérebro, visando alta eficiência energética e/ou velocidade para tarefas específicas. Pode ser analógico, digital ou de modo misto.

*   **Hodgkin-Huxley (Modelo de):** Modelo matemático biofisicamente detalhado que descreve a geração do potencial de ação com base na dinâmica dependente de voltagem das condutâncias iônicas de Sódio ($Na^+$) e Potássio ($K^+$). Consiste em um sistema de quatro ODEs acopladas.

*   **Imagem de Cálcio:** Técnica óptica para monitorar a atividade neural medindo mudanças na fluorescência de indicadores sensíveis ao cálcio (corantes ou GECIs), que servem como proxy para o influxo de $Ca^{2+}$ associado a potenciais de ação ou atividade sináptica. Oferece resolução de célula única, mas resolução temporal limitada.

*   **Implementação *In Silico*:** Abordagens de computação neuromórfica que utilizam silício (hardware eletrônico) ou simulações em computadores digitais convencionais. Contrasta com abordagens *in vitro*.

*   **Implementação *In Vitro*:** Abordagens de computação neuromórfica que utilizam substratos neurais biológicos cultivados em laboratório (e.g., culturas de neurônios, organoides cerebrais) como o próprio meio computacional, frequentemente interfaceados com eletrônicos.

*   **Inferência Baseada em Modelos (Model-Based Inference - MBI):** Abordagem para inferir parâmetros ou estrutura de um sistema (como conectividade neural) ajustando os parâmetros de um modelo mecanístico simulável (e.g., uma SNN em Brian2) para que ele reproduza da melhor forma os dados observados experimentalmente.

*   **Integra-e-Dispara (Integrate-and-Fire - IF):** Classe de modelos neuronais simplificados onde o neurônio integra a corrente de entrada e dispara um spike quando seu potencial de membrana atinge um limiar, sendo então resetado. A variante mais comum é o LIF (Leaky Integrate-and-Fire).

*   **Interface Cérebro-Computador (Brain-Computer Interface - BCI):** Sistema que mede a atividade cerebral (ou de tecido neural) e a traduz em comandos para um dispositivo externo, ou que fornece input sensorial/controle diretamente ao cérebro/tecido neural através de estimulação. No contexto deste livro, refere-se às interfaces com organoides.

*   **Interneurônio:** Neurônio que se conecta primariamente a outros neurônios dentro do mesmo circuito ou região local, em contraste com neurônios de projeção que enviam axônios para regiões distantes. Muitos interneurônios são inibitórios (GABAérgicos) e desempenham papéis cruciais na regulação da dinâmica da rede.

*   **Izhikevich (Modelo de):** Modelo neuronal fenomenológico de duas variáveis (potencial de membrana $v$ e variável de recuperação $u$) notável por sua eficiência computacional e capacidade de reproduzir uma ampla gama de padrões de disparo neuronal (RS, IB, CH, FS, etc.) ajustando apenas quatro parâmetros.

*   **Leaky Integrate-and-Fire (LIF):** Modelo IF que inclui um termo de "vazamento" (leak) na dinâmica sub-limiar, fazendo com que o potencial de membrana retorne passivamente ao potencial de repouso na ausência de input. É um dos modelos de spiking mais simples e amplamente utilizados.

*   **Lei de Dale:** Princípio (com algumas exceções conhecidas) que afirma que um neurônio libera o mesmo tipo de neurotransmissor(es) em todas as suas sinapses. Consequentemente, um neurônio é tipicamente classificado como puramente excitatório ou puramente inibitório em suas ações pós-sinápticas.

*   **Lei de Hebb:** Ver *Aprendizado Hebbiano*.

*   **Local Field Potential (LFP):** Potencial elétrico extracelular de baixa frequência (< 300 Hz) registrado por microeletrodos, que reflete a atividade sináptica e sub-limiar somada de uma população neuronal local. Contém informação sobre oscilações de rede e sincronia populacional.

*   **Long-Term Depression (LTD):** Ver *Depressão de Longo Prazo*.

*   **Long-Term Potentiation (LTP):** Ver *Potenciação de Longo Prazo*.

*   **Memória Associativa:** Tipo de memória onde a informação é armazenada e recuperada com base em relações (associações) entre itens ou padrões, permitindo recuperação baseada em conteúdo e conclusão de padrões. Contrastada com memória endereçada por endereço.

*   **Memória Endereçável por Conteúdo (Content-Addressable Memory - CAM):** Sistema de memória onde a recuperação de dados é iniciada fornecendo parte do conteúdo (uma chave ou pista), e a memória retorna o dado completo associado. Memórias associativas implementam CAM.

*   **Memristor:** Dispositivo eletrônico passivo de dois terminais cuja resistência depende do histórico da carga ou fluxo magnético que passou por ele. Dispositivos RRAM são frequentemente considerados memristores. Vistos como candidatos a sinapses artificiais.

*   **Modelo Comportamental (NVM):** Modelo matemático ou computacional que descreve o comportamento elétrico observado de um dispositivo de memória não volátil (e.g., RRAM, PCM) em resposta a estímulos elétricos, focando nas relações input-output e na evolução do estado interno (resistência), sem necessariamente modelar a física subjacente em detalhe. Usado para simulação de circuitos e sistemas.

*   **Muro da Energia (Power Wall):** Limitação fundamental no desempenho computacional devido ao consumo de energia e à capacidade de dissipação de calor em circuitos integrados, que restringe aumentos na frequência de clock ou densidade de integração.

*   **Muro da Memória (Memory Wall):** Limitação de desempenho em arquiteturas de Von Neumann causada pela crescente disparidade entre a velocidade dos processadores e a latência/largura de banda da memória principal.

*   **Não Volátil (Memória):** Propriedade de um dispositivo de memória de reter a informação armazenada mesmo quando a fonte de energia é removida. Característica essencial de NVMs como RRAM, PCM, MRAM e Flash.

*   **Neocórtex:** A camada externa e evolutivamente mais recente do cérebro dos mamíferos, responsável por funções cognitivas superiores. Caracterizado por uma estrutura em seis camadas e organização colunar.

*   **Network (Objeto Brian2):** Objeto no Brian2 que agrupa todos os componentes de uma simulação (NeuronGroups, Synapses, Monitores, etc.) que devem ser simulados juntos. Gerencia o cronograma de execução da simulação.

*   **NeuronGroup (Objeto Brian2):** Objeto fundamental no Brian2 que representa uma população de $N$ neurônios homogêneos descritos pelo mesmo modelo matemático (equações, limiar, reset).

*   **Neuromórfico:** Adjetivo que descreve sistemas (hardware ou software) ou princípios de computação inspirados na estrutura ou função do sistema nervoso biológico.

*   **Neurônio:** Célula eletricamente excitável do sistema nervoso, considerada a unidade fundamental de processamento de informação. Tipicamente consiste em soma, dendrites e axônio.

*   **Neurotransmissor:** Molécula química liberada por um neurônio pré-sináptico na fenda sináptica para transmitir um sinal a um neurônio pós-sináptico, ligando-se a receptores específicos. Exemplos: Glutamato (excitatório), GABA (inibitório), Acetilcolina, Dopamina.

*   **Nó de Ranvier:** Pequenas lacunas na bainha de mielina que isola os axônios, onde a membrana axonal está exposta e contém alta densidade de canais iônicos dependentes de voltagem. Essenciais para a condução saltatória rápida dos potenciais de ação.

*   **NPU (Neural Processing Unit) Neuromórfica:** Termo genérico para um processador ou acelerador de hardware projetado especificamente para implementar e executar modelos de redes neurais de spikes ou outros algoritmos neuromórficos de forma eficiente. Exemplos incluem Loihi, Akida.

*   **Optogenética:** Conjunto de técnicas que combinam métodos ópticos e genéticos para controlar ou monitorar a atividade de neurônios específicos (ou outras células) que foram geneticamente modificados para expressar proteínas sensíveis à luz (opsinas).

*   **Organoide Cerebral:** Estrutura tridimensional auto-organizada cultivada *in vitro* a partir de células-tronco pluripotentes (CTPs), que recapitula aspectos da arquitetura celular, organização tecidual e desenvolvimento inicial do cérebro humano. Usado como modelo para estudo e potencialmente como substrato computacional.

*   **Oscilações Neurais:** Atividade rítmica ou periódica na atividade elétrica de neurônios individuais ou populações (visível em spikes ou LFPs). Ocorrem em diferentes bandas de frequência (delta, teta, alfa, beta, gama) e estão associadas a diversas funções cognitivas e estados cerebrais.

*   **PCM (Phase-Change Memory):** Tipo de memória não volátil que utiliza a transição reversível de um material calcogeneto entre estados amorfo (alta resistência) e cristalino (baixa resistência), controlada por aquecimento via pulsos elétricos. Candidata a sinapse artificial analógica.

*   **Plasticidade Homeostática:** Processos regulatórios lentos que mantêm a atividade neuronal dentro de uma faixa funcional estável, agindo para compensar mudanças na atividade induzidas por plasticidade hebbiana ou alterações no input. Inclui escalonamento sináptico e plasticidade intrínseca.

*   **Plasticidade Intrínseca (IP):** Modificação dependente de atividade das propriedades de disparo intrínsecas de um neurônio (e.g., limiar, adaptação), geralmente através da regulação da expressão ou função de canais iônicos. Contribui para a homeostase da rede.

*   **Plasticidade Sináptica:** Capacidade das sinapses de alterar sua força (peso ou eficácia) em resposta à atividade neural. É considerada a base celular fundamental do aprendizado e da memória. Inclui formas de curto prazo (STP) e de longo prazo (LTP, LTD, STDP).

*   **Plasticidade Estrutural:** Mudanças físicas na estrutura da rede neural, como crescimento/retração de axônios, dendrites e espinhas dendríticas, ou formação/eliminação de sinapses. Ocorre em escalas de tempo mais lentas e é influenciada pela atividade e fatores de crescimento.

*   **Potencial de Ação (PA / Spike):** Sinal elétrico fundamental para comunicação neural de longa distância. É uma despolarização rápida, transiente, regenerativa e de amplitude estereotipada ("tudo ou nada") da membrana neuronal, que se propaga ao longo do axônio.

*   **Potencial de Campo Local (Local Field Potential - LFP):** Ver *Local Field Potential*.

*   **Potencial de Membrana ($V_m$):** Diferença de potencial elétrico através da membrana plasmática de uma célula. Em neurônios, flutuações em $V_m$ (PSPs, PAs) constituem a base da sinalização elétrica. O potencial de repouso é o $V_m$ na ausência de estímulos significativos.

*   **Potencial Pós-Sináptico (PSP):** Mudança transitória no potencial de membrana da célula pós-sináptica causada pela liberação de neurotransmissor na sinapse. Pode ser Excitatório (EPSP, despolarizante) ou Inibitório (IPSP, geralmente hiperpolarizante ou estabilizador).

*   **Potenciação de Longo Prazo (Long-Term Potentiation - LTP):** Aumento persistente (horas ou mais) na força de uma sinapse, geralmente induzido por estimulação de alta frequência ou por temporização pré-antes-de-pós na STDP. Considerado um mecanismo celular chave para a formação de memórias.

*   **Processamento Baseado em Eventos (Event-Driven Processing):** Paradigma computacional onde a computação e/ou comunicação ocorrem apenas em resposta a eventos discretos e assíncronos (como spikes), em vez de serem ditadas por um clock global. Contribui para a eficiência energética de sistemas neuromórficos com atividade esparsa.

*   **Python:** Linguagem de programação de alto nível, interpretada e de propósito geral, amplamente utilizada em computação científica, análise de dados e aprendizado de máquina. Brian2 é primariamente uma biblioteca Python.

*   **Receptor (Neurotransmissor):** Proteína na membrana pós-sináptica (ou pré-sináptica) que se liga especificamente a um neurotransmissor, desencadeando uma resposta na célula pós-sináptica. Podem ser Ionotrópicos (canais iônicos diretos) ou Metabotrópicos (acoplados a proteínas G).

*   **Reconstrução de Rede:** Processo de inferir a estrutura de conectividade (funcional ou efetiva) de uma rede neural a partir de observações de sua atividade.

*   **Rede Neural de Spikes (Spiking Neural Network - SNN):** Tipo de rede neural artificial que utiliza neurônios de spiking (como LIF, AdEx, Izhikevich) que se comunicam através de eventos discretos (spikes). Considerada mais biologicamente realista e potencialmente mais eficiente energeticamente do que ANNs tradicionais para certas tarefas, especialmente em hardware neuromórfico.

*   **Reservoir Computing (RC):** Paradigma de computação para processamento de sinais temporais que utiliza uma rede recorrente fixa (o "reservatório") com dinâmica rica para projetar o input em um espaço de estados de alta dimensão, a partir do qual a saída desejada é lida por uma camada de saída linear treinável. Substratos biológicos como organoides podem potencialmente atuar como reservatórios.

*   **RRAM (Resistive Random-Access Memory):** Tipo de memória não volátil baseada na comutação reversível da resistência de um material dielétrico (geralmente óxido metálico) entre estados de alta e baixa resistência, tipicamente através da formação/ruptura de filamentos condutores. Candidata a implementar sinapses memristivas.

*   **`run()` (Função Brian2):** Função (ou método do objeto `Network`) que executa a simulação de uma rede Brian2 por uma duração especificada.

*   **Simulação *In Silico*:** Uso de computadores digitais para simular o comportamento de sistemas físicos ou biológicos (neste caso, SNNs) através da resolução numérica de modelos matemáticos.

*   **Sinapse:** Junção especializada onde um neurônio transmite um sinal para outro neurônio ou célula efetuadora. Pode ser química (via neurotransmissores) ou elétrica (via junções comunicantes). A força (peso) das sinapses químicas é frequentemente modificável (plástica).

*   **Sincronia Neural:** Ocorrência temporalmente correlacionada de disparos (spikes) em dois ou more neurônios. Pode ocorrer em diferentes escalas de tempo (precisa, de rajadas, de fase) e é considerada importante para codificação e comunicação neural.

*   **Sistema Híbrido Bio-Silício:** Sistema computacional integrado que combina componentes biológicos vivos (e.g., organoides) com componentes eletrônicos *in silico* (e.g., interfaces, processadores), estabelecendo uma comunicação bidirecional em loop fechado.

*   **Soma:** O corpo celular de um neurônio, contendo o núcleo e a maior parte da maquinaria metabólica. Integra sinais das dendrites.

*   **Spike:** Termo comum para *Potencial de Ação*.

*   **Spike Sorting:** Processo computacional aplicado a registros MEA extracelulares para distinguir e classificar as formas de onda de spikes detectadas em um único eletrodo, atribuindo-as a diferentes neurônios individuais putativos (single-units).

*   **SpikeMonitor (Objeto Brian2):** Objeto no Brian2 usado para registrar os tempos e índices dos neurônios que disparam em um `NeuronGroup` durante uma simulação.

*   **STDP (Spike-Timing-Dependent Plasticity):** Forma de plasticidade sináptica hebbiana onde a magnitude e o sinal da mudança no peso sináptico dependem da ordem temporal relativa e do intervalo preciso (na escala de milissegundos) entre os spikes pré e pós-sinápticos.

*   **StateMonitor (Objeto Brian2):** Objeto no Brian2 usado para registrar a evolução temporal de variáveis de estado contínuas (e.g., potencial de membrana, correntes de adaptação) de `NeuronGroup`s ou `Synapses` durante uma simulação.

*   **Synapses (Objeto Brian2):** Objeto no Brian2 que representa as conexões sinápticas entre um grupo de neurônios pré-sinápticos e um grupo pós-sináptico. Define o modelo sináptico (incluindo peso, dinâmica, plasticidade) e a conectividade.

*   **Taxa de Disparo (Firing Rate):** Medida da frequência com que um neurônio dispara potenciais de ação, geralmente expressa em Hertz (Hz) ou spikes por segundo. Pode ser calculada como uma média ao longo do tempo ou como uma taxa instantânea.

*   **Unidades Físicas (Brian2):** Sistema integrado no Brian2 que requer a especificação explícita de unidades físicas (e.g., mV, ms, nA) para todas as quantidades e realiza verificação automática de consistência dimensional nas equações, promovendo a correção e legibilidade dos modelos.
