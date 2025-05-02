---

# CAPÍTULO 08

# MEMÓRIAS NEUROINSPIRADAS E COMPUTAÇÃO EM MEMÓRIA

---

![imagem](neuro.png)

*Este capítulo aborda um dos desafios mais fundamentais enfrentados pela computação convencional e explora como a inspiração neuromórfica oferece soluções promissoras: a separação entre memória e processamento. Investigaremos as limitações inerentes impostas pelo chamado "Muro da Memória" na arquitetura de Von Neumann, detalhando os custos energéticos e de latência associados à constante movimentação de dados, particularmente exacerbados por algoritmos de inteligência artificial modernos. Em contrapartida, introduziremos os conceitos de memória associativa e endereçável por conteúdo, inspirados na forma como o cérebro armazena e recupera informações baseadas em relações e similaridade, em vez de endereços fixos. O cerne do capítulo residirá na exploração de tecnologias emergentes de memória não volátil – com destaque para memristores/RRAM e PCM – que exibem propriedades análogas às sinapses biológicas e se mostram candidatas ideais para atuar como sinapses artificiais, permitindo a fusão de armazenamento e computação. Discutiremos os princípios operacionais desses dispositivos e a necessidade de modelos comportamentais que capturem suas características, incluindo não idealidades, para fins de simulação e design. Finalmente, através de um estudo de caso detalhado implementado em Brian2, demonstraremos conceitualmente como os princípios da memória associativa podem ser emulados utilizando regras de plasticidade sináptica padrão, ilustrando a capacidade de armazenamento e recuperação de padrões em uma rede neural de spikes, servindo como uma ponte entre os conceitos biológicos e as futuras implementações de hardware baseadas em computação em memória.*

---

**8.1. Limitações da Separação Memória-Processamento (Memory Wall)**

Conforme introduzido no Capítulo 1, a arquitetura de Von Neumann, apesar de seu sucesso histórico e ubiquidade, sofre de uma limitação arquitetônica fundamental: a separação física entre as unidades onde os dados e instruções são armazenados (memória principal, tipicamente DRAM) e as unidades onde são processados (CPU, GPU, ou outros aceleradores). Essa separação impõe a necessidade de um fluxo constante de informação através de um barramento de comunicação (bus) que interliga esses componentes (Backus, 1978; Wulf & McKee, 1995). Embora essa organização tenha permitido flexibilidade programática, a crescente disparidade entre a velocidade de processamento e a velocidade de acesso à memória principal culminou no que é conhecido como o **"Muro da Memória" (Memory Wall)**, um gargalo que limita severamente o desempenho e a eficiência energética dos sistemas computacionais modernos, especialmente para aplicações intensivas em dados.

A manifestação mais direta do Muro da Memória é a **latência de acesso**. O tempo necessário para que um processador solicite um dado da memória principal e o receba pode ser ordens de magnitude maior (centenas de ciclos de clock) do que o tempo necessário para realizar uma operação aritmética simples (um ou poucos ciclos de clock) sobre dados que já estejam em seus registradores ou caches locais (Sze et al., 2017). Durante esse tempo de espera (stall), o processador permanece ocioso, subutilizando sua capacidade computacional e degradando o desempenho geral do sistema. Hierarquias de memória com múltiplos níveis de cache (L1, L2, L3), que são memórias SRAM menores, mais rápidas e mais próximas do processador, foram introduzidas para mitigar esse problema, armazenando dados e instruções acessados frequentemente. No entanto, a eficácia das caches depende crucialmente da localidade espacial e temporal dos acessos à memória – ou seja, da probabilidade de que dados próximos aos acessados recentemente ou os próprios dados recentes sejam acessados novamente em breve. Para muitas aplicações modernas, como o treinamento de redes neurais profundas com modelos de bilhões de parâmetros ou a análise de grandes grafos de dados, os padrões de acesso à memória podem ser irregulares ou o conjunto de trabalho ativo (working set) pode exceder em muito a capacidade das caches, resultando em frequentes "cache misses" e na necessidade de recorrer à lenta memória principal.

Além da latência, a **largura de banda da memória** (a taxa máxima na qual os dados podem ser transferidos entre o processador e a memória principal) também constitui um gargalo significativo. Mesmo com tecnologias de DRAM e interfaces de barramento cada vez mais rápidas (e.g., DDR5, HBM - High Bandwidth Memory), a demanda por largura de banda em aplicações como processamento gráfico de alta resolução, simulações científicas e, novamente, treinamento de IA, frequentemente supera a capacidade disponível, limitando a taxa de transferência de dados e, consequentemente, o desempenho.

Talvez o aspecto mais crítico do Muro da Memória na era atual, onde o consumo energético (o "Muro da Energia") é uma preocupação primordial, seja o **custo energético da movimentação de dados**. Estimativas rigorosas baseadas em tecnologias de semicondutores contemporâneas indicam que a energia consumida para buscar um operando de 64 bits da memória DRAM externa pode ser de 2 a 3 ordens de magnitude maior do que a energia para realizar uma operação de ponto flutuante de 64 bits (FP64 ADD ou MULT) dentro do processador (Horowitz, 2014, conforme citado em Chen et al., 2021; Murmann & Hoefflinger, 2020). Por exemplo, uma busca em DRAM pode custar da ordem de 10-100 nanoJoules (nJ), enquanto uma operação FP64 pode custar da ordem de 10-100 picoJoules (pJ). Isso significa que, em algoritmos onde a razão entre acessos à memória e operações computacionais (bytes/FLOP) é alta, a energia gasta na movimentação de dados pode dominar completamente o orçamento energético total, excedendo largamente a energia gasta na computação útil.

**Figura 8.1**: Custo Energético Comparativo: Computação vs. Movimentação de Dados. Gráfico de barras (escala logarítmica no eixo Y) mostrando a energia aproximada (em pJ) para diferentes operações em tecnologia CMOS moderna: (a) Operação aritmética simples (e.g., ADD 8-bit ~0.03 pJ, MULT 8-bit ~0.2 pJ, FP64 ADD ~20 pJ). (b) Acesso a memória local (e.g., Registrador ~1 pJ, Cache L1 SRAM ~5 pJ, Cache L3 SRAM ~100 pJ). (c) Acesso à memória principal externa (e.g., DRAM LPDDR4 ~1000-10000 pJ). Destaca a disparidade de ordens de magnitude entre computação e acesso à DRAM. (Fonte de dados conceitual baseada em Horowitz 2014, Sze 2017, etc.)

As implicações para a Inteligência Artificial e o Aprendizado de Máquina são profundas. O treinamento de redes neurais profundas (DNNs) envolve tipicamente a apresentação repetida de grandes conjuntos de dados e a atualização de milhões ou bilhões de parâmetros (pesos sinápticos) armazenados na memória. Operações fundamentais como a multiplicação de matrizes (e.g., em camadas convolucionais ou totalmente conectadas) e a aplicação de funções de ativação requerem o acesso constante a esses pesos e aos estados das ativações neuronais. Durante o backpropagation, gradientes também precisam ser calculados, armazenados e acessados. A inferência com modelos grandes também pode ser limitada pela memória. Consequentemente, tanto o tempo de treinamento/inferência quanto o consumo energético são fortemente dominados pela comunicação memória-processador. O gargalo de Von Neumann limita a taxa de processamento efetiva e contribui significativamente para os custos operacionais e a pegada de carbono associados à IA em larga escala (Patterson et al., 2021; Strubell et al., 2019).

Essa constatação motiva uma mudança de paradigma arquitetônico: em vez de trazer os dados para a unidade de processamento, por que não levar o processamento para onde os dados estão armazenados? Essa é a ideia central por trás da **Computação em Memória (Compute-in-Memory - CIM)** ou **Processamento em Memória (Processing-in-Memory - PIM)** (Sebastian et al., 2020; Ielmini & Wong, 2018; Mutlu et al., 2019). A computação neuromórfica, com sua inspiração na co-localização de processamento (neurônios) e memória (sinapses) no cérebro, alinha-se naturalmente com este paradigma. Ao implementar sinapses artificiais usando dispositivos que podem armazenar o peso e, ao mesmo tempo, modular o sinal que passa por eles, torna-se possível realizar operações computacionais chave (como a multiplicação vetor-matriz, que é a base de muitas operações neurais) diretamente dentro da matriz de memória, eliminando ou reduzindo drasticamente a necessidade de movimentar os pesos sinápticos para unidades de processamento externas. Antes de explorarmos os dispositivos que permitem isso, vamos revisitar os conceitos de memória inspirados biologicamente que essas abordagens buscam implementar.

**8.2. Conceitos de Memória Associativa e Endereçável por Conteúdo**

A memória em computadores convencionais opera primariamente com base em **endereços**. Para armazenar ou recuperar um dado, o processador fornece um endereço numérico único que especifica a localização física daquele dado na memória. A informação em si não tem relação intrínseca com seu endereço; o endereço é apenas um ponteiro para a localização.

Em contraste, a memória no cérebro parece operar de forma fundamentalmente diferente, baseada em **associações** e **conteúdo** (Kohonen, 1988; Rolls, 2013). Não recuperamos uma memória fornecendo um "endereço" neural, mas sim ativando uma parte da informação (uma pista, um fragmento de padrão) que, através de associações aprendidas, evoca o restante da memória ou informações relacionadas. Esse tipo de memória é conhecido como **memória associativa**.

Características chave da memória associativa inspirada biologicamente incluem:

*   **Endereçamento por Conteúdo (Content-Addressable Memory - CAM):** A informação é recuperada com base em seu próprio conteúdo ou em parte dele, não em um endereço arbitrário. Dada uma pista (um padrão de entrada parcial ou ruidoso), a memória associativa busca e retorna o padrão armazenado mais similar ou relacionado.
*   **Armazenamento Distribuído:** Memórias individuais não são armazenadas em locais únicos, mas sim codificadas no padrão de força (pesos) de um grande número de conexões sinápticas distribuídas por uma rede neural. A mesma sinapse pode participar da representação de múltiplas memórias.
*   **Robustez:** Devido ao armazenamento distribuído, a memória associativa é inerentemente robusta a danos parciais na rede. A perda de alguns neurônios ou sinapses pode degradar ligeiramente a qualidade da recuperação, mas geralmente não causa perda catastrófica da memória (degradação graciosa).
*   **Conclusão de Padrões (Pattern Completion):** A capacidade de recuperar uma memória completa a partir de uma pista parcial ou corrompida. A rede "preenche" as partes faltantes com base nas associações armazenadas. A região CA3 do hipocampo, com sua alta conectividade recorrente, é frequentemente citada como um exemplo biológico de um sistema auto-associativo capaz de conclusão de padrões (Marr, 1971; Nakazawa et al., 2002).
*   **Separação de Padrões (Pattern Separation):** A capacidade de tornar representações de padrões de entrada similares mais distintas na saída, de modo que possam ser armazenadas e recuperadas sem interferência mútua. O giro denteado do hipocampo é considerado crucial para esta função (Treves & Rolls, 1994; Leutgeb et al., 2007).

**Tipos de Memória Associativa:**

*   **Memória Auto-Associativa:** Armazena um conjunto de padrões e, quando apresentada com uma versão ruidosa ou incompleta de um padrão armazenado, converge para a versão original completa desse mesmo padrão. A rede associa um padrão a si mesmo. Redes de Hopfield são um modelo clássico de memória auto-associativa (Hopfield, 1982).
*   **Memória Hetero-Associativa:** Armazena associações entre pares de padrões diferentes $(X, Y)$. Quando apresentada com o padrão $X$ (ou uma versão ruidosa), a rede recupera o padrão associado $Y$. É útil para tarefas como recuperação de informações baseada em pistas, tradução, ou aprendizado de sequências.

**Implementação com Redes Neurais:**

A ideia fundamental por trás da implementação de memórias associativas em redes neurais é que os padrões a serem armazenados correspondem a estados de atividade estáveis (atratores) da dinâmica da rede. A estrutura de conectividade e os pesos sinápticos são configurados (através de aprendizado hebbiano ou outras regras) de forma que, quando a rede é inicializada com um estado próximo a um padrão armazenado, sua dinâmica a leva a convergir para o atrator correspondente àquele padrão.

*   **Regra de Hebb:** Em sua forma mais simples, a regra de Hebb pode ser usada para armazenar associações. Para armazenar um conjunto de padrões de atividade neuronal $\{ \xi^\mu \}$, onde $\xi_i^\mu$ é a atividade do neurônio $i$ no padrão $\mu$, o peso da sinapse $w_{ij}$ entre o neurônio $j$ (pré) e o neurônio $i$ (pós) pode ser definido como a correlação entre suas atividades ao longo dos padrões:
    $w_{ij} \propto \sum_{\mu} \xi_i^\mu \xi_j^\mu$
    Esta regra fortalece conexões entre neurônios que estão frequentemente co-ativos nos padrões armazenados. Em redes recorrentes, isso cria atratores correspondentes aos padrões $\xi^\mu$.
*   **Redes de Hopfield:** Uma rede recorrente de unidades binárias (e.g., +1/-1) com conexões simétricas ($w_{ij} = w_{ji}$) e sem autoconexões ($w_{ii}=0$), onde os pesos são definidos pela regra de Hebb (ou uma variante), funciona como uma memória auto-associativa. A dinâmica da rede (atualização assíncrona dos estados neuronais) minimiza uma função de "energia", e os padrões armazenados correspondem a mínimos locais dessa energia (Hopfield, 1982). Embora influente, a capacidade de armazenamento das redes de Hopfield clássicas é limitada (~0.14N padrões para N neurônios) e elas podem sofrer com "atratores espúrios".
*   **SNNs como Memórias Associativas:** Redes Neurais de Spikes (SNNs) com conectividade recorrente e plasticidade sináptica (como STDP) podem implementar formas mais biologicamente plausíveis e potencialmente mais poderosas de memória associativa (Palm, 2013; Lansner, 2009). A STDP pode aprender naturalmente as correlações temporais nos padrões de entrada e fortalecer as conexões que suportam a recuperação de padrões. A dinâmica de spiking pode permitir representações mais ricas e processamento temporal.

**Figura 8.2**: Conceito de Memória Associativa. (a) Memória Endereçada por Endereço: Input é um endereço, Output é o dado naquela localização. (b) Memória Endereçada por Conteúdo (Associativa): Input é uma pista (padrão parcial/ruidoso), Output é o padrão completo/associado armazenado mais próximo. Ilustra a recuperação baseada em similaridade e a conclusão de padrões.

O paradigma da memória associativa, com seu armazenamento distribuído e recuperação baseada em conteúdo, oferece uma alternativa atraente ao modelo de memória convencional e se alinha bem com a arquitetura e a plasticidade observadas no cérebro. A questão chave é como implementar eficientemente essa funcionalidade em hardware, o que nos leva aos dispositivos emergentes de memória.

**8.3. Dispositivos Emergentes como Sinapses Artificiais**

A realização prática da computação em memória e das memórias neuroinspiradas depende crucialmente da disponibilidade de dispositivos eletrônicos que possam funcionar como **sinapses artificiais eficientes**. Uma sinapse artificial ideal deveria:

1.  **Armazenar um Peso Analógico:** Ser capaz de representar a força sináptica $w$ como um valor analógico (ou multi-nível) com precisão suficiente.
2.  **Ser Não Volátil:** Reter o valor do peso armazenado mesmo quando a energia é desligada, para memória de longo prazo e baixo consumo estático.
3.  **Ser Modulável (Plástico):** Permitir que o valor do peso seja atualizado de forma incremental e controlada (preferencialmente no local) para implementar regras de aprendizado como STDP.
4.  **Modular o Sinal:** Utilizar o peso armazenado para modular o sinal que passa através da sinapse (e.g., multiplicar a ativação pré-sináptica pelo peso).
5.  **Ser Compacto e Escalável:** Permitir integração em alta densidade, idealmente em arranjos de barra cruzada (crossbar).
6.  **Ser Rápido e Energeticamente Eficiente:** Tanto para leitura (modulação do sinal) quanto para escrita (atualização do peso).

Os transistores CMOS padrão, usados como base da computação digital, não atendem bem a esses requisitos, especialmente a combinação de armazenamento analógico não volátil e modulação local em um dispositivo compacto. Isso levou à exploração intensiva de **dispositivos de memória não volátil (NVM) emergentes**, que exibem naturalmente algumas dessas propriedades desejadas (Wong & Salahuddin, 2015; Yu & Chen, 2016; Ielmini & Wong, 2018). Os candidatos mais promissores incluem:

**1. Memristores e RRAM (Resistive Random-Access Memory):**

O **memristor** (resistor com memória), teorizado por Leon Chua em 1971 e demonstrado experimentalmente pela HP Labs em 2008 (Strukov et al., 2008), é um dispositivo passivo de dois terminais cuja resistência $R$ não é constante, mas depende do histórico da voltagem aplicada ou da corrente que passou por ele (Chua, 1971; Chua & Kang, 1976). Dispositivos RRAM, que exibem comutação resistiva (Resistive Switching - RS), são frequentemente considerados uma classe de memristores.

*   **Princípio de Funcionamento (típico em RRAM de óxido metálico):** Um dispositivo RRAM geralmente consiste em uma fina camada de material dielétrico (e.g., óxidos metálicos como HfO₂, TaOₓ, TiO₂) sanduichada entre dois eletrodos metálicos. A resistência do dispositivo pode ser comutada entre um Estado de Alta Resistência (High Resistance State - HRS ou OFF) e um Estado de Baixa Resistência (Low Resistance State - LRS ou ON) aplicando pulsos de voltagem de polaridade e/ou amplitude apropriadas. Acredita-se que o mecanismo subjacente em muitos casos envolva a formação e ruptura de **filamentos condutores** (constituídos por vacâncias de oxigênio ou íons metálicos) através do dielétrico.
    *   *Operação SET:* Um pulso de voltagem mais alto (tipicamente positivo) forma ou engrossa o filamento, mudando o dispositivo para o LRS.
    *   *Operação RESET:* Um pulso de voltagem de polaridade oposta ou de amplitude diferente (tipicamente negativo ou de maior corrente) rompe ou enfraquece o filamento, retornando o dispositivo para o HRS.
*   **Comutação Analógica vs. Binária:** Dependendo do material e do protocolo de programação, alguns dispositivos RRAM exibem comutação binária (apenas HRS e LRS estáveis), enquanto outros podem exibir **comutação analógica** ou **multi-nível**, onde a resistência pode ser ajustada para valores intermediários de forma gradual aplicando sequências de pulsos de SET/RESET de baixa amplitude ou duração controlada. Esta capacidade analógica é particularmente atraente para representar pesos sinápticos com múltiplos níveis de precisão.
*   **Vantagens como Sinapse Artificial:**
    *   *Não Volatilidade:* O estado de resistência (HRS/LRS/intermediário) é retido sem energia.
    *   *Escalabilidade:* Dispositivos RRAM podem ser fabricados em tamanhos muito pequenos (nanômetros) e integrados em arranjos de barra cruzada de alta densidade.
    *   *Baixo Consumo (Potencial):* A leitura (usando baixas tensões) e a escrita (pulsos curtos) podem ser energeticamente eficientes.
    *   *Compatibilidade CMOS:* Muitos materiais e processos RRAM são compatíveis com a fabricação CMOS padrão.
    *   *Capacidade Analógica (em alguns casos):* Permite representar pesos sinápticos com múltiplos bits de precisão.
*   **Desafios:**
    *   *Variabilidade:* Grande variabilidade nas características de comutação (voltagens SET/RESET, resistências HRS/LRS) entre diferentes dispositivos (device-to-device) e para o mesmo dispositivo ao longo de múltiplos ciclos (cycle-to-cycle) é um dos maiores desafios.
    *   *Não-Linearidade e Assimetria na Atualização:* A mudança na resistência (ou condutância) em resposta a pulsos de programação frequentemente não é linear e pode ser assimétrica (diferente para aumento vs. diminuição da resistência), dificultando a implementação precisa de regras de aprendizado incremental.
    *   *Endurance e Retenção:* O número de ciclos de escrita que um dispositivo pode suportar antes de falhar (endurance) pode ser limitado (embora melhorias estejam sendo feitas). A estabilidade do estado de resistência ao longo do tempo (retenção) também é uma preocupação, especialmente para estados analógicos.
    *   *Problema dos Caminhos Furtivos (Sneak Paths):* Em arranjos de barra cruzada passivos (sem um transistor de seleção por célula), correntes podem fluir por caminhos não desejados através de células vizinhas, interferindo na leitura e escrita da célula selecionada. Soluções incluem dispositivos com alta não-linearidade I-V, retificação intrínseca, ou a adição de um seletor (e.g., transistor, diodo) em série com cada célula RRAM (configuração 1T1R), mas isso aumenta a área e complexidade.

**Figura 8.3**: Dispositivo RRAM/Memristor e Arranjo Crossbar. (a) Estrutura esquemática de um dispositivo RRAM (eletrodo superior / dielétrico / eletrodo inferior). (b) Símbolo esquemático do memristor. (c) Curva I-V característica mostrando histerese e comutação entre HRS e LRS. (d) Arranjo de barra cruzada (crossbar) com dispositivos RRAM nas interseções, permitindo alta densidade e computação em memória via Lei de Ohm/Kirchhoff. Ilustração do problema de sneak path.

**2. PCM (Phase-Change Memory):**

A memória de mudança de fase utiliza materiais calcogenetos (como Ge₂Sb₂Te₅ - GST) que podem existir em dois ou mais estados estáveis com propriedades elétricas (e ópticas) distintas: um estado **amorfo** (desordenado, alta resistência) e um estado **cristalino** (ordenado, baixa resistência). A transição entre esses estados é controlada pela aplicação de pulsos de corrente ou voltagem que aquecem o material.

*   **Princípio de Funcionamento:**
    *   *Operação SET (Cristalização):* Aquecer o material acima de sua temperatura de cristalização, mas abaixo de sua temperatura de fusão, por um tempo suficiente (pulso de amplitude moderada e duração mais longa) permite que os átomos se reorganizem em uma estrutura cristalina de baixa resistência.
    *   *Operação RESET (Amorfização):* Aquecer o material rapidamente acima de sua temperatura de fusão e, em seguida, resfriá-lo muito rapidamente (quenching) (pulso de alta amplitude e curta duração) congela os átomos em um estado amorfo desordenado de alta resistência.
*   **Capacidade Multi-Nível:** Ao controlar cuidadosamente o processo de cristalização (e.g., aplicando múltiplos pulsos SET de baixa amplitude), é possível criar estados parcialmente cristalinos com níveis de resistência intermediários entre o totalmente amorfo (HRS) e o totalmente cristalino (LRS). Isso permite o armazenamento analógico de pesos sinápticos.
*   **Vantagens como Sinapse Artificial:**
    *   *Não Volatilidade:* Ambos os estados (amorfo, cristalino e intermediários) são estáveis sem energia.
    *   *Velocidade de Escrita Relativamente Rápida:* Operações RESET podem ser muito rápidas (dezenas de nanosegundos). SET é mais lento (centenas de ns a microssegundos).
    *   *Boa Escalabilidade:* Dispositivos PCM podem ser fabricados em pequenas dimensões.
    *   *Capacidade Multi-Nível Demonstrada:* A capacidade de armazenar múltiplos bits por célula foi bem estabelecida.
    *   *Alta Endurance (relativa):* Dispositivos PCM podem suportar um número significativo de ciclos de escrita ($10^6$ a $10^9$ ou mais).
*   **Desafios:**
    *   *Deriva da Resistência (Resistance Drift):* A resistência do estado amorfo (e dos estados intermediários) tende a aumentar lentamente ao longo do tempo devido a relaxamentos estruturais, o que pode corromper o valor do peso armazenado. Compensar a deriva é um desafio significativo.
    *   *Alto Consumo de Energia na Escrita (RESET):* A operação RESET requer correntes relativamente altas para fundir o material, o que pode ser um fator limitante de energia, especialmente em arranjos grandes.
    *   *Resistência de Contato e Limitações Térmicas:* O aquecimento e resfriamento repetidos podem levar à degradação do dispositivo ou afetar células vizinhas (crosstalk térmico).

**Figura 8.4**: Dispositivo PCM e Curva R-I. (a) Estrutura esquemática de uma célula PCM típica (com aquecedor). (b) Curva mostrando a resistência final do dispositivo em função da amplitude do pulso de programação aplicado, ilustrando as operações SET e RESET e a possibilidade de estados intermediários.

**3. Outros Dispositivos:**

*   **MRAM (Magnetoresistive RAM) / STT-RAM (Spin-Transfer Torque RAM):** Armazena informação na orientação magnética relativa de duas camadas ferromagnéticas em uma Junção de Túnel Magnético (MTJ). A resistência da MTJ depende dessa orientação relativa (baixa para paralelo, alta para antiparalelo). A comutação é feita por campos magnéticos (MRAM clássica) ou por correntes polarizadas por spin (STT-RAM). Vantagens incluem alta velocidade, endurance virtualmente ilimitada e não volatilidade. Desafios incluem a dificuldade em obter estados analógicos (geralmente binária) e a magnitude da mudança de resistência (baixo TMR - Tunneling Magnetoresistance ratio).
*   **FeFET (Ferroelectric Field-Effect Transistor):** Um transistor onde a camada de gate dielétrica é feita de um material ferroelétrico, cuja polarização pode ser comutada por um campo elétrico e retida sem energia. A polarização ferroelétrica modula a condutividade do canal do transistor, permitindo armazenamento não volátil. Potencial para baixo consumo e integração com lógica CMOS, mas materiais ferroelétricos compatíveis com CMOS (e.g., HfO₂ dopado) e sua confiabilidade ainda estão em desenvolvimento ativo.

**Integração em Arranjos Crossbar para CIM:**

A principal vantagem arquitetônica desses dispositivos NVM de dois terminais (RRAM, PCM) é sua capacidade de serem integrados em **arranjos de barra cruzada (crossbar arrays)** de alta densidade. Neste arranjo, uma matriz de dispositivos é formada nas interseções de um conjunto de linhas metálicas horizontais (wordlines) e um conjunto de linhas verticais (bitlines).

A condutância $G_{ij}$ do dispositivo na interseção da linha $i$ e coluna $j$ pode representar o peso sináptico $w_{ij}$. Para realizar uma operação de **multiplicação vetor-matriz (Vector-Matrix Multiplication - VMM)**, que é fundamental para a propagação de sinais em redes neurais ($y = Wx$), pode-se aplicar tensões $V_j$ (representando o vetor de entrada $x_j$) nas colunas (bitlines) e medir as correntes $I_i$ resultantes nas linhas (wordlines). Pela Lei de Ohm ($I = G V$) e Lei de Kirchhoff (soma das correntes em um nó), a corrente total na linha $i$ é:
$I_i = \sum_j G_{ij} V_j$
Esta operação VMM é realizada *em paralelo* e *no local* onde os pesos $G_{ij}$ estão armazenados, explorando diretamente a física dos dispositivos e do arranjo, e evitando a movimentação massiva de dados para uma unidade de processamento separada. Isso constitui a essência da **Computação em Memória Analógica (Analog CIM)** baseada em NVMs (Sebastian et al., 2020).

**Figura 8.5**: Computação em Memória com Crossbar NVM. Diagrama de um arranjo crossbar NVM realizando VMM. Tensões $V_j$ (input $x$) são aplicadas às colunas. Correntes $I_i$ (output $y$) são coletadas nas linhas, onde $I_i = \sum_j G_{ij} V_j$. Mostra como a operação ocorre in-situ. Circuitos periféricos (DACs para $V_j$, TIAs/ADCs para $I_i$) são necessários.

Embora promissora, a Analog CIM enfrenta desafios práticos significativos decorrentes das não idealidades dos dispositivos (variabilidade, não linearidade) e do arranjo (resistência de linha/coluna, sneak paths), além da necessidade de periféricos analógico-digitais eficientes. Abordagens alternativas exploram CIM digital ou de modo misto, onde a lógica é trazida para mais perto da memória, mas a computação ainda ocorre digitalmente.

A escolha do dispositivo NVM e da arquitetura CIM (analógica, digital, mista) depende dos requisitos específicos da aplicação em termos de precisão, velocidade, energia e necessidade de aprendizado on-chip. No entanto, o potencial desses dispositivos para revolucionar a eficiência da computação neuroinspirada é inegável e impulsiona intensa pesquisa e desenvolvimento.

**8.4. Modelagem Comportamental de Dispositivos de Memória Neuromórfica**

Para projetar e simular sistemas neuromórficos que utilizam dispositivos de memória emergentes como RRAM ou PCM, é essencial ter **modelos computacionais** que capturem com precisão suficiente o comportamento elétrico desses dispositivos. Estes modelos são usados em simuladores de circuito (como SPICE) para o design de hardware e em simuladores de rede neural (como Brian2, ou frameworks mais especializados) para investigar o impacto das características do dispositivo no desempenho do sistema e no aprendizado.

O objetivo da **modelagem comportamental** não é necessariamente replicar a física detalhada do dispositivo (o que seria muito complexo e lento para simulações de larga escala), mas sim capturar as relações chave entre as entradas elétricas (voltagem, corrente, pulsos de programação) e as saídas (estado de resistência/condutância, corrente lida) e como o estado interno (resistência) evolui ao longo do tempo e com a operação.

**Tipos de Modelos:**

*   **Modelos Empíricos / Fenomenológicos:** Baseiam-se em ajustar equações matemáticas ou procedimentos algorítmicos a dados experimentais obtidos da caracterização de dispositivos reais. Eles visam reproduzir o comportamento observado sem necessariamente ter uma derivação física rigorosa para todos os parâmetros.
    *   **Exemplo (Modelo de Limiar para RRAM):** Pode modelar a comutação SET/RESET com base em se a voltagem aplicada excede limiares específicos ($V_{SET}, V_{RESET}$) por um tempo suficiente, talvez com alguma variabilidade estocástica nesses limiares ou nos estados finais HRS/LRS. A mudança gradual de resistência (para comutação analógica) pode ser modelada como uma função da amplitude, duração ou número de pulsos aplicados.
    *   **Exemplo (Modelo de Deriva para PCM):** Pode modelar o aumento da resistência do estado amorfo ao longo do tempo $t$ após a programação (deriva) usando uma lei de potência: $R(t) = R_0 (t/t_0)^\nu$, onde $R_0$ é a resistência inicial em $t_0$ e $\nu$ é o coeficiente de deriva (tipicamente 0.05-0.1) (Ielmini et al., 2011).
    *   **Modelos Baseados em Variáveis de Estado Internas:** Alguns modelos fenomenológicos introduzem variáveis de estado internas que representam, por exemplo, o tamanho ou a posição do filamento condutor em RRAM, ou a fração de fase cristalina em PCM. A dinâmica dessas variáveis de estado é descrita por ODEs que são ajustadas para reproduzir o comportamento elétrico observado (e.g., Modelo de Simmons Tunneling Barrier para RRAM; modelos baseados em nucleação e crescimento para PCM). O **TEAM (Threshold Adaptive Memristor Model)** é um exemplo de modelo baseado em variáveis de estado para memristores que captura muitas características observadas (Kvatinsky et al., 2014).

*   **Modelos Compactos (para Simulação de Circuito):** São modelos projetados para serem usados em simuladores de circuito como SPICE ou Verilog-A. Eles tentam fornecer uma ponte entre a física do dispositivo e o comportamento em nível de circuito, sendo mais detalhados que modelos puramente fenomenológicos, mas ainda computacionalmente tratáveis. Eles são essenciais para o design e verificação de circuitos que integram esses dispositivos.

**Importância de Modelar Não Idealidades:**

Um aspecto crucial da modelagem comportamental é capturar as **não idealidades** dos dispositivos NVM, pois elas podem ter um impacto significativo no desempenho e na robustez dos sistemas neuromórficos baseados neles (Yu, 2018; Indiveri & Sandamirskaya, 2019). As não idealidades mais importantes a serem consideradas incluem:

1.  **Variabilidade:**
    *   *Device-to-Device (D2D):* Diferenças nos parâmetros (limiares de comutação, HRS/LRS, taxa de atualização) entre dispositivos diferentes no mesmo chip, devido a variações de fabricação.
    *   *Cycle-to-Cycle (C2C):* Flutuações nos parâmetros do mesmo dispositivo a cada ciclo de escrita/leitura, devido à natureza estocástica dos mecanismos físicos subjacentes (e.g., formação/ruptura de filamentos).
    A variabilidade pode degradar a precisão da representação dos pesos e a eficácia do aprendizado. Modelos devem incorporar distribuições estatísticas para os parâmetros chave.

2.  **Não-Linearidade e Assimetria na Atualização de Peso:** A mudança na condutância ($\Delta G$) em resposta a pulsos de programação idênticos frequentemente não é constante ao longo da faixa dinâmica do dispositivo (não-linearidade) e pode ser diferente para aumento (potenciação) versus diminuição (depressão) da condutância (assimetria). Isso dificulta a implementação de atualizações de peso lineares e simétricas assumidas por muitos algoritmos de aprendizado. Modelos precisam capturar essas dependências de $G$ e da direção da mudança em $\Delta G$.

3.  **Faixa Dinâmica e Resolução Limitadas:** A razão entre a resistência máxima (HRS) e mínima (LRS) (ON/OFF ratio) define a faixa dinâmica disponível para representar o peso. O número de estados intermediários estáveis e distinguíveis (resolução) também é limitado. Isso restringe a precisão com que os pesos podem ser armazenados.

4.  **Ruído de Leitura:** O processo de leitura da resistência/condutância também está sujeito a ruído, adicionando incerteza ao valor do peso efetivo usado na computação.

5.  **Endurance Limitada:** Os dispositivos só podem suportar um número finito de ciclos de escrita antes de degradarem ou falharem. Isso pode ser uma limitação para algoritmos que exigem atualizações frequentes de peso durante o aprendizado.

6.  **Retenção e Deriva:** A capacidade de reter o estado de resistência ao longo do tempo pode ser limitada (retenção), especialmente para estados analógicos. A deriva da resistência em PCM é um exemplo específico e importante de problema de retenção.

**Integração em Simuladores de SNN:**

Incorporar modelos comportamentais detalhados de dispositivos NVM diretamente em simuladores de SNN de larga escala como Brian2 pode ser desafiador. Brian2 é otimizado para resolver as ODEs dos neurônios e sinapses (assumindo frequentemente plasticidade idealizada), mas não possui modelos embutidos para RRAM/PCM. No entanto, é possível:

*   **Implementar Modelos Simplificados via `Synapses`:** Pode-se definir regras de atualização de peso `w` nas cláusulas `on_pre`/`on_post` que capturem aspectos chave do comportamento do dispositivo (e.g., não-linearidade, limites, talvez alguma variabilidade estocástica adicionada manualmente), mesmo sem modelar variáveis de estado internas complexas do dispositivo.
*   **Estender Brian2 ou Usar Ferramentas Complementares:** Para simulações mais rigorosas que envolvam modelos de dispositivo complexos, pode ser necessário estender o Brian2 com código customizado (e.g., através de `Function` ou código C++ externo) ou usar simuladores especializados que co-simulem a rede neural e as características do dispositivo/circuito, ou frameworks que integrem modelos de dispositivo (muitas vezes desenvolvidos em conjunto com hardware).
*   **Análise Pós-Simulação:** Realizar simulações em Brian2 com plasticidade idealizada e, em seguida, analisar como as não idealidades (quantização, variabilidade, não-linearidade), aplicadas *a posteriori* aos pesos aprendidos, afetariam o desempenho da rede.

**Figura 8.6**: Impacto das Não Idealidades do Dispositivo NVM. Gráficos ilustrativos: (a) Variabilidade D2D e C2C nos níveis HRS/LRS. (b) Atualização não linear e assimétrica da condutância em resposta a pulsos de potenciação (azul) e depressão (vermelho). (c) Deriva da condutância ao longo do tempo para diferentes estados programados em PCM.

Compreender e modelar o comportamento (ideal e não ideal) dos dispositivos NVM é crucial para avaliar realisticamente o potencial das arquiteturas neuromórficas baseadas em CIM e para desenvolver algoritmos de aprendizado que sejam robustos a essas imperfeições do hardware.

**8.5. Estudo de Caso: Emulando Memória Associativa com Plasticidade Sináptica em Brian2**

Este estudo de caso visa demonstrar, de forma conceitual, como os princípios da memória associativa (armazenamento e recuperação de padrões por associação) podem ser emulados em uma SNN simulada com Brian2, utilizando regras de plasticidade sináptica padrão como um proxy para o armazenamento de informação nos pesos. É importante reiterar que **não estaremos modelando o comportamento detalhado de dispositivos memristivos ou PCM aqui**, mas sim usando a plasticidade hebbiana/STDP do Brian2 para ilustrar o *princípio computacional* de como uma rede pode aprender a completar padrões.

**Tarefa:** Armazenar dois padrões binários simples em uma pequena rede recorrente e, em seguida, testar a capacidade da rede de recuperar o padrão correto quando apresentada com uma versão ruidosa ou incompleta como pista.

**Arquitetura:** Usaremos uma única população de neurônios LIF, com conexões recorrentes (todos-para-todos, exceto autoconexões) que são modificadas por uma regra de plasticidade hebbiana simple ou STDP. Neurônios de input externos serão usados para apresentar os padrões durante as fases de armazenamento e recuperação.

**Fases:**

1.  **Armazenamento:** Apresentar cada padrão alvo repetidamente à rede. Durante a apresentação, os neurônios correspondentes aos pixels "ON" do padrão são forçados a disparar (ou recebem forte input). A regra de plasticidade fortalece as conexões entre neurônios que disparam juntos dentro de um padrão.
2.  **Recuperação (Recall):** Apresentar uma versão parcial ou ruidosa de um dos padrões armazenados como um input breve. Observar se a dinâmica recorrente da rede, moldada pelos pesos aprendidos, faz com que a atividade da rede convirja para o padrão completo original.

**Implementação Conceitual em Brian2:**

```python
# Importar bibliotecas
import brian2 as b2
import matplotlib.pyplot as plt
import numpy as np

b2.prefs.codegen.target = 'cython'
b2.defaultclock.dt = 0.1*b2.ms

# --- Parâmetros da Rede e Padrões ---
N_neurons = 100 # Neurônios na rede recorrente (e.g., 10x10 grid)
# Definir Padrões (como listas de índices de neurônios ativos)
pattern1_indices = np.random.choice(N_neurons, size=20, replace=False) # 20 neurônios aleatórios
pattern2_indices = np.random.choice(np.setdiff1d(np.arange(N_neurons), pattern1_indices), size=20, replace=False) # Outros 20

# Parâmetros de Simulação
duration_storage_epoch = 150 * b2.ms # Duração da apresentação de cada padrão
duration_pause = 50 * b2.ms # Pausa entre padrões
N_storage_repeats = 20 # Número de repetições de cada padrão
duration_recall_cue = 50 * b2.ms # Duração da pista de recuperação
duration_recall_evol = 200 * b2.ms # Tempo para a rede evoluir após a pista

# --- Neurônios LIF (População Recorrente) ---
tau = 10*b2.ms; Vt = -55*b2.mV; Vr = -70*b2.mV; El = -65*b2.mV; Rm = 100*b2.Mohm; ref_p = 2*b2.ms
lif_eqs_rec = '''
dv/dt = (El - v + Rm * (I_syn + I_ext))/tau : volt (unless refractory)
I_syn : amp # Corrente sináptica recorrente
I_ext : amp # Corrente externa (para input/pista)
'''
neuron_pop = b2.NeuronGroup(N_neurons, lif_eqs_rec, threshold='v>Vt', reset='v=Vr', refractory=ref_p, method='euler')
neuron_pop.v = El
neuron_pop.I_syn = 0*b2.nA
neuron_pop.I_ext = 0*b2.nA

# --- Plasticidade Hebbiana/STDP nas Conexões Recorrentes ---
# Usaremos uma regra STDP simples para fortalecer conexões correlacionadas
tau_pre = 15*b2.ms; tau_post = 15*b2.ms
A_pre = 0.005; A_post = 0.005
eta_ltp = 0.01; eta_ltd = 0.01 # Taxas iguais para Hebbian simples (pode precisar ajuste)
w_max_rec = 8.0 * b2.nA # Peso máximo (corrente injetada)
w_initial_rec = 1.0 * b2.nA

stdp_eqs_rec = '''
w : amp # Peso é uma corrente
dapre/dt = -apre / tau_pre : 1 (event-driven)
dapost/dt = -apost / tau_post : 1 (event-driven)
'''
on_pre_rec = '''
I_syn_post += w # Efeito pós-sináptico
w = clip(w - eta_ltd * apost * nA, 0, w_max_rec) # LTD
apre += A_pre
'''
on_post_rec = '''
w = clip(w + eta_ltp * apre * nA, 0, w_max_rec) # LTP
apost += A_post
'''

# Criação das Sinapses Recorrentes
syn_rec = b2.Synapses(neuron_pop, neuron_pop, model=stdp_eqs_rec,
                     on_pre=on_pre_rec, on_post=on_post_rec)
syn_rec.connect(condition='i!=j') # Todos para todos, exceto autoconexões
syn_rec.w = w_initial_rec
syn_rec.apre = 0.0; syn_rec.apost = 0.0
# Passar parâmetros (simplificado, definindo globalmente aqui)
syn_rec.namespace['tau_pre'] = tau_pre
syn_rec.namespace['tau_post'] = tau_post
syn_rec.namespace['eta_ltp'] = eta_ltp
syn_rec.namespace['eta_ltd'] = eta_ltd
syn_rec.namespace['w_max_rec'] = w_max_rec
syn_rec.namespace['A_pre'] = A_pre
syn_rec.namespace['A_post'] = A_post

# --- Monitores ---
spikemon_rec = b2.SpikeMonitor(neuron_pop)
# Monitorar pesos (opcional, pode ser lento para N^2 sinapses)
# weight_mon_rec = b2.StateMonitor(syn_rec, 'w', record=np.random.choice(len(syn_rec), 100, replace=False), dt=duration_storage_epoch)

# --- Simulação ---
net = b2.Network(neuron_pop, syn_rec, spikemon_rec) # Adicionar weight_mon_rec se usado
net.store('initial') # Salvar estado inicial (incluindo pesos iniciais)

# 1. Fase de Armazenamento
print("Iniciando fase de armazenamento...")
input_current_storage = 0.5 * b2.nA # Corrente para forçar disparo durante armazenamento
for r in range(N_storage_repeats):
    print(f"  Repetição {r+1}/{N_storage_repeats}")
    # Apresentar Padrão 1
    neuron_pop.I_ext = 0*b2.nA
    neuron_pop.I_ext[pattern1_indices] = input_current_storage
    net.run(duration_storage_epoch, report=None)
    neuron_pop.I_ext = 0*b2.nA # Remover input
    net.run(duration_pause, report=None) # Pausa

    # Apresentar Padrão 2
    neuron_pop.I_ext = 0*b2.nA
    neuron_pop.I_ext[pattern2_indices] = input_current_storage
    net.run(duration_storage_epoch, report=None)
    neuron_pop.I_ext = 0*b2.nA
    net.run(duration_pause, report=None)
net.store('trained') # Salvar estado após treinamento
print("Armazenamento concluído.")

# 2. Fase de Recuperação (Testar com Pista Ruidosa do Padrão 1)
print("Iniciando fase de recuperação...")
net.restore('trained') # Carregar pesos treinados
spikemon_rec.resize(0) # Limpar spikes do monitor para ver só a recuperação

# Criar pista ruidosa (e.g., 70% dos neurônios do padrão 1)
cue_indices = np.random.choice(pattern1_indices, size=int(0.7 * len(pattern1_indices)), replace=False)
input_current_recall = 0.4 * b2.nA # Corrente da pista (pode ser menor)

# Aplicar a pista
neuron_pop.I_ext = 0*b2.nA
neuron_pop.I_ext[cue_indices] = input_current_recall
net.run(duration_recall_cue, report=None)
# Remover a pista e deixar a rede evoluir
neuron_pop.I_ext = 0*b2.nA
net.run(duration_recall_evol, report=None)
print("Recuperação concluída.")

# --- Visualização ---
# Plotar Raster da Fase de Recuperação
plt.figure(figsize=(10, 6))
plt.plot(spikemon_rec.t / b2.ms, spikemon_rec.i, '.k', markersize=2)
# Destacar neurônios que pertencem ao padrão 1 (alvo)
pattern1_spikes_mask = np.isin(spikemon_rec.i, pattern1_indices)
plt.plot(spikemon_rec.t[pattern1_spikes_mask] / b2.ms, spikemon_rec.i[pattern1_spikes_mask], '.g', markersize=3, label='Neurônios Padrão 1')
# Marcar período da pista
plt.axvspan(0, duration_recall_cue / b2.ms, color='yellow', alpha=0.3, label='Período da Pista')
plt.xlabel('Tempo na Fase de Recuperação (ms)')
plt.ylabel('Índice do Neurônio')
plt.title('Recuperação de Padrão (Pista Padrão 1 Ruidoso)')
plt.legend()
plt.show()

# Opcional: Visualizar matriz de pesos final (pode ser grande)
final_weights = syn_rec.w[:] # Obter todos os pesos
# weight_matrix = np.zeros((N_neurons, N_neurons))
# weight_matrix[syn_rec.i, syn_rec.j] = final_weights / b2.nA # Converter para nA para plotar
# plt.figure(figsize=(7,7))
# plt.imshow(weight_matrix, cmap='viridis', origin='lower')
# plt.colorbar(label='Peso Sináptico (nA)')
# plt.xlabel('Neurônio Pré-sináptico (j)')
# plt.ylabel('Neurônio Pós-sináptico (i)')
# plt.title('Matriz de Pesos Recorrentes Após Treinamento')
# plt.show()
```
Este código configura uma rede recorrente onde os pesos são modificados por STDP durante a apresentação repetida de dois padrões distintos. A fase de recuperação aplica uma versão parcial de um dos padrões e observa a atividade subsequente. Idealmente, o raster plot da recuperação deve mostrar que, após a remoção da pista, a atividade da rede se concentra nos neurônios que pertencem ao padrão 1 completo, indicando que a rede "completou" o padrão com base nos pesos associativos aprendidos. A visualização da matriz de pesos (se computacionalmente viável) poderia mostrar blocos de pesos mais fortes entre neurônios pertencentes ao mesmo padrão. Este exemplo, embora conceitual, ilustra como a plasticidade local em SNNs pode implementar princípios de memória associativa.

**Figura 8.7**: Emulação de Memória Associativa (Recuperação). Raster plot da fase de recuperação gerado pelo código. Mostra a atividade dos neurônios da rede (eixo Y) ao longo do tempo (eixo X). O período inicial (sombreamento amarelo) mostra a aplicação da pista ruidosa (ativando um subconjunto de neurônios do padrão 1). Após a remoção da pista, a atividade da rede (pontos pretos) idealmente converge e se sustenta nos neurônios pertencentes ao padrão 1 completo (pontos verdes), demonstrando pattern completion.

Este capítulo destacou a profunda conexão entre as limitações da computação convencional (Muro da Memória), os princípios de memória inspirados no cérebro (associativa, distribuída) e o potencial das tecnologias emergentes de NVM (RRAM, PCM) para realizar computação em memória. A capacidade desses dispositivos de atuar como sinapses artificiais abre caminho para hardwares neuromórficos radicalmente mais eficientes. Modelar seu comportamento, incluindo não idealidades, e explorar seus princípios computacionais através de simulações (mesmo conceituais, como com Brian2) são passos essenciais para concretizar essa promessa.

---

## REFERÊNCIAS BIBLIOGRÁFICAS

Ambrogio, S., Narayanan, P., Tsai, H., Shelby, R. M., Corrado, G. S., Pittore, M. L., ... & Burr, G. W. (2018). Equivalent-accuracy accelerated neural-network training using analogue memory. *Nature*, *558*(7708), 60-67. https://doi.org/10.1038/s41586-018-0180-5
*   *Resumo:* *Demonstra o uso de memória de mudança de fase (PCM) em arranjos crossbar para acelerar o treinamento de redes neurais. Exemplo chave de computação em memória com dispositivos emergentes, mostrando potencial e abordando desafios. Relevante para Seções 8.3 e 8.5 (contexto de hardware).*

Yao, P., Wu, H., Gao, B., Tang, J., Zhang, Q., Zhang, W., ... & Qian, H. (2020). Fully hardware-implemented memristor convolutional neural network. *Nature*, *577*(7792), 641-646. https://doi.org/10.1038/s41586-020-1942-4
*   *Resumo:* *Demonstra uma CNN implementada totalmente em hardware usando crossbars de memristores para VMM, mostrando alta eficiência energética. Exemplo de aplicação de CIM com NVM para IA. Relevante para Seções 8.3 e motivação do capítulo.*

Chen, J., Qi, S., Wang, J., & Ma, D. (2021). Challenges and opportunities for low power AI hardware computation. *Science China Information Sciences*, *64*(10), 201401. https://doi.org/10.1007/s11432-021-3309-5
*   *Resumo:* *Revisa os desafios energéticos da computação de IA, incluindo o Memory Wall. Discute explicitamente a computação em memória e neuromórfica como soluções. Relevante para Seção 8.1 e motivação geral do capítulo.*

Christensen, D. V., Dittmann, R., Linares-Barranco, B., Sebastian, A., Le Gallo, M., Redaelli, A., ... & Spiga, S. (2022). 2022 roadmap on neuromorphic computing and engineering. *Neuromorphic Computing and Engineering*, *2*(2), 022501. https://doi.org/10.1088/2634-4386/ac4a83
*   *Resumo:* *Cobre materiais e dispositivos NVM, arquiteturas CIM e desafios. Fornece visão atualizada dos tópicos do capítulo (Seções 8.3, 8.4). Contextualiza a pesquisa em memórias neuroinspiradas.*

Zidan, M. A., Strachan, J. P., & Lu, W. D. (2018). The future of electronics based on memristive systems. *Nature Electronics*, *1*(1), 22-29. https://doi.org/10.1038/s41928-017-0006-8
*   *Resumo:* *(Ligeiramente anterior a 2020, mas muito influente). Perspectiva sobre o potencial dos sistemas memristivos, incluindo CIM para IA. Cobre dispositivos e arquiteturas. Relevante para Seção 8.3.*

Karunaratne, G., Le Gallo, M., Cherubini, G., Benini, L., Rahimi, A., & Sebastian, A. (2020). In-memory hyperdimensional computing. *Nature Electronics*, *3*(6), 327-337. https://doi.org/10.1038/s41928-020-0410-3
*   *Resumo:* *Demonstra o uso de CIM (com PCM) para Computação Hiperdimensional. Mostra aplicação de CIM para além de VMMs padrão e para memória associativa. Relevante para Seções 8.2 e 8.3.*

Kvatinsky, S., Ramadan, M., Ramadan, E., & Fish, A. (2014). VTEAM: A general model for voltage-controlled memristors. *IEEE Transactions on Circuits and Systems II: Express Briefs*, *62*(8), 786-790. https://doi.org/10.1109/TCSII.2015.2433536
*   *Resumo:* *(Ligeiramente anterior a 2020, mas um modelo comportamental chave). Apresenta o modelo VTEAM para memristores. Exemplo de modelo comportamental (Seção 8.4) usado para simulação. Captura histerese e dinâmica de chaveamento.*

Sebastian, A., Le Gallo, M., Khaddam-Aljameh, R., & Eleftheriou, E. (2020). Memory devices and applications for in-memory computing. *Nature Nanotechnology*, *15*(7), 529-544. https://doi.org/10.1038/s41565-020-0655-8
*   *Resumo:* *Revisão focada em dispositivos NVM (PCM, RRAM) para CIM. Discute abordagens de CIM e desafios dos dispositivos. Referência central para Seções 8.3 e 8.4.*

Wang, Z., Wu, H., Burr, G. W., Peng, H., Rose, J., & Xia, Q. (2020). Resistive switching materials for information processing. *Nature Reviews Materials*, *5*(3), 173-195. https://doi.org/10.1038/s41578-019-0159-3
*   *Resumo:* *Revisa materiais para RRAM, mecanismos de comutação e suas aplicações. Cobre memória e computação neuromórfica baseada em RRAM. Relevante para Seção 8.3 e contexto de dispositivos.*

Xia, Q., & Yang, J. J. (2019). Memristive crossbar arrays for brain-inspired computing. *Nature Materials*, *18*(4), 309-323. https://doi.org/10.1038/s41563-019-0291-x
*   *Resumo:* *(Ligeiramente anterior a 2020, mas fundamental). Foca em arrays crossbar memristivos. Cobre dispositivos, desafios de integração (sneak path, variabilidade), e aplicações em CIM e neuromórfica. Relevante para Seções 8.3 e 8.4.*
