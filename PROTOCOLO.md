# Protocolo de pesquisa — Alerta precoce de dengue em municípios pequenos

**Versão 0.1, 20 de novembro de 2026.** Proposta de delineamento para revisão e checagem de viabilidade. Os parâmetros abaixo são escolhas de estudo. A versão confirmatória será datada e congelada antes da análise de desempenho nos dados reservados.

## 1. Pergunta e contribuição

Em municípios brasileiros com até 50 mil habitantes, acrescentar as notificações dos municípios vizinhos ao histórico local aumenta a proporção de surtos de dengue avisados com pelo menos duas semanas de antecedência, sob o mesmo limite de alarmes falsos?

Um estudo nacional de alerta baseado em atendimentos da atenção primária encontrou menor desempenho em municípios pequenos e sugeriu aproveitar sinais regionais. Chen e Moraga já testaram o uso de dados de municípios vizinhos e encontraram melhora na previsão de casos nas dez grandes cidades estudadas. O presente estudo avalia o ganho para alertas de início de surtos em cidades pequenas, com uma aproximação do atraso com que chegam as notificações. [Estudo da atenção primária](https://doi.org/10.1016/j.lana.2025.101165) · [Chen e Moraga](https://pmc.ncbi.nlm.nih.gov/articles/PMC12657288/)

Os resultados e o desenho da comparação de Chen e Moraga servem de referência. A checagem do repositório público dos autores encontrou o README na árvore principal. A comparação proposta pode ser implementada com regressão logística e ferramentas estatísticas existentes. [Repositório dos autores](https://github.com/ChenXiang1998/dengue-lstm-forecasting-brazil)

## 2. Dados e população

| Insumo | Uso |
| --- | --- |
| Microdados públicos de dengue do SINAN, arquivos anuais | Data da primeira digitação, data de início dos sintomas, município de residência e classificação disponível na extração. |
| Estimativas populacionais e malha municipal do IBGE de 2015 | Definição fixa do porte, denominadores e vizinhança ao longo da série. |
| Regiões e unidades da Federação do IBGE | Agrupamento na avaliação geográfica e na estimativa da incerteza. |

[Dados e dicionário do SINAN](https://dadosabertos.saude.gov.br/dataset/arboviroses-dengue) · [População municipal de 2015](https://agenciadenoticias.ibge.gov.br/agencia-sala-de-imprensa/2013-agencia-de-noticias/releases/9647-ibge-divulga-as-estimativas-populacionais-dos-municipios-em-2015-atualizado-as-18-00h-do-dia-28-08-2015) · [Malha municipal de 2015](https://geoftp.ibge.gov.br/organizacao_do_territorio/malhas_territoriais/malhas_municipais/municipio_2015/)

A população-alvo é formada por municípios com até 50 mil residentes na estimativa de 2015, correspondência inequívoca de código na malha e pelo menos um vizinho elegível. Um vizinho é um município da mesma unidade da Federação com o qual compartilha um trecho de fronteira. Contato em um único ponto fica fora. O sinal pode incluir vizinhos de qualquer porte. A análise principal usa as séries comparáveis do SINAN e exclui o Espírito Santo por causa da mudança da fonte estadual em 2020. O Distrito Federal fica fora do recorte municipal. A análise documentará as exclusões e sua distribuição geográfica. [Notas do Ministério da Saúde sobre a cobertura do SINAN](https://dadosabertos.saude.gov.br/dataset/arboviroses-dengue)

Antes de avaliar qualquer previsão, conferir a cobertura dos arquivos, datas ausentes ou incompatíveis, marcas oficiais de duplicidade quando disponíveis e o cruzamento dos códigos do SINAN com os do IBGE. Para os registros com data utilizável, calcular o atraso entre sintomas e primeira digitação. Um zero semanal significa ausência de registros em uma série cuja cobertura esteja documentada. As lacunas de cobertura ficam registradas como faltantes.

## 3. Como reconstruir as semanas

A unidade de previsão é um município em uma semana epidemiológica. Ao final da semana t, cada versão gera um escore para o início de um surto em t+2, t+3 ou t+4. Nas entradas das duas versões, contar notificações de suspeita de dengue pela semana da primeira digitação, inclusive as que foram descartadas depois. Isso evita usar a classificação final para decidir se um registro já contaria no momento do alerta. Registros sem data de digitação utilizável ficam fora das entradas da análise principal. A data de primeira digitação aproxima o momento de disponibilidade e as revisões posteriores do registro podem não estar preservadas na extração pública. [Dicionário no portal de dados do SINAN](https://dadosabertos.saude.gov.br/dataset/arboviroses-dengue)

Para identificar o que aconteceu, usar os casos prováveis por semana de início dos sintomas na extração consolidada, segundo a definição do Ministério da Saúde, que exclui os casos descartados. O início de surto será a primeira de pelo menos duas semanas consecutivas em que a janela móvel de quatro semanas, incluindo a semana atual, acumule ao menos 100 casos por 100 mil habitantes e pelo menos cinco casos. Cada sequência que satisfizer ambos os critérios conta como um episódio. Essa é a definição operacional do estudo. Avaliar também uma definição de 200 casos por 100 mil, mantendo o mínimo de cinco. [Definição de caso provável no portal do SINAN](https://dadosabertos.saude.gov.br/dataset/arboviroses-dengue)

## 4. Comparação e separação dos dados

- **Versão local.** Regressão logística com penalização L2, sazonalidade anual e taxas municipais de notificações por 100 mil habitantes nas últimas uma e quatro semanas, incluindo a semana t.
- **Versão com vizinhos.** A mesma especificação, acrescida das duas taxas equivalentes para o conjunto de vizinhos. Calcular a taxa dividindo a soma das notificações pela soma das populações dos vizinhos. Usar as mesmas observações elegíveis nas duas versões.
- **Ajuste e teste.** Ajustar os modelos e qualquer transformação dos dados nas semanas epidemiológicas de 2016 a 2021. Escolher os limiares de alerta em 2022 e 2023. Avaliar o desempenho uma vez em 2024 e 2025, por ano e no conjunto. Usar dados anteriores à janela para preencher o histórico quando disponíveis. Excluir as previsões ou episódios cujo intervalo completo de avaliação atravesse a fronteira entre conjuntos.
- **Verificação geográfica.** Repetir a comparação deixando de fora uma grande região do país por vez durante o ajuste e a escolha dos limiares. Avaliar a região omitida nas semanas de 2024 e 2025. Usar o sinal dos vizinhos dessa região quando ele estiver disponível na semana simulada.

## 5. Medidas e critério de interpretação

Uma semana com alerta conta como falso alarme quando nenhum surto começa de duas a quatro semanas depois. Um surto conta como antecipado se receber pelo menos um alerta entre quatro e duas semanas antes do início. A contagem de surtos é por episódio. A carga de alarmes falsos é medida por 100 semanas-município elegíveis e inclui alertas em semanas distintas.

Na etapa de escolha do limiar, usar para cada versão o ponto que antecipa mais surtos sob o teto comum de **um alarme falso por 100 semanas-município**. Em caso de empate, escolher o ponto com menos alarmes falsos e, persistindo o empate, o maior limiar. Incluir a opção de não emitir alertas. Manter os limiares fixos depois dessa etapa.

No teste, publicar por versão a proporção de surtos antecipados, os alarmes falsos por 100 semanas-município, a proporção de alertas úteis e a antecedência dos alertas úteis. Estimar a diferença entre as proporções de surtos antecipados com intervalo de 95%, reamostrando estados em blocos e mantendo as observações das duas versões juntas. Isso preserva a dependência entre as semanas e os municípios vizinhos de um estado na comparação.

O ganho será considerado promissor se ambas as versões ficarem dentro do teto no teste, a versão com vizinhos antecipar **pelo menos cinco surtos adicionais por 100** e o limite inferior do intervalo da diferença ficar acima de zero. O teto e os cinco pontos percentuais são referências analíticas deste protocolo. Se alguma versão exceder o teto, publicar o desempenho observado e registrar que a comparação no limite pretendido ficou inconclusiva. Verificar a robustez atrasando a disponibilidade de todas as notificações em mais sete e 14 dias, com o mesmo procedimento de ajuste e calibração em cada cenário. Os resultados serão interpretados como desempenho retrospectivo.

## 6. Registro e reprodução

- **Verificações antes do fechamento.** Confirmar acesso e cobertura dos arquivos anuais, possibilidade de reconstrução temporal e enquadramento ético do uso dos dados. Registrar a regra adotada e eventual parecer ou justificativa aplicável. Se uma falha de cobertura exigir mudança no desenho, datar a alteração e motivá-la antes de medir o desempenho no conjunto reservado.
- **Registro prévio de exploração.** Já foram consultadas séries agregadas de Tijucas, Canelinha e São João Batista e uma amostra inicial de 12.646 registros do arquivo de 2024 para verificar a presença das datas. Essas inspeções serviram à checagem da fonte. A comparação de desempenho entre as duas versões ainda depende da análise prevista neste protocolo.
- **Materiais reproduzíveis.** Identificar arquivos, datas de extração e somas de verificação. Versionar os scripts, as dependências, a semente de reamostragem e as instruções para reconstruir as tabelas. Referenciar os microdados na origem e compartilhar os resultados agregados da análise.
- **Relato científico.** Documentar desvios, limitações, contribuição dos autores humanos e uso de IA. Usar o guia RECORD para reportar a origem e a preparação das bases e os critérios de seleção. [RECORD](https://doi.org/10.1371/journal.pmed.1001885) · [Instruções da revista do SUS](https://www.scielo.br/journal/ress/about/)
