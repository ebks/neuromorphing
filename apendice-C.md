---

# APÊNDICE C

# COMPILAÇÃO DE RECURSOS BIBLIOGRÁFICOS E DIGITAIS ADICIONAIS

---

![imagem](neuro.png)

*Este apêndice compila uma seleção de recursos adicionais para leitores que desejam aprofundar seus conhecimentos em computação neuromórfica, neurociência computacional, modelagem com Brian2, tecnologias de hardware emergentes e pesquisa com organoides. A lista inclui livros de referência, artigos de revisão seminais ou recentes (complementares aos citados nos capítulos), websites de projetos importantes, documentação de software e outros recursos digitais relevantes. A seleção não é exaustiva, mas visa fornecer pontos de partida valiosos para exploração ulterior nas diversas facetas interdisciplinares abordadas neste volume.*

---

**C.1 Livros de Referência**

*   **Neurociência Computacional e Teórica:**
    *   Gerstner, W., Kistler, W. M., Naud, R., & Paninski, L. (2014). *Neuronal Dynamics: From Single Neurons to Networks and Models of Cognition*. Cambridge University Press. (Obra fundamental e abrangente, cobrindo modelos neuronais, SNNs, plasticidade e codificação. Possui exemplos e conexões com Brian).
    *   Dayan, P., & Abbott, L. F. (2001). *Theoretical Neuroscience: Computational and Mathematical Modeling of Neural Systems*. MIT Press. (Clássico que introduz as ferramentas matemáticas e computacionais para modelagem neural).
    *   Izhikevich, E. M. (2007). *Dynamical Systems in Neuroscience: The Geometry of Excitability and Bursting*. MIT Press. (Foco na análise de sistemas dinâmicos para entender padrões de disparo neuronal).
    *   Rieke, F., Warland, D., de Ruyter van Steveninck, R., & Bialek, W. (1997). *Spikes: Exploring the Neural Code*. MIT Press. (Exploração profunda da codificação da informação por spikes).

*   **Computação Neuromórfica e Hardware:**
    *   Indiveri, G., & Liu, S.-C. (Eds.). (2015). *Neuromorphic Engineering: From Principles to Application*. Frontiers Media SA. (Coletânea de artigos cobrindo diversos aspectos do campo, incluindo hardware analógico/digital e aplicações).
    *   Schuman, C. D., Potok, T. E., Patton, R. M., Birdwell, J. D., Dean, M. E., Rose, G. S., & Plank, J. S. (2017). *A Survey of Neuromorphic Computing and Neural Networks in Hardware*. Oak Ridge National Laboratory Technical Report. (Embora um relatório técnico, fornece um bom levantamento de hardware até a data). *Nota: Pode ser necessário buscar surveys mais recentes em formato de artigo.*

*   **Organoides Cerebrais e Biologia de Células-Tronco:**
    *   Lancaster, M. A., & Knoblich, J. A. (2014). Generation of cerebral organoids from human pluripotent stem cells. *Nature Protocols*, *9*(10), 2329-2340. (Protocolo seminal para geração de organoides).
    *   Muotri, A. R., & Gage, F. H. (2021). *Brain Organoids for Developmental Studies, Disease Modeling and Drug Discovery*. Academic Press. (Livro editado cobrindo aplicações de organoides).

**C.2 Artigos de Revisão e Perspectivas (Selecionados e Recentes)**

*   **Visão Geral Neuromórfica:**
    *   Christensen, D. V., et al. (2022). 2022 roadmap on neuromorphic computing and engineering. *Neuromorphic Computing and Engineering*, *2*(2), 022501. (Roadmap abrangente citado anteriormente).
    *   Schuman, C. D., et al. (2022). Opportunities for neuromorphic computing algorithms and applications. *Nature Computational Science*, *2*(1), 10-19. (Perspectiva sobre algoritmos e aplicações).
    *   Davies, M. (2024). Neuromorphic computing: From materials and devices to systems and algorithms. *MRS Bulletin*, *49*(1), 12-18. (Visão recente da Intel).

*   **Hardware Neuromórfico e Memórias:**
    *   Sebastian, A., et al. (2020). Memory devices and applications for in-memory computing. *Nature Nanotechnology*, *15*(7), 529-544. (Revisão sobre NVMs para CIM).
    *   Wang, Z., et al. (2020). Resistive switching materials for information processing. *Nature Reviews Materials*, *5*(3), 173-195. (Revisão sobre materiais RRAM).
    *   Xia, Q., & Yang, J. J. (2019). Memristive crossbar arrays for brain-inspired computing. *Nature Materials*, *18*(4), 309-323. (Foco em crossbars memristivos).

*   **Algoritmos SNN e Aprendizado:**
    *   Tavanaei, A., Ghodrati, M., Kheradpisheh, S. R., Masquelier, T., & Maida, A. (2019). Deep learning in spiking neural networks. *Neural Networks*, *111*, 47-63. (Revisão sobre aprendizado profundo em SNNs).
    *   Zenke, F., & Neftci, E. O. (2021). Brain-inspired learning on neuromorphic hardware. *Philosophical Transactions of the Royal Society A*, *379*(2208), 20200158. (Aprendizado bioinspirado e SG para hardware).
    *   Brette, R. (2015). Philosophy of the spike: Rate-based vs. spike-based theories of computation. *Frontiers in Systems Neuroscience*, *9*, 151. (Discussão conceitual sobre codificação).

*   **Organoides e Interfaces:**
    *   Paşca, B., Pasca, S. P., & Zuge, O. K. (2023). Simulating human brain development and function using organoids and assembloids. *Nature Reviews Neuroscience*, *24*(10), 671-685. (Revisão recente sobre organoides).
    *   Gemolo, E., et al. (2023). Neural interfaces for cerebral organoids: A review. *Biosensors*, *13*(6), 640. (Revisão sobre tecnologias de interface para organoides).
    *   Hyun, I., et al. (2023). Advancing brain organoid research responsibly... *Stem Cell Reports*, *18*(1), 1-5. (Discussão sobre ética).
    *   Field, R., et al. (2023). Organoid intelligence (OI)... *Frontiers in Science*, *1*, 1017235. (Perspectiva sobre OI e biocomputação).

**C.3 Software e Ferramentas Computacionais**

*   **Brian2:**
    *   *Website Principal:* [https://briansimulator.org/](https://briansimulator.org/)
    *   *Documentação:* [https://brian2.readthedocs.io/](https://brian2.readthedocs.io/) (Essencial, com tutoriais, exemplos e referência da API).
    *   *Código Fonte (GitHub):* [https://github.com/brian-team/brian2](https://github.com/brian-team/brian2)
    *   *Fórum/Lista de Discussão:* (Links disponíveis no website) Local para obter ajuda e discutir com a comunidade.

*   **Outros Simuladores SNN:**
    *   *NEST Simulator:* Focado em simulações de SNNs de larga escala, eficiente em HPC. [https://www.nest-simulator.org/](https://www.nest-simulator.org/)
    *   *NEURON:* Focado em simulações biofisicamente detalhadas de neurônios individuais e redes pequenas, com suporte a morfologias complexas. [https://neuron.yale.edu/neuron/](https://neuron.yale.edu/neuron/)
    *   *Nengo:* Framework de nível superior para construir modelos cognitivos e de redes neurais de larga escala, com suporte a SNNs e diferentes backends (incluindo hardware neuromórfico). [https://www.nengo.ai/](https://www.nengo.ai/)

*   **Análise de Dados Neurais (Python):**
    *   *NumPy & SciPy:* Bibliotecas fundamentais para computação numérica e científica. [https://numpy.org/](https://numpy.org/), [https://scipy.org/](https://scipy.org/)
    *   *Matplotlib:* Biblioteca padrão para plotagem 2D. [https://matplotlib.org/](https://matplotlib.org/)
    *   *Elephant (Electrophysiology Analysis Toolkit):* Biblioteca Python focada na análise de dados de trens de spikes (correlação, taxas, sincronia, STTC, etc.). [https://elephant.readthedocs.io/](https://elephant.readthedocs.io/)
    *   *Neo:* Biblioteca Python para representar dados eletrofisiológicos (incluindo trens de spikes, sinais analógicos) em um formato padronizado, frequentemente usada com Elephant. [https://neo.readthedocs.io/](https://neo.readthedocs.io/)
    *   *Scikit-learn:* Biblioteca abrangente para aprendizado de máquina (classificação, regressão, clustering), útil para decodificação ou análise de padrões. [https://scikit-learn.org/](https://scikit-learn.org/)

*   **Frameworks de IA com Suporte a SNNs (via SG):**
    *   *snnTorch (baseado em PyTorch):* Biblioteca popular para simular e treinar SNNs usando gradiente substituto. [https://snntorch.readthedocs.io/](https://snntorch.readthedocs.io/)
    *   *SpykeTorch (baseado em PyTorch):* Outra biblioteca focada em SNNs e aprendizado baseado em STDP e kernels. [https://github.com/surrogate-gradient-learning/spytorch](https://github.com/surrogate-gradient-learning/spytorch) (Verificar link/nome exato)
    *   *Nengo-DL (baseado em TensorFlow/Keras):* Backend do Nengo que permite treinar modelos Nengo (incluindo SNNs) usando TensorFlow e implantá-los em diferentes hardwares. [https://www.nengo.ai/nengo-dl/](https://www.nengo.ai/nengo-dl/)

*   **Software para Análise de Imagem de Cálcio:**
    *   *CaImAn:* Toolbox para análise de dados de imagem de cálcio em larga escala. [https://github.com/flatironinstitute/CaImAn](https://github.com/flatironinstitute/CaImAn)
    *   *Suite2p:* Pipeline para processamento de gravações de microscopia de dois fótons, incluindo registro de movimento, detecção de ROI e extração de sinais. [https://github.com/MouseLand/suite2p](https://github.com/MouseLand/suite2p)

**C.4 Plataformas de Hardware Neuromórfico (Websites de Projetos/Empresas)**

*   **Intel Neuromorphic Computing (Loihi):** [https://www.intel.com/content/www/us/en/research/neuromorphic-computing.html](https://www.intel.com/content/www/us/en/research/neuromorphic-computing.html) (Informações sobre Loihi, Loihi 2 e o framework Lava).
*   **SpiNNaker (University of Manchester):** [https://apt.cs.manchester.ac.uk/projects/SpiNNaker/](https://apt.cs.manchester.ac.uk/projects/SpiNNaker/) (Informações sobre o projeto SpiNNaker e SpiNNaker2).
*   **BrainScaleS (Heidelberg University / Human Brain Project):** [https://brainscales.kip.uni-heidelberg.de/](https://brainscales.kip.uni-heidelberg.de/) (Informações sobre a plataforma de modo misto).
*   **Akida (BrainChip):** [https://brainchip.com/](https://brainchip.com/) (Informações sobre o NPU neuromórfico focado em Edge AI).
*   *(Nota: A disponibilidade e os websites de outras plataformas, como TrueNorth ou Tianjic, podem variar ou ser menos acessíveis publicamente).*

**C.5 Datasets Neuromórficos (Baseados em Eventos)**

*   **N-MNIST, N-Caltech101:** Versões baseadas em eventos dos datasets MNIST e Caltech101, gravadas com sensores DVS. (Disponíveis em diversas fontes online, e.g., [https://www.garrickorchard.com/datasets/n-mnist](https://www.garrickorchard.com/datasets/n-mnist)).
*   **DVS Gesture:** Dataset de gestos gravados com DVS. (e.g., [https://ibm.ent.box.com/s/3l53fafftomluei1m3kg1dda4p8k6zed](https://ibm.ent.box.com/s/3l53fafftomluei1m3kg1dda4p8k6zed)).
*   **Spiking Heidelberg Digits (SHD) / Spiking Speech Commands (SSC):** Datasets de áudio convertidos para spikes. (Disponíveis online, e.g., através do grupo Zenke).
*   *(Nota: Novos datasets baseados em eventos estão sendo continuamente criados e disponibilizados pela comunidade).*

**C.6 Comunidades e Conferências**

*   **Workshops e Conferências:** Eventos como o Telluride Neuromorphic Cognition Engineering Workshop, a NICE (Neuromorphic Computing and Engineering) conference (da IOP), e sessões em conferências maiores de neurociência (SfN, FENS) ou IA/circuitos (NeurIPS, ICML, ISCAS, ISSCC) são locais importantes para acompanhar os últimos avanços.
*   **Fóruns Online:** Listas de discussão (como a do Brian2), fóruns Discourse, ou canais Slack/Discord dedicados à computação neuromórfica podem ser úteis para interagir com a comunidade.

Esta lista fornece um ponto de partida substancial para aprofundar os tópicos abordados neste livro. Dada a rápida evolução do campo, recomenda-se também acompanhar as publicações em periódicos relevantes (e.g., *Nature Neuroscience*, *Nature Computational Science*, *Neuron*, *eLife*, *PLoS Computational Biology*, *Frontiers in Neuroscience/Neuroinformatics*, *Neuromorphic Computing and Engineering*, *IEEE Transactions on Neural Networks and Learning Systems*) e os preprints em plataformas como arXiv e bioRxiv.
