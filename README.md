# EyetrackingMOS-STIL

## Revisão sobre métricas subjetivas para análise de sistemas de TTS

Um padrão de referência de análise para descrever pontuação MOS para os avaliadores é apresentado na tabela abaixo [Loizou, 2011]:

| **Rating** | **Speech quality** (Qualidade da fala) | **Level of distortion** (Nível de distorção)           |
|------------|----------------------------------------|-------------------------------------------------------|
| 5          | Excellent (Excelente)                  | Imperceptible (Imperceptível)                         |
| 4          | Good (Boa)                             | Just perceptible, but not annoying (Apenas perceptível, mas não incômoda) |
| 3          | Fair (Razoável)                        | Perceptible and slightly annoying (Perceptível e ligeiramente incômoda) |
| 2          | Poor (Ruim)                            | Annoying, but not objectionable (Incômoda, mas aceitável) |
| 1          | Bad (Péssima)                          | Very annoying and objectionable (Muito incômoda e inaceitável) |
---

## Materiais e métodos
### Pré-processamento

![Fluxo da preprocessamento](Figs/Fig-fluxo_prepross.png)

A primeira etapa é a **anonimização**, em que os anotadores ocultaram informações pessoais dos falantes para proteger sua privacidade. Em seguida, realiza-se a **diarização automática**, que segmenta o áudio para identificar e separar diferentes fontes sonoras, como falantes ou ruídos de fundo, utilizando a biblioteca [pyannote](https://github.com/pyannote/pyannote-audio). Esse processo envolve a segmentação inicial do áudio com Detecção de Atividade de Voz (Voice Activity Detection, VAD) e o corte em partes que variam entre 1 e 30 segundos. Depois da **diarização**, ocorre a transcrição automática, onde os segmentos de áudio são convertidos em texto utilizando o Whisper X [Bain, 2022], que transcreve os segmentos de áudio. Esta transcrição é então revisada na etapa de **revisão manual da transcrição automática**, na qual anotadores corrigem possíveis erros, garantindo que o texto seja fiel ao áudio original. Na etapa de **anotação de artefatos**, os anotadores marcam no texto os trechos que contêm ruídos, cortes abruptos, palavras inaudíveis, mudanças de tom, ou outros elementos que possam interferir na qualidade da síntese de fala. Essas anotações são essenciais para identificar e tratar variações no áudio que possam afetar o treinamento do modelo. Na etapa de **limpeza dos dados**, são corrigidos erros e removidos segmentos problemáticos. Segmentos com falantes sem identificação ou transcrições inadequadas são removidos, e dados como nomes de cidades são padronizados. Por fim, na etapa de **modelagem dos dados**, os dados são organizados no formato adequado para uso em modelos de síntese de fala.

---

### Modelo de Síntese

O **SYNTACC** inclui componentes como o variational autoencoder (VAE) e outros elementos que visam melhorar a síntese de fala. O preditor de duração estocástico e o alinhamento monotônico auxiliam na diversidade e fluidez das sínteses de áudio. Durante o treinamento, os áudios foram reamostrados para 16 kHz, e o modelo SYNTACC, foi pré-treinado utilizando um \textit{checkpoint} disponível na plataforma Coqui. O tamanho do \textit{batch} foi definido como 26, equilibrando a eficiência do treinamento e a capacidade de memória. O treinamento foi monitorado com o \textit{TensorBoard}, permitindo a visualização das métricas em tempo real. Os dados de entrada foram organizados em 13 \textit{datasets} diferentes, correspondentes aos diferentes sotaques linguísticos regionais e internacionais. O treinamento utilizou o otimizador AdamW, com uma taxa de aprendizado inicial de 0,001 ajustada exponencialmente ao longo das épocas para melhorar a convergência (ver \ref{tab:model_config} para detalhes). Os parâmetros de configuração, como a escala do ruído para inferência (\verb|inference_noise_scale|) e o \textit{Stochastic Duration Predictor}, foram ajustados para 0,33. Durante o processo de avaliação, foram utilizadas sentenças de teste variadas para cobrir diferentes padrões de fala, refletindo a diversidade linguística abordada pelo modelo. Isso incentiva o modelo a generalizar, sintetizando fala de novos falantes e sotaques não observados durante o treinamento. Ao total, o modelo sintetizou as falas do experimento com um treinamento de 295 mil \textit{steps} no MuPe-v1.

Tabela de configuração de treinamento:
| Aspectos | Valores |
| --- | --- |
| SAMPLE_RATE | 16kHz |
| MAX_AUDIO_LEN_IN_SECONDS | sem limites |
| RESTORE_PATH | checkpoint |
| BATCH_SIZE | 26 |
| DASHBOARD_LOGGER | tensorboard |
| optmizer | AdamW |
| lr | 0.001 |
| lr_gen | 0.0002 |
| lr_scheduler_gen | ExponentialLR |
| lr_disc | 0.0002 |
| lr_scheduler_disc | ExponentialLR |

---

### Experimentos

O modelo foi avaliado utilizando o MOS e o EyetrackingMOS para medir a qualidade e percepção dos estímulos sonoros. A figura abaixo demonstra o fluxo da avaliação:

####Interface

A imagem abaixo corresponde a interface que o usuário recebe durante os estímulos de áudio masculinos e femininos, respectivamente, como demonstrado no artigo.

![Interface do Estímulo](Figs/Fig-interface_estimulo.png)

####Estímulos

Os arquivos de áudio presentes na pasta `stimuli` são utilizados durante os testes.

---

## Referências

- Bain, M. et al. WhisperX: Multilingual Speech Recognition and Alignment, 2022.

- Loizou, P. C. Speech Quality Assessment, pages 623–654, 2011. Springer Berlin Heidelberg, Berlin, Heidelberg.
