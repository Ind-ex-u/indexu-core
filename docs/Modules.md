## Module: analysis

### Responsabilidade principal
Orquestrar pipelines de análise de conteúdo (texto e mídia) e coordenar a persistência dos resultados analíticos.

---

### O que este módulo FAZ hoje
- Recebe requisições de análise por rotas próprias.
- Orquestra pipelines analíticos (semantic, sentiment, vision, transcription, reflection, safety, feedback).
- Coordena a execução de serviços analíticos especializados.
- Persiste resultados analíticos por meio da camada de persistência (`analysis.persistence`).
- Atua como orquestrador central dos fluxos de análise.

---

### O que este módulo explicitamente NÃO FAZ
- Não decide regras de negócio relacionadas ao feed.
- Não gerencia identidade, perfil ou estados do usuário.
- Não altera diretamente dados de UI, feed ou reputação.
- Não executa autenticação ou autorização.

---

### Dados que este módulo pode LER
- Conteúdo submetido pelo usuário (texto, mídia e metadados).
- Resultados analíticos intermediários retornados por subserviços.
- Estado de capacidades do sistema (dependência indireta, quando presente).

---

### Dados que este módulo pode ESCREVER
- Registros analíticos persistidos via camada de persistência.
- Sinais semânticos, sentimentos, reflexões e derivados, conforme o pipeline acionado.

---

### Módulos dos quais este módulo DEPENDE
- `analysis/semantic`
- `analysis/reflection`
- `analysis/sentiment`
- `analysis/transcription`
- `analysis/vision`
- `analysis/safety`
- `analysis/feedback`
- `capacity` (AMBÍGUO — dependência indireta observável em alguns fluxos)
- `cost` (AMBÍGUO — tracking pode ocorrer, mas não é central no service)

---

### Módulos dos quais este módulo NUNCA deve depender
- `feed`
- `profile`
- `identity`
- `reputation`
- `tokenengine`
- `participation`

---

### Este módulo pode chamar IA?
**Sim.**  
Coordena chamadas a serviços especializados que utilizam IA (ex.: semantic, sentiment, vision, transcription, reflection), sempre de forma orquestrada e rastreável.

---

### Este módulo pode alterar IND ou U?
**Não.**  
O módulo apenas produz insumos analíticos.  
Qualquer alteração em IND ou U ocorre fora deste módulo, por meio de eventos e mediação explícita.


## Module: analysis/semantic

### Responsabilidade principal
Extrair, estruturar e persistir sinais semânticos derivados de conteúdo textual analisado.

---

### O que este módulo FAZ hoje
- Recebe requisições de extração de sinais semânticos por rotas próprias.
- Processa texto para identificar entidades e sinais semânticos.
- Normaliza sinais conforme tipos definidos localmente.
- Persiste sinais semânticos no banco de dados.
- Expõe endpoints específicos para persistência e testes de sinais semânticos.

---

### O que este módulo explicitamente NÃO FAZ
- Não gera reflexão.
- Não interpreta significado existencial.
- Não altera estado emocional, IND, EXD ou U.
- Não decide políticas de análise.
- Não orquestra pipelines completos.
- Não escreve em feed, perfil ou reputação.

---

### Dados que este módulo pode LER
- Texto fornecido no request.
- Tipos semânticos definidos em `analysis/semantic/types`.
- Payloads encaminhados pelo módulo `analysis`.

---

### Dados que este módulo pode ESCREVER
- Registros de semantic signals persistidos via Prisma.
- Metadados associados aos sinais (label, span, confidence, source).

---

### Módulos dos quais este módulo DEPENDE
- `analysis` (chamador / orquestrador)
- `db/prisma`
- Infra HTTP (controllers / routes)
- Tipos internos do próprio módulo

---

### Módulos dos quais este módulo NUNCA deve depender
- `feed`
- `profile`
- `identity`
- `userdimensions`
- `reputation`
- `tokenengine`
- `participation`

---

### Este módulo pode chamar IA?
**Não.**  
O código atual não contém chamadas diretas a LLMs ou modelos externos; o módulo apenas processa e persiste sinais calculados localmente ou recebidos de forma explícita.

---

### Este módulo pode alterar IND ou U?
**Não.**  
O módulo produz apenas sinais semânticos brutos, sem impacto direto em camadas dimensionais ou estados humanos.

---

### Estado do módulo
**Contrato arquitetural fechado.**  
Não possui dependências implícitas nem efeitos colaterais fora de seu domínio.


## Module: analysis/reflection

### Responsabilidade principal
Gerar reflexões textuais a partir de conteúdo analisado, aplicando políticas de profundidade, elegibilidade e formato.

---

### O que este módulo FAZ hoje
- Gera reflexões textuais com base em input previamente analisado.
- Aplica regras de política para determinar nível, profundidade e estrutura da reflexão.
- Atua como serviço interno, acionado por orquestradores (ex.: `analysis.service`).
- Retorna reflexões como objetos em memória para o chamador.

---

### O que este módulo explicitamente NÃO FAZ
- Não expõe rotas HTTP próprias.
- Não persiste reflexões diretamente em banco de dados.
- Não escreve em feed.
- Não altera perfil, identidade ou estados dimensionais.
- Não executa controle de capacidade ou custo diretamente.
- Não decide regras globais de negócio fora do escopo de reflexão.

---

### Dados que este módulo pode LER
- Texto e metadados analíticos fornecidos pelo módulo `analysis`.
- Regras e parâmetros definidos em `reflectionPolicy.service`.
- Flags e configurações de ambiente, quando presentes no fluxo.

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido diretamente.
- Apenas objetos de reflexão retornados em memória ao chamador.

---

### Módulos dos quais este módulo DEPENDE
- `analysis/types`
- `analysis/reflection/reflectionPolicy`
- Serviços de IA (via abstrações internas)

---

### Módulos dos quais este módulo NUNCA deve depender
- `feed`
- `profile`
- `identity`
- `reputation`
- `userdimensions`
- `tokenengine`
- `participation`

---

### Este módulo pode chamar IA?
**Sim.**  
O serviço de reflexão realiza chamadas a IA/LLM exclusivamente para geração do conteúdo reflexivo, sempre sob orquestração externa e políticas explícitas.

---

### Este módulo pode alterar IND ou U?
**Não.**  
O módulo apenas gera conteúdo reflexivo.  
Não escreve, persiste ou muta estados dimensionais.

---

### Observações contratuais
- A persistência de reflexões ocorre fora deste módulo.
- Qualquer vínculo entre reflexão e feed, IND ou U é responsabilidade de módulos externos.
- `reflectionPolicy.service` atua apenas como governança interna de reflexão, não como regra de negócio global.

---

### Estado do módulo
**Contrato arquitetural fechado.**  
Módulo puro de geração reflexiva, sem efeitos colaterais persistentes.


## Module: analysis/feedback

### Responsabilidade principal
Gerar feedback analítico assíncrono a partir de conteúdo processado, utilizando filas e workers dedicados.

---

### O que este módulo FAZ hoje
- Expõe endpoint para solicitação de feedback analítico.
- Enfileira jobs de feedback por meio de BullMQ.
- Processa feedback de forma assíncrona em worker dedicado.
- Gera feedback textual a partir de conteúdo previamente analisado.
- Retorna ou persiste resultados conforme o fluxo definido no worker.

---

### O que este módulo explicitamente NÃO FAZ
- Não orquestra pipelines completos de análise.
- Não altera feed diretamente.
- Não altera perfil, identidade ou reputação.
- Não controla capacidade global ou custos.
- Não executa autenticação ou autorização.

---

### Dados que este módulo pode LER
- Conteúdo analítico fornecido no request.
- Payloads encaminhados pelo módulo `analysis`.
- Dados do job (BullMQ).
- Configurações de fila e processamento.

---

### Dados que este módulo pode ESCREVER
- Jobs de feedback na fila.
- Resultados de feedback processados (destino final definido no worker).
- Logs de processamento e execução.

---

### Módulos dos quais este módulo DEPENDE
- `analysis`
- Infraestrutura de filas (`BullMQ` / `Redis`)
- Serviços de IA utilizados no worker
- Configuração de Redis

---

### Módulos dos quais este módulo NUNCA deve depender
- `feed`
- `profile`
- `identity`
- `userdimensions`
- `reputation`
- `tokenengine`
- `participation`

---

### Este módulo pode chamar IA?
**Sim.**  
A geração de feedback no worker envolve chamadas a IA/LLM, sempre em contexto assíncrono e controlado.

---

### Este módulo pode alterar IND ou U?
**Não.**  
O módulo apenas gera feedback analítico e não muta estados dimensionais.

---

### Observações contratuais
- O processamento de feedback é explicitamente assíncrono.
- Existe separação clara entre request, fila, processamento e worker.
- Qualquer persistência definitiva fora do escopo direto do feedback deve ocorrer em módulos externos apropriados.

---

### Estado do módulo
**Contrato arquitetural fechado.**  
Módulo assíncrono isolado, sem efeitos colaterais diretos em estados humanos ou de feed.


## Module: analysis/safety

### Responsabilidade principal
Avaliar riscos e aplicar verificações de segurança sobre conteúdo analisado antes de sua utilização em pipelines subsequentes.

---

### O que este módulo FAZ hoje
- Analisa conteúdo (principalmente texto) sob critérios de segurança.
- Classifica ou valida conteúdo para uso em fluxos analíticos posteriores.
- Atua como serviço interno, acionado exclusivamente pelo módulo `analysis`.
- Retorna avaliações de risco para orientar decisões externas ao módulo.

---

### O que este módulo explicitamente NÃO FAZ
- Não expõe rotas HTTP próprias.
- Não persiste dados em banco de dados.
- Não bloqueia usuários nem executa ações disciplinares.
- Não altera feed, perfil, identidade ou estados dimensionais.
- Não controla permissões ou autenticação.

---

### Dados que este módulo pode LER
- Texto e metadados fornecidos pelo módulo `analysis`.
- Parâmetros internos de segurança definidos no serviço.
- Flags e configurações de ambiente relacionadas à segurança, quando presentes.

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.
- Apenas resultados de avaliação de segurança retornados em memória ao chamador.

---

### Módulos dos quais este módulo DEPENDE
- `analysis`
- Serviços de IA ou classificadores de segurança, quando configurados internamente

---

### Módulos dos quais este módulo NUNCA deve depender
- `feed`
- `profile`
- `identity`
- `userdimensions`
- `reputation`
- `tokenengine`
- `participation`

---

### Este módulo pode chamar IA?
**Sim.**  
O serviço pode utilizar modelos ou classificadores baseados em IA para avaliação de risco e segurança do conteúdo.

---

### Este módulo pode alterar IND ou U?
**Não.**  
O módulo apenas avalia segurança e não muta estados dimensionais ou identitários.

---

### Observações contratuais
- O módulo de safety atua exclusivamente como **avaliador**, nunca como decisor final.
- Qualquer ação derivada da avaliação de segurança é responsabilidade de módulos externos.
- Resultados de safety orientam fluxo, mas não produzem efeitos colaterais diretos.

---

### Estado do módulo
**Contrato arquitetural fechado.**  
Módulo puro de avaliação de risco, sem persistência ou impacto direto em estados humanos.


## Module: analysis/sentiment

### Responsabilidade principal
Analisar sentimento emocional de conteúdo textual e retornar classificações ou scores associados.

---

### O que este módulo FAZ hoje
- Analisa texto para identificar sentimento ou tonalidade emocional.
- Gera classificações e/ou scores de sentimento.
- Atua como serviço interno, acionado pelo módulo `analysis`.
- Retorna resultados de sentimento exclusivamente em memória ao chamador.

---

### O que este módulo explicitamente NÃO FAZ
- Não expõe rotas HTTP próprias.
- Não persiste dados em banco de dados.
- Não gera reflexão.
- Não altera feed, perfil ou identidade.
- Não altera estados dimensionais diretamente.
- Não decide políticas globais de interpretação emocional.

---

### Dados que este módulo pode LER
- Texto fornecido pelo módulo `analysis`.
- Configurações internas do serviço.
- Flags e variáveis de ambiente relacionadas à análise.

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.
- Apenas resultados analíticos de sentimento retornados em memória.

---

### Módulos dos quais este módulo DEPENDE
- `analysis`
- Serviços de IA ou modelos de classificação de sentimento

---

### Módulos dos quais este módulo NUNCA deve depender
- `feed`
- `profile`
- `identity`
- `userdimensions`
- `reputation`
- `tokenengine`
- `participation`

---

### Este módulo pode chamar IA?
**Sim.**  
O serviço utiliza modelos ou IA para classificação de sentimento, sempre como sinal analítico auxiliar.

---

### Este módulo pode alterar IND ou U?
**Não.**  
O módulo produz apenas sinais analíticos de sentimento, sem impacto direto em estados dimensionais ou identitários.

---

### Estado do módulo
**Contrato arquitetural fechado.**  
Módulo analítico puro, sem persistência e sem efeitos colaterais.


## Module: analysis/transcription

### Responsabilidade principal
Transcrever conteúdo de áudio ou vídeo em texto para uso em pipelines analíticos subsequentes.

---

### O que este módulo FAZ hoje
- Recebe referência a mídia (áudio ou vídeo).
- Executa transcrição de fala para texto.
- Retorna o texto transcrito ao módulo chamador (`analysis`).
- Atua exclusivamente como serviço interno utilitário.

---

### O que este módulo explicitamente NÃO FAZ
- Não expõe rotas HTTP próprias.
- Não persiste transcrições em banco de dados.
- Não realiza análise semântica, emocional ou interpretativa.
- Não altera feed, perfil, identidade ou estados dimensionais.
- Não gera reflexão ou feedback.

---

### Dados que este módulo pode LER
- Arquivos ou streams de mídia fornecidos pelo chamador.
- Configurações internas de transcrição.
- Flags e variáveis de ambiente relacionadas ao provedor de transcrição.

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.
- Apenas texto transcrito retornado em memória ao chamador.

---

### Módulos dos quais este módulo DEPENDE
- `analysis`
- Serviços externos ou modelos de IA de speech-to-text

---

### Módulos dos quais este módulo NUNCA deve depender
- `feed`
- `profile`
- `identity`
- `userdimensions`
- `reputation`
- `tokenengine`
- `participation`

---

### Este módulo pode chamar IA?
**Sim.**  
Utiliza serviços ou modelos de IA para transcrição de áudio e vídeo em texto.

---

### Este módulo pode alterar IND ou U?
**Não.**  
O módulo apenas converte mídia em texto, sem mutar estados dimensionais ou identitários.

---

### Estado do módulo
**Contrato arquitetural fechado.**  
Módulo utilitário puro de conversão mídia → texto, sem efeitos colaterais persistentes.


## Module: analysis/vision

### Responsabilidade principal
Analisar conteúdo visual (imagens ou frames de vídeo) e extrair descrições ou sinais visuais estruturados para uso analítico.

---

### O que este módulo FAZ hoje
- Recebe imagens ou frames de vídeo para análise.
- Executa análise visual para extração de descrições e sinais visuais.
- Retorna resultados visuais estruturados ao módulo chamador (`analysis`).
- Atua exclusivamente como serviço interno utilitário.

---

### O que este módulo explicitamente NÃO FAZ
- Não expõe rotas HTTP próprias.
- Não persiste resultados em banco de dados.
- Não realiza OCR textual.
- Não gera reflexão ou feedback.
- Não altera feed, perfil, identidade ou estados dimensionais.

---

### Dados que este módulo pode LER
- Imagens ou frames fornecidos pelo módulo `analysis`.
- Configurações internas do serviço de visão.
- Flags e variáveis de ambiente relacionadas ao provedor de visão computacional.

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.
- Apenas resultados de análise visual retornados em memória ao chamador.

---

### Módulos dos quais este módulo DEPENDE
- `analysis`
- Serviços externos ou modelos de IA de visão computacional

---

### Módulos dos quais este módulo NUNCA deve depender
- `feed`
- `profile`
- `identity`
- `userdimensions`
- `reputation`
- `tokenengine`
- `participation`

---

### Este módulo pode chamar IA?
**Sim.**  
Utiliza serviços ou modelos de IA para análise visual de imagens e vídeos.

---

### Este módulo pode alterar IND ou U?
**Não.**  
O módulo produz apenas sinais visuais auxiliares, sem impacto direto em estados dimensionais ou identitários.

---

### Estado do módulo
**Contrato arquitetural fechado.**  
Módulo analítico puro de visão computacional, sem persistência e sem efeitos colaterais.


## Module: analysis/ocr

### Responsabilidade principal
Extrair texto de imagens por meio de OCR para uso em pipelines analíticos subsequentes.

---

### O que este módulo FAZ hoje
- Recebe imagens para processamento.
- Executa OCR para extração de texto.
- Retorna o texto extraído ao módulo chamador (`analysis`).
- Atua exclusivamente como serviço interno utilitário de conversão imagem → texto.

---

### O que este módulo explicitamente NÃO FAZ
- Não expõe rotas HTTP próprias.
- Não persiste texto extraído em banco de dados.
- Não realiza análise semântica, emocional ou interpretativa.
- Não gera reflexão ou feedback.
- Não altera feed, perfil, identidade ou estados dimensionais.

---

### Dados que este módulo pode LER
- Imagens fornecidas pelo módulo `analysis`.
- Configurações internas do serviço de OCR.
- Flags e variáveis de ambiente relacionadas ao provedor de OCR.

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.
- Apenas texto extraído retornado em memória ao chamador.

---

### Módulos dos quais este módulo DEPENDE
- `analysis`
- Serviços externos ou modelos de IA de OCR / visão computacional

---

### Módulos dos quais este módulo NUNCA deve depender
- `feed`
- `profile`
- `identity`
- `userdimensions`
- `reputation`
- `tokenengine`
- `participation`

---

### Este módulo pode chamar IA?
**Sim.**  
Utiliza serviços ou modelos de OCR baseados em IA/visão computacional para extração de texto.

---

### Este módulo pode alterar IND ou U?
**Não.**  
O módulo apenas converte imagem em texto, sem mutar estados dimensionais ou identitários.

---

### Estado do módulo
**Contrato arquitetural fechado.**  
Módulo utilitário puro de OCR, sem persistência e sem efeitos colaterais.


## Module: analysis/types

### Responsabilidade principal
Definir contratos de tipos TypeScript compartilhados entre submódulos do domínio `analysis`.

---

### O que este módulo FAZ hoje
- Define interfaces e tipos TypeScript para estruturas analíticas.
- Padroniza formatos de dados utilizados entre serviços de `analysis`.
- Atua como camada passiva de contrato, sem lógica executável.
- É utilizado exclusivamente em tempo de compilação (TypeScript).

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica.
- Não processa conteúdo.
- Não chama IA.
- Não persiste dados.
- Não expõe rotas.
- Não orquestra pipelines.
- Não participa de fluxos de execução em runtime.

---

### Dados que este módulo pode LER
- Nenhum dado em runtime.
- Apenas utilizado em tempo de compilação (TypeScript).

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.

---

### Módulos dos quais este módulo DEPENDE
- Nenhum módulo interno.
- Apenas o sistema de tipos do TypeScript.

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `feed`
- `identity`
- `userdimensions`
- `reputation`
- `tokenengine`
- `participation`
- Qualquer outro módulo de domínio

---

### Este módulo pode chamar IA?
**Não.**  
Não contém lógica executável nem integração com serviços de IA.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Define apenas contratos estruturais, sem mutação de estados dimensionais.

---

### Estado do módulo
**Contrato arquitetural fechado.**


## Module: capacity

### Responsabilidade principal
Controlar e avaliar limites de uso relacionados à capacidade de reflexão e consumo de tokens.

---

### O que este módulo FAZ hoje
- Calcula estimativas de consumo de tokens.
- Avalia a capacidade disponível para geração de reflexões.
- Registra uso e limites de capacidade em persistência própria.
- Fornece serviços internos de verificação de capacidade para outros módulos.

---

### O que este módulo explicitamente NÃO FAZ
- Não gera reflexão.
- Não chama IA diretamente.
- Não expõe rotas HTTP.
- Não executa autenticação.
- Não altera feed, perfil ou identidade.
- Não decide políticas de negócio fora do escopo de capacidade.

---

### Dados que este módulo pode LER
- Registros de capacidade armazenados em banco de dados.
- Parâmetros de entrada relacionados a texto ou tamanho de payload.
- Configurações internas de limites.

---

### Dados que este módulo pode ESCREVER
- Registros de uso de capacidade de reflexão.
- Registros de consumo estimado ou efetivo de tokens.

---

### Módulos dos quais este módulo DEPENDE
- `db/prisma`
- Tipos internos do próprio módulo
- Utilitários de estimativa de tokens

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `feed`
- `profile`
- `identity`
- `userdimensions`
- `networkai`
- `reputation`
- `tokenengine`

---

### Este módulo pode chamar IA?
**Não.**  
O módulo apenas estima e registra consumo, sem invocar modelos de IA.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Capacidade é um limite operacional e não um estado dimensional.

---

### Estado do módulo
**Contrato arquitetural fechado.**


## Module: cost

### Responsabilidade principal
Registrar e acompanhar custos operacionais associados ao processamento do sistema, especialmente relacionados ao uso de IA.

---

### O que este módulo FAZ hoje
- Registra eventos de custo relacionados a operações do sistema.
- Normaliza informações de custo (ex.: tipo, origem, valor estimado).
- Atua como serviço interno chamado por outros módulos.

---

### O que este módulo explicitamente NÃO FAZ
- Não executa cobrança.
- Não bloqueia usuários por custo.
- Não estima tokens.
- Não expõe rotas HTTP.
- Não gera reflexão.
- Não altera feed, perfil ou identidade.

---

### Dados que este módulo pode LER
- Parâmetros de custo fornecidos por módulos chamadores.
- Configurações internas relacionadas a rastreamento de custo.

---

### Dados que este módulo pode ESCREVER
- Registros de custo persistidos (ex.: logs ou banco de dados).

---

### Módulos dos quais este módulo DEPENDE
- `db/prisma` (se presente na implementação)
- Tipos internos do próprio módulo

---

### Módulos dos quais este módulo NUNCA deve depender
- `feed`
- `profile`
- `identity`
- `userdimensions`
- `reputation`
- `tokenengine`
- `participation`

---

### Este módulo pode chamar IA?
**Não.**  
Apenas registra custos informados por outros módulos.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Custos são métricas operacionais e não estados dimensionais.

---

### Estado do módulo
**Contrato arquitetural fechado.**


## Module: feed/feed.service

### Responsabilidade principal
Fornecer leitura estruturada de conteúdos já persistidos para composição do feed do usuário.

---

### O que este módulo FAZ hoje
- Recupera registros de feed a partir do banco de dados.
- Aplica ordenação, paginação e filtros básicos.
- Agrega dados previamente calculados (ex.: reflexão, mídia, autor).
- Retorna estruturas prontas para consumo pelo frontend.

---

### O que este módulo explicitamente NÃO FAZ
- Não chama IA.
- Não executa análise.
- Não recalcula reflexão, sentimento ou semântica.
- Não persiste novos dados de análise.
- Não altera IND, EXD ou U.
- Não decide políticas de recomendação inteligentes.

---

### Dados que este módulo pode LER
- Tabelas de feed.
- Registros de reflexão já persistidos.
- Dados de usuário associados ao conteúdo.
- Metadados de mídia associados.

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.

---

### Módulos dos quais este módulo DEPENDE
- `db/prisma`
- Tipos internos do módulo `feed`
- `profile` (apenas para leitura de dados públicos, se presente)

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `capacity`
- `cost`
- `userdimensions`
- `networkai`
- `tokenengine`

---

### Este módulo pode chamar IA?
**Não.**  
Opera exclusivamente sobre dados já processados e persistidos.

---

### Este módulo pode alterar IND ou U?
**Não.**  
É um submódulo de leitura pura, sem mutação de estados humanos ou dimensionais.

---

### Estado do módulo
**Contrato arquitetural fechado.**


## Module: feed/feed.controller

### Responsabilidade principal
Expor endpoints HTTP para leitura do feed, delegando integralmente a lógica ao `feed.service`.

---

### O que este módulo FAZ hoje
- Define endpoints públicos de acesso ao feed.
- Valida parâmetros básicos de request (ex.: paginação).
- Encaminha requisições para o `feed.service`.
- Retorna respostas formatadas ao cliente.

---

### O que este módulo explicitamente NÃO FAZ
- Não contém lógica de negócio.
- Não chama IA.
- Não executa análise.
- Não persiste dados.
- Não altera IND, EXD ou U.
- Não decide ordenação inteligente ou recomendação.

---

### Dados que este módulo pode LER
- Parâmetros de request (query, params).
- Contexto de usuário autenticado (se presente).

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.

---

### Módulos dos quais este módulo DEPENDE
- `feed/feed.service`
- Infraestrutura HTTP (Express)

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `capacity`
- `cost`
- `userdimensions`
- `networkai`
- `tokenengine`

---

### Este módulo pode chamar IA?
**Não.**  
Apenas expõe endpoints de leitura e delega a execução ao serviço de feed.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Opera exclusivamente como camada de transporte HTTP, sem mutação de estados.

---

### Estado do módulo
**Contrato arquitetural fechado.**


## Module: feed/feed.routes

### Responsabilidade principal
Declarar as rotas HTTP do feed e conectar o controller à infraestrutura de roteamento.

---

### O que este módulo FAZ hoje
- Define rotas públicas de acesso ao feed.
- Encaminha requisições para o `feed.controller`.
- Restringe operações a leitura (GET).

---

### O que este módulo explicitamente NÃO FAZ
- Não contém lógica de negócio.
- Não chama serviços diretamente.
- Não executa análise ou IA.
- Não persiste dados.
- Não altera estados humanos.

---

### Dados que este módulo pode LER
- Definições de rota.
- Middleware de autenticação (se aplicado externamente).

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.

---

### Módulos dos quais este módulo DEPENDE
- `feed/feed.controller`
- Infraestrutura HTTP / router

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `capacity`
- `cost`
- `userdimensions`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Define apenas rotas e delega o fluxo para camadas superiores.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Não executa lógica nem muta estados dimensionais.

---

### Estado do módulo
**Contrato arquitetural fechado.**


## Module: identity/identity.service

### Responsabilidade principal
Criar, atualizar e recuperar registros de identidade básica do usuário, atuando como fonte de verdade para identidade persistida.

---

### O que este módulo FAZ hoje
- Cria e atualiza registros de identidade básica do usuário.
- Recupera dados de identidade a partir de `userId`.
- Centraliza leitura e escrita de campos de identidade persistidos.
- Atua como fonte de verdade para identidade (não perfil).

---

### O que este módulo explicitamente NÃO FAZ
- Não autentica usuários.
- Não emite, valida ou renova tokens.
- Não chama IA.
- Não executa análise semântica ou emocional.
- Não altera IND, EXD ou U.
- Não escreve em feed.
- Não coordena onboarding.

---

### Dados que este módulo pode LER
- Tabela de identidade do usuário (via Prisma).
- Parâmetros básicos de entrada (DTOs).

---

### Dados que este módulo pode ESCREVER
- Registros de identidade do usuário.
- Atualizações pontuais de campos de identidade.

---

### Módulos dos quais este módulo DEPENDE
- `db/prisma`
- Tipos e DTOs do próprio módulo

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `feed`
- `profile`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Opera exclusivamente sobre dados de identidade persistidos.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Identidade é tratada como dado estrutural, sem mutação de estados dimensionais.

---

### Estado do módulo
**Contrato arquitetural fechado.**


## Module: identity/identity.controller

### Responsabilidade principal
Expor endpoints HTTP para leitura e atualização de identidade, delegando integralmente a lógica ao `identity.service`.

---

### O que este módulo FAZ hoje
- Expõe endpoints HTTP para leitura e atualização de identidade.
- Encaminha todas as operações para o `identity.service`.
- Valida entrada básica por meio de DTOs.
- Utiliza contexto de usuário autenticado quando disponível.

---

### O que este módulo explicitamente NÃO FAZ
- Não contém lógica de negócio.
- Não autentica usuários.
- Não emite ou valida tokens.
- Não chama IA.
- Não executa análise.
- Não altera IND, EXD ou U.
- Não escreve diretamente em banco de dados.

---

### Dados que este módulo pode LER
- Parâmetros de request (body, params).
- Contexto do usuário autenticado (ex.: `req.user`, se presente).

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.

---

### Módulos dos quais este módulo DEPENDE
- `identity/identity.service`
- DTOs do módulo `identity`
- Infraestrutura HTTP (Express)

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `feed`
- `profile`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Apenas expõe endpoints HTTP e delega a execução ao serviço de identidade.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Opera como camada de transporte, sem mutação de estados humanos ou dimensionais.

---

### Estado do módulo
Ponto de atenção (não é erro) Se no futuro este controller aceitar campos “híbridos” (ex: bio, mood, capa), isso invadirá o domínio de profile. Hoje isso não ocorre.
**Contrato arquitetural fechado.**

## Module: identity/identity.routes

### Responsabilidade principal
Declarar as rotas HTTP do módulo `identity` e conectar o controller à infraestrutura de roteamento.

---

### O que este módulo FAZ hoje
- Declara rotas HTTP do módulo `identity`.
- Conecta rotas ao `identity.controller`.
- Restringe operações a leitura e atualização explícitas de identidade.

---

### O que este módulo explicitamente NÃO FAZ
- Não contém lógica de negócio.
- Não executa autenticação.
- Não chama IA.
- Não executa análise.
- Não persiste dados.
- Não altera IND, EXD ou U.

---

### Dados que este módulo pode LER
- Definições de rota.
- Middleware aplicado externamente (ex.: autenticação).

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.

---

### Módulos dos quais este módulo DEPENDE
- `identity/identity.controller`
- Infraestrutura de roteamento HTTP (Express)

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `feed`
- `profile`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Define apenas rotas e delega o fluxo para camadas superiores.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Não executa lógica nem muta estados humanos ou dimensionais.

---

### Estado do módulo
**Contrato arquitetural fechado.**

## Module: identity

### Responsabilidade principal
Gerenciar a identidade básica e estável do usuário, separada de autenticação, perfil e estados humanos.

---

### O que este módulo FAZ hoje
- Centraliza leitura e escrita de dados de identidade do usuário.
- Expõe endpoints controlados para acesso e atualização de identidade.
- Mantém identidade como fonte de verdade isolada.
- Organiza exports internos do módulo sem lógica adicional.

---

### O que este módulo explicitamente NÃO FAZ
- Não autentica usuários.
- Não gerencia sessões ou tokens.
- Não chama IA.
- Não executa análise.
- Não altera IND, EXD ou U.
- Não atua como perfil, expressão ou estado emocional.
- Não escreve em feed.
- Não participa de onboarding.

---

### Dados que este módulo pode LER
- Registros de identidade persistidos (via Prisma).
- Parâmetros de request e DTOs do módulo.

---

### Dados que este módulo pode ESCREVER
- Registros de identidade do usuário.
- Atualizações pontuais de campos de identidade.

---

### Módulos dos quais este módulo DEPENDE
- `db/prisma`
- Infraestrutura HTTP (controllers e routes)
- DTOs internos do módulo `identity`

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `feed`
- `profile`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`
- `tokenengine`

---

### Este módulo pode chamar IA?
**Não.**  
Opera exclusivamente sobre dados de identidade persistidos.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Identidade é tratada como dado estrutural, sem mutação de estados humanos ou dimensionais.

---

### Estado do módulo
Alerta contratual (não é erro) Se futuramente forem adicionados campos expressivos (bio, mood, capa, texto livre), este módulo quebrará o contrato atual e invadirá profile.
**Contrato arquitetural fechado.**

## Module: identity/identity.dto

### Responsabilidade principal
Definir estruturas de dados (DTOs) utilizadas nas operações de identidade como contrato de entrada entre controller e service.

---

### O que este módulo FAZ hoje
- Define estruturas de dados (DTOs) usadas em operações de identidade.
- Padroniza payloads de criação e atualização de identidade.
- Atua como contrato de entrada entre `identity.controller` e `identity.service`.

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de negócio.
- Não acessa banco de dados.
- Não chama IA.
- Não executa análise.
- Não altera IND, EXD ou U.
- Não valida regras complexas (apenas forma estrutural).

---

### Dados que este módulo pode LER
- Nenhum dado em runtime.
- Utilizado apenas como definição de tipo e estrutura.

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.

---

### Módulos dos quais este módulo DEPENDE
- Tipos básicos do TypeScript
- Decorators de validação estrutural (se presentes)

---

### Módulos dos quais este módulo NUNCA deve depender
- `db/prisma`
- `analysis`
- `analysis/*`
- `feed`
- `profile`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Define apenas contratos estruturais para entrada de dados.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Não executa lógica nem muta estados humanos ou dimensionais.

---

### Estado do módulo
**Contrato arquitetural fechado.**


## Module: profile/profile.service

### Responsabilidade principal
Ler e atualizar dados base e estáveis do perfil do usuário, centralizando sua persistência.

---

### O que este módulo FAZ hoje
- Lê e atualiza dados base do perfil do usuário.
- Centraliza a persistência de informações visíveis e estáveis do perfil.
- Atua como camada de acesso direto ao banco para atributos de perfil.
- Serve como base comum para outros subdomínios de `profile`.

---

### O que este módulo explicitamente NÃO FAZ
- Não autentica usuários.
- Não chama IA.
- Não executa análise semântica ou emocional.
- Não gera reflexão.
- Não calcula ou interpreta estado interno.
- Não altera IND, EXD ou U.
- Não escreve em feed.
- Não executa lógica de recomendação.

---

### Dados que este módulo pode LER
- Registros de perfil do usuário persistidos em banco de dados.
- Identificador do usuário (`userId`) proveniente do contexto.

---

### Dados que este módulo pode ESCREVER
- Registros persistidos de perfil do usuário.
- Campos básicos e estáveis do perfil.

---

### Módulos dos quais este módulo DEPENDE
- `db/prisma`
- Tipos e DTOs do módulo `profile`

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `identity`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Opera exclusivamente sobre dados de perfil persistidos.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Perfil é tratado como dado superficial e não dimensional.

---

### Estado do módulo
Se este service começar a aceitar texto livre com carga semântica profunda, haverá sobreposição com analysis ou userdimensions. Hoje isso não ocorre.
**Contrato arquitetural fechado.**

## Module: profile/mood.service

### Responsabilidade principal
Ler e atualizar o estado de humor declarado do usuário como atributo do perfil.

---

### O que este módulo FAZ hoje
- Lê e atualiza o estado de humor associado ao perfil do usuário.
- Persiste alterações explícitas de humor no banco de dados.
- Fornece leitura direta do humor atual para consumo externo.
- Trata humor como atributo declarativo, não inferido.

---

### O que este módulo explicitamente NÃO FAZ
- Não chama IA.
- Não executa análise emocional.
- Não infere sentimento a partir de texto ou comportamento.
- Não gera reflexão.
- Não altera IND, EXD ou U.
- Não executa lógica temporal ou histórica.
- Não escreve em feed.

---

### Dados que este módulo pode LER
- Registro de humor persistido do usuário.
- Identificador do usuário (`userId`).

---

### Dados que este módulo pode ESCREVER
- Estado de humor persistido do usuário.
- Campos associados ao humor (valor atual).

---

### Módulos dos quais este módulo DEPENDE
- `db/prisma`
- Tipos e DTOs do módulo `profile`

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `sentiment`
- `reflection`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Apenas persiste e expõe humor declarado pelo usuário.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Humor é tratado como atributo declarativo de perfil, não como estado dimensional.

---

### Estado do módulo
Alerta contratual (não é erro)
Se futuramente o humor for usado como input para recomendação ou inferência, isso não pode acontecer aqui. Hoje isso não ocorre.
**Contrato arquitetural fechado.**

## Module: profile/cover.service

### Responsabilidade principal
Gerenciar a imagem de capa declarativa associada ao perfil do usuário, centralizando sua persistência e recuperação.

---

### O que este módulo FAZ hoje
- Gerencia a imagem de capa associada ao perfil do usuário.
- Persiste referência e metadados da capa no banco de dados.
- Atualiza e recupera a capa atual do perfil.
- Trata a capa como atributo visual declarativo.

---

### O que este módulo explicitamente NÃO FAZ
- Não processa mídia (resize, compressão ou análise).
- Não chama IA.
- Não executa OCR ou visão computacional.
- Não infere estado emocional ou significado.
- Não altera IND, EXD ou U.
- Não escreve em feed.
- Não gerencia upload direto de mídia.

---

### Dados que este módulo pode LER
- Registro de capa persistido do usuário.
- Identificador do usuário (`userId`).
- Metadados simples da imagem (ex.: URL, key).

---

### Dados que este módulo pode ESCREVER
- Referência persistida da imagem de capa.
- Metadados associados à capa.

---

### Módulos dos quais este módulo DEPENDE
- `db/prisma`
- Tipos e DTOs do módulo `profile`

---

### Módulos dos quais este módulo NUNCA deve depender
- `media`
- `analysis`
- `analysis/*`
- `vision`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Apenas referencia mídia já existente, sem qualquer processamento.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Capa é tratada como atributo visual declarativo, não como estado humano.

---

### Estado do módulo
Alerta contratual (não é erro)
Se futuramente este serviço começar a validar ou transformar imagens, isso deve migrar para media. Hoje isso não ocorre.
**Contrato arquitetural fechado.**


## Module: profile/profile.controller

### Responsabilidade principal
Expor endpoints HTTP para leitura e atualização de dados base do perfil, delegando integralmente a lógica ao `profile.service`.

---

### O que este módulo FAZ hoje
- Expõe endpoints HTTP para leitura e atualização de dados base de perfil.
- Encaminha todas as operações para o `profile.service`.
- Valida payloads de entrada por meio de DTOs.
- Utiliza contexto de usuário autenticado quando necessário.

---

### O que este módulo explicitamente NÃO FAZ
- Não contém lógica de negócio.
- Não chama IA.
- Não executa análise semântica ou emocional.
- Não infere estado interno do usuário.
- Não altera IND, EXD ou U.
- Não persiste dados diretamente.
- Não interage com feed.

---

### Dados que este módulo pode LER
- Parâmetros de request (body, params).
- Contexto do usuário autenticado (ex.: `req.user`, se presente).

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.

---

### Módulos dos quais este módulo DEPENDE
- `profile/profile.service`
- DTOs do módulo `profile`
- Infraestrutura HTTP (Express)

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `sentiment`
- `reflection`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Apenas expõe endpoints HTTP e delega a execução ao serviço de perfil.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Opera como camada de transporte, sem mutação de estados humanos ou dimensionais.

---

### Estado do módulo
Alerta contratual (não é erro) Controllers específicos para mood e cover devem existir se esses subdomínios forem expostos publicamente. Este controller não deve absorver essas responsabilidades.
**Contrato arquitetural fechado.**

## Module: profile/mood.controller

### Responsabilidade principal
Expor endpoints HTTP para leitura e atualização do humor do usuário, delegando integralmente a lógica ao `mood.service`.

---

### O que este módulo FAZ hoje
- Expõe endpoints HTTP para leitura e atualização do humor do usuário.
- Encaminha operações exclusivamente para o `mood.service`.
- Valida payloads simples relacionados ao humor.
- Utiliza contexto de usuário autenticado.

---

### O que este módulo explicitamente NÃO FAZ
- Não contém lógica de negócio.
- Não chama IA.
- Não executa análise emocional.
- Não infere sentimento.
- Não altera IND, EXD ou U.
- Não persiste dados diretamente.
- Não escreve em feed.

---

### Dados que este módulo pode LER
- Parâmetros de request (body, params).
- Contexto do usuário autenticado.

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.

---

### Módulos dos quais este módulo DEPENDE
- `profile/mood.service`
- DTOs e validators do módulo `profile`
- Infraestrutura HTTP (Express)

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `sentiment`
- `reflection`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Apenas expõe endpoints e delega a execução ao serviço de humor.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Opera como camada de transporte, sem mutação de estados humanos ou dimensionais.

---

### Estado do módulo
Alerta contratual (não é erro) Se futuramente o humor for usado como input para qualquer lógica sistêmica, isso não pode ocorrer neste controller.
**Contrato arquitetural fechado.**

## Module: profile/cover.controller

### Responsabilidade principal
Expor endpoints HTTP para leitura e atualização da capa do perfil, delegando integralmente a lógica ao `cover.service`.

---

### O que este módulo FAZ hoje
- Expõe endpoints HTTP para leitura e atualização da capa do perfil.
- Encaminha operações exclusivamente para o `cover.service`.
- Valida payloads relacionados à capa (ex.: referência ou identificador da imagem).
- Utiliza contexto de usuário autenticado.

---

### O que este módulo explicitamente NÃO FAZ
- Não contém lógica de negócio.
- Não processa imagem (resize, compressão ou validação visual).
- Não chama IA.
- Não executa OCR ou visão computacional.
- Não infere estado emocional ou significado.
- Não altera IND, EXD ou U.
- Não persiste dados diretamente.
- Não escreve em feed.

---

### Dados que este módulo pode LER
- Parâmetros de request (body, params).
- Contexto do usuário autenticado.

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.

---

### Módulos dos quais este módulo DEPENDE
- `profile/cover.service`
- DTOs e validators do módulo `profile`
- Infraestrutura HTTP (Express)

---

### Módulos dos quais este módulo NUNCA deve depender
- `media`
- `analysis`
- `analysis/*`
- `vision`
- `sentiment`
- `reflection`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Apenas expõe endpoints HTTP e delega a execução ao serviço de capa.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Opera como camada de transporte, sem mutação de estados humanos ou dimensionais.

---

### Estado do módulo
Alerta contratual (não é erro) Se futuramente este controller aceitar upload binário ou transformação de imagem, isso deve migrar para media. Hoje isso não ocorre.
**Contrato arquitetural fechado.**

## Module: profile/profile.routes

### Responsabilidade principal
Declarar as rotas HTTP do domínio de perfil base e conectar o controller à infraestrutura de roteamento.

---

### O que este módulo FAZ hoje
- Declara as rotas HTTP do domínio de perfil base.
- Conecta rotas ao `profile.controller`.
- Restringe operações às ações explícitas de leitura e atualização de perfil.

---

### O que este módulo explicitamente NÃO FAZ
- Não contém lógica de negócio.
- Não chama serviços diretamente.
- Não chama IA.
- Não executa análise.
- Não persiste dados.
- Não altera IND, EXD ou U.
- Não escreve em feed.

---

### Dados que este módulo pode LER
- Definições de rota.
- Middlewares aplicados externamente (ex.: autenticação).

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.

---

### Módulos dos quais este módulo DEPENDE
- `profile/profile.controller`
- Infraestrutura de roteamento HTTP (Express)

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `sentiment`
- `reflection`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Define apenas rotas HTTP e delega o fluxo para o controller de perfil.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Não executa lógica nem muta estados humanos ou dimensionais.

---

### Estado do módulo
**Contrato arquitetural fechado.**

## Module: profile/updateProfile.dto

### Responsabilidade principal
Definir a estrutura de dados utilizada para operações de atualização do perfil base.

---

### O que este módulo FAZ hoje
- Define a estrutura de dados para atualização do perfil.
- Padroniza os campos aceitos em operações de update.
- Atua como contrato entre `profile.controller` e `profile.service`.

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de negócio.
- Não acessa banco de dados.
- Não chama IA.
- Não executa análise.
- Não infere estado emocional ou significado.
- Não altera IND, EXD ou U.

---

### Dados que este módulo pode LER
- Nenhum dado em runtime.
- Utilizado apenas para validação estrutural e tipagem.

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.

---

### Módulos dos quais este módulo DEPENDE
- Decorators de validação estrutural (ex.: `class-validator`)
- Tipos básicos do TypeScript

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `identity`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Define apenas contratos estruturais para atualização de perfil.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Não executa lógica nem muta estados humanos ou dimensionais.

---

### Estado do módulo
**Contrato arquitetural fechado.**

## Module: profile/updateMood.dto

### Responsabilidade principal
Definir a estrutura de dados utilizada para operações de atualização do humor declarado do usuário.

---

### O que este módulo FAZ hoje
- Define a estrutura de dados aceita para atualização de humor.
- Padroniza o payload de entrada para operações de `mood`.
- Atua como contrato entre `mood.controller` e `mood.service`.

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de negócio.
- Não acessa banco de dados.
- Não chama IA.
- Não executa análise emocional.
- Não infere sentimento.
- Não altera IND, EXD ou U.
- Não persiste dados.

---

### Dados que este módulo pode LER
- Nenhum dado em runtime.
- Utilizado apenas para validação estrutural e tipagem.

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.

---

### Módulos dos quais este módulo DEPENDE
- Decorators de validação estrutural (ex.: `class-validator`)
- Tipos básicos do TypeScript

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `sentiment`
- `reflection`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Define apenas contratos estruturais para atualização de humor.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Não executa lógica nem muta estados humanos ou dimensionais.

---

### Estado do módulo
**Contrato arquitetural fechado.**

## Module: profile/updateCover.dto

### Responsabilidade principal
Definir a estrutura de dados utilizada para operações de atualização da capa do perfil.

---

### O que este módulo FAZ hoje
- Define a estrutura de dados aceita para atualização da capa do perfil.
- Padroniza o payload de entrada para operações de capa.
- Atua como contrato entre `cover.controller` e `cover.service`.

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de negócio.
- Não acessa banco de dados.
- Não chama IA.
- Não processa ou valida imagem.
- Não executa OCR ou visão computacional.
- Não infere estado emocional ou significado.
- Não altera IND, EXD ou U.
- Não persiste dados.

---

### Dados que este módulo pode LER
- Nenhum dado em runtime.
- Utilizado apenas para validação estrutural e tipagem.

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.

---

### Módulos dos quais este módulo DEPENDE
- Decorators de validação estrutural (ex.: `class-validator`)
- Tipos básicos do TypeScript

---

### Módulos dos quais este módulo NUNCA deve depender
- `media`
- `analysis`
- `analysis/*`
- `vision`
- `sentiment`
- `reflection`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Define apenas contratos estruturais para atualização de capa.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Não executa lógica nem muta estados humanos ou dimensionais.

---

### Estado do módulo
**Contrato arquitetural fechado.**


## Module: profile/updateProfile.validator

### Responsabilidade principal
Validar forma e consistência básica dos dados de atualização de perfil antes da execução do `profile.service`.

---

### O que este módulo FAZ hoje
- Valida a forma e consistência básica dos dados de atualização de perfil.
- Garante presença, tipo e limites simples dos campos permitidos.
- Atua como guarda antes da execução do `profile.service`.

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de negócio.
- Não acessa banco de dados.
- Não chama IA.
- Não executa análise semântica ou emocional.
- Não infere estado interno do usuário.
- Não altera IND, EXD ou U.
- Não persiste dados.

---

### Dados que este módulo pode LER
- Payload de request (dados de atualização de perfil).
- Regras locais de validação estrutural.

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.

---

### Módulos dos quais este módulo DEPENDE
- `class-validator` / `class-transformer` (ou equivalentes)
- DTOs do módulo `profile`

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `identity`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Executa apenas validação estrutural em memória.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Validação é puramente estrutural e não muta estados humanos ou dimensionais.

---

### Estado do módulo
**Contrato arquitetural fechado.**

## Module: profile/updateMood.validator

### Responsabilidade principal
Validar forma e limites básicos dos dados de atualização de humor antes da execução do `mood.service`.

---

### O que este módulo FAZ hoje
- Valida a forma e os limites básicos dos dados de atualização de humor.
- Garante que o payload esteja estruturalmente correto antes de chegar ao `mood.service`.
- Atua como guarda de integridade do domínio declarativo de humor.

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de negócio.
- Não acessa banco de dados.
- Não chama IA.
- Não executa análise emocional.
- Não infere sentimento.
- Não altera IND, EXD ou U.
- Não persiste dados.

---

### Dados que este módulo pode LER
- Payload de request (dados de atualização de humor).
- Regras locais de validação estrutural.

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.

---

### Módulos dos quais este módulo DEPENDE
- `class-validator` / `class-transformer`
- DTOs do módulo `profile`

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `sentiment`
- `reflection`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Executa apenas validação estrutural em memória.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Validação é puramente estrutural e não muta estados humanos ou dimensionais.

---

### Estado do módulo
**Contrato arquitetural fechado.**

## Module: profile

### Responsabilidade principal
Gerenciar expressões visíveis e declarativas do usuário (perfil base, humor e capa), sem inferência, análise ou impacto em estados humanos profundos.

---

### O que este módulo FAZ hoje
- Permite leitura e atualização de dados base de perfil.
- Permite leitura e atualização de humor declarado.
- Permite leitura e atualização de capa visual.
- Centraliza a persistência de atributos visíveis e mutáveis do usuário.
- Expõe endpoints controlados para cada subdomínio do perfil.
- Valida forma e limites básicos dos dados antes da persistência.

---

### O que este módulo explicitamente NÃO FAZ
- Não autentica usuários.
- Não chama IA.
- Não executa análise semântica, emocional ou existencial.
- Não infere estado interno do usuário.
- Não altera IND, EXD ou U.
- Não gera reflexão.
- Não escreve em feed.
- Não processa mídia.
- Não toma decisões sistêmicas.

---

### Dados que este módulo pode LER
- Registros persistidos de perfil, humor e capa do usuário.
- Parâmetros de request e DTOs associados aos subdomínios.
- Contexto de usuário autenticado quando aplicável.

---

### Dados que este módulo pode ESCREVER
- Registros persistidos de perfil base.
- Estado de humor declarado.
- Referências e metadados de capa visual.

---

### Módulos dos quais este módulo DEPENDE
- `db/prisma`
- Infraestrutura HTTP (controllers e routes)
- DTOs e validators internos do módulo `profile`

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `sentiment`
- `reflection`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`
- `tokenengine`

---

### Este módulo pode chamar IA?
**Não.**  
Todos os subdomínios operam apenas com dados declarativos e persistidos.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Perfil, humor e capa não são tratados como estados dimensionais.

---

### Estado do módulo
Alerta contratual (não é erro)
Qualquer futura tentativa de:

inferir emoção a partir de humor

analisar imagem de capa

usar perfil como input analítico

quebrará o contrato deste módulo e deverá ocorrer em módulos próprios (analysis, media, userdimensions).
**Contrato arquitetural fechado.**

## Module: profile/index

### Responsabilidade principal
Centralizar e expor os exports públicos do módulo `profile`, definindo sua fronteira externa.

---

### O que este módulo FAZ hoje
- Centraliza exports públicos do módulo `profile`.
- Expõe controllers, routes e services do módulo para registro no `app.module` ou equivalente.
- Define a fronteira pública do módulo por meio de barrel export.

---

### O que este módulo explicitamente NÃO FAZ
- Não contém lógica de negócio.
- Não chama IA.
- Não acessa banco de dados diretamente.
- Não executa análise.
- Não altera IND, EXD ou U.
- Não persiste dados.

---

### Dados que este módulo pode LER
- Nenhum dado em runtime.

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.

---

### Módulos dos quais este módulo DEPENDE
- Arquivos internos do próprio módulo `profile`.

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Atua apenas como ponto de agregação e exportação.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Não executa lógica nem muta estados humanos ou dimensionais.

---

### Estado do módulo
**Contrato arquitetural fechado.**


## Module: media/media.service

### Responsabilidade principal
Gerenciar registro e persistência de mídia associada ao usuário, orquestrando o envio para processamento assíncrono.

---

### O que este módulo FAZ hoje
- Gerencia o registro e a persistência de mídia associada ao usuário.
- Cria entradas de mídia (ex.: vídeo ou imagem) no banco de dados.
- Orquestra o envio de jobs para processamento assíncrono.
- Atua como ponto central entre a camada HTTP e o pipeline de fila.

---

### O que este módulo explicitamente NÃO FAZ
- Não executa processamento pesado de mídia.
- Não transcodifica vídeo.
- Não extrai frames.
- Não chama IA.
- Não executa análise semântica, emocional ou visual.
- Não altera IND, EXD ou U.
- Não escreve em feed.
- Não decide uso ou significado da mídia.

---

### Dados que este módulo pode LER
- Metadados de mídia enviados pelo usuário.
- Identificador do usuário (`userId`).
- Configurações internas de processamento.

---

### Dados que este módulo pode ESCREVER
- Registros persistidos de mídia (status, tipo, referência).
- Jobs enfileirados para processamento assíncrono.

---

### Módulos dos quais este módulo DEPENDE
- `db/prisma`
- `media/queues/video.queue`
- Tipos internos do módulo `media`

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `vision`
- `sentiment`
- `reflection`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Apenas registra mídia e dispara processamento assíncrono via fila.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Mídia é tratada como ativo técnico, não como estado humano.

---

### Estado do módulo
Alerta contratual (não é erro) Se futuramente este service começar a: extrair frames gerar thumbnails chamar IA isso quebra o contrato e deve migrar para processor/worker.
**Contrato arquitetural fechado.**

## Module: media/video.queue

### Responsabilidade principal
Definir e expor a fila de processamento de vídeo como ponto de desacoplamento entre o service e o processamento assíncrono.

---

### O que este módulo FAZ hoje
- Define a fila de processamento de vídeo.
- Configura parâmetros da fila (nome, conexão e opções).
- Expõe interface para enfileirar jobs de vídeo.
- Atua como ponto de desacoplamento entre `media.service` e o processamento.

---

### O que este módulo explicitamente NÃO FAZ
- Não processa mídia.
- Não executa lógica de negócio.
- Não chama IA.
- Não executa análise visual ou semântica.
- Não altera IND, EXD ou U.
- Não escreve em feed.
- Não acessa banco de dados.

---

### Dados que este módulo pode LER
- Configurações de fila (ex.: Redis/BullMQ).
- Payloads de job recebidos do `media.service`.

---

### Dados que este módulo pode ESCREVER
- Jobs enfileirados no backend de fila (Redis).

---

### Módulos dos quais este módulo DEPENDE
- Infraestrutura de filas (BullMQ).
- Configuração de Redis.
- Tipos internos do módulo `media`.

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `vision`
- `sentiment`
- `reflection`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Apenas define e publica jobs em fila, sem qualquer processamento.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Atua exclusivamente como infraestrutura de fila, sem mutação de estados humanos.

---

### Estado do módulo
Alerta contratual (não é erro) Se lógica de decisão (ex: tipo de mídia, regra de negócio) entrar aqui, o contrato será quebrado. Hoje isso não ocorre.
**Contrato arquitetural fechado.**

## Module: media/video.processor

### Responsabilidade principal
Consumir jobs da fila de vídeo e coordenar o pipeline técnico de preparação e processamento de mídia.

---

### O que este módulo FAZ hoje
- Consome jobs da fila de processamento de vídeo.
- Executa lógica de preparação e processamento técnico de mídia.
- Coordena etapas técnicas do pipeline (ex.: validação de payload, chamadas utilitárias).
- Encaminha execução pesada para o worker quando aplicável.

---

### O que este módulo explicitamente NÃO FAZ
- Não expõe rotas HTTP.
- Não executa lógica de negócio do usuário.
- Não chama IA de interpretação humana.
- Não executa análise semântica ou emocional.
- Não altera IND, EXD ou U.
- Não escreve em feed.
- Não decide significado da mídia.

---

### Dados que este módulo pode LER
- Payload do job de vídeo.
- Configurações técnicas de processamento.
- Referências ou paths temporários de mídia.

---

### Dados que este módulo pode ESCREVER
- Resultados intermediários de processamento técnico.
- Logs de execução do pipeline.
- Atualizações técnicas de status (se previstas).

---

### Módulos dos quais este módulo DEPENDE
- Infraestrutura de filas (BullMQ).
- Utilitários técnicos de mídia.
- Configurações internas do módulo `media`.

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `vision`
- `sentiment`
- `reflection`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Executa apenas coordenação técnica de processamento de mídia.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Pipeline técnico isolado, sem mutação de estados humanos.

---

### Estado do módulo
Alerta contratual (não é erro) Se lógica humana (ex: “o que o vídeo significa”) entrar aqui, o contrato será quebrado. Hoje isso não ocorre.
**Contrato arquitetural fechado.**

## Module: media/media.worker

### Responsabilidade principal
Executar o processamento técnico pesado de mídia de forma assíncrona, aplicando transformações e atualizações técnicas.

---

### O que este módulo FAZ hoje
- Executa o processamento técnico pesado de mídia.
- Consome jobs preparados pelo `video.processor`.
- Realiza operações como transcodificação, manipulação de arquivos e geração técnica de derivados.
- Atualiza status técnico da mídia após o processamento.

---

### O que este módulo explicitamente NÃO FAZ
- Não expõe rotas HTTP.
- Não contém lógica de negócio do usuário.
- Não chama IA de interpretação humana.
- Não executa análise semântica, emocional ou visual.
- Não altera IND, EXD ou U.
- Não escreve em feed.
- Não decide significado ou relevância da mídia.

---

### Dados que este módulo pode LER
- Payload do job de mídia.
- Arquivos temporários ou armazenados de mídia.
- Configurações técnicas de processamento.

---

### Dados que este módulo pode ESCREVER
- Arquivos processados (derivados técnicos).
- Atualizações de status técnico da mídia.
- Logs de execução.

---

### Módulos dos quais este módulo DEPENDE
- Utilitários técnicos de mídia (ex.: FFmpeg, filesystem).
- Infraestrutura de worker e fila.
- Configurações internas do módulo `media`.

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `vision`
- `sentiment`
- `reflection`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Executa apenas processamento técnico pesado de mídia.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Processamento puramente técnico, sem mutação de estados humanos.

---

### Estado do módulo
**Contrato arquitetural fechado.**

## Module: media/media.controller

### Responsabilidade principal
Expor endpoints HTTP para registro e envio de mídia, delegando integralmente a lógica ao `media.service`.

---

### O que este módulo FAZ hoje
- Expõe endpoints HTTP para registro e envio de mídia.
- Encaminha requisições para o `media.service`.
- Valida parâmetros básicos de entrada (metadados e tipo).
- Utiliza contexto de usuário autenticado quando aplicável.

---

### O que este módulo explicitamente NÃO FAZ
- Não processa mídia.
- Não executa lógica de negócio.
- Não chama IA.
- Não executa análise semântica ou visual.
- Não altera IND, EXD ou U.
- Não escreve em feed.
- Não acessa filas ou workers diretamente.

---

### Dados que este módulo pode LER
- Parâmetros de request (body, params).
- Contexto do usuário autenticado.

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido.

---

### Módulos dos quais este módulo DEPENDE
- `media/media.service`
- Infraestrutura HTTP (Express)

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `vision`
- `sentiment`
- `reflection`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`

---

### Este módulo pode chamar IA?
**Não.**  
Apenas expõe endpoints HTTP e delega o fluxo ao serviço de mídia.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Opera como camada de transporte, sem mutação de estados humanos ou dimensionais.

---

### Estado do módulo
Alerta contratual (não é erro) Se futuramente este controller aceitar upload binário direto ou streaming pesado, isso deve ser tratado com cuidado para não misturar HTTP com processamento.
**Contrato arquitetural fechado.**

## Module: media

### Responsabilidade principal
Gerenciar registro, armazenamento e processamento técnico assíncrono de mídia, sem interpretação humana, análise semântica ou impacto em estados dimensionais.

---

### O que este módulo FAZ hoje
- Registra mídia enviada pelo usuário.
- Persiste metadados e status técnico da mídia.
- Enfileira jobs para processamento assíncrono.
- Executa pipeline técnico de processamento (queue → processor → worker).
- Processa mídia tecnicamente (ex.: transcodificação, geração de derivados).
- Atualiza status técnico da mídia após processamento.
- Expõe endpoints controlados para envio e registro de mídia.

---

### O que este módulo explicitamente NÃO FAZ
- Não chama IA de interpretação humana.
- Não executa análise semântica, emocional ou visual.
- Não executa OCR.
- Não altera IND, EXD ou U.
- Não escreve em feed.
- Não decide significado, relevância ou moderação de conteúdo.
- Não executa lógica de recomendação.
- Não mistura processamento pesado com a camada HTTP.

---

### Dados que este módulo pode LER
- Metadados de mídia enviados pelo usuário.
- Identificador do usuário (`userId`).
- Payloads de jobs de mídia.
- Configurações técnicas internas de processamento.

---

### Dados que este módulo pode ESCREVER
- Registros persistidos de mídia (metadados e status técnico).
- Jobs enfileirados para processamento assíncrono.
- Arquivos processados e derivados técnicos.
- Logs técnicos de execução.

---

### Módulos dos quais este módulo DEPENDE
- `db/prisma`
- Infraestrutura HTTP (controllers e routes)
- Infraestrutura de filas (BullMQ / Redis)
- Utilitários técnicos de mídia (ex.: FFmpeg, filesystem)
- Configurações internas do módulo `media`

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `vision`
- `sentiment`
- `reflection`
- `userdimensions`
- `capacity`
- `cost`
- `networkai`
- `tokenengine`

---

### Este módulo pode chamar IA?
**Não.**  
Todo o processamento é puramente técnico, sem interpretação humana.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Mídia é tratada como ativo técnico, sem mutação de estados dimensionais.

---

### Estado do módulo
Alerta contratual (não é erro) Qualquer futura tentativa de: classificar conteúdo interpretar imagem/vídeo moderar semanticamente mídia não pode ocorrer neste módulo e deve ser implementada em analysis ou networkai.
**Contrato arquitetural fechado.**


## Module: userdimensions/userdimensions.service

### Responsabilidade principal
Atuar como orquestrador central das operações do módulo `userdimensions`, centralizando leitura e escrita das dimensões do usuário.

---

### O que este módulo FAZ hoje
- Atua como orquestrador central das operações do módulo `userdimensions`.
- Encaminha ações para serviços específicos de dimensão (IND, EXD, U, state, event).
- Centraliza pontos de entrada para leitura e escrita de dimensões.
- Expõe operações agregadas relacionadas ao estado dimensional do usuário.

---

### O que este módulo explicitamente NÃO FAZ
- Não expõe rotas HTTP diretamente.
- Não escreve em feed.
- Não processa mídia.
- Não executa análise semântica ou emocional do domínio `analysis/*`.
- Não chama IA diretamente.
- Não executa autenticação.

---

### Dados que este módulo pode LER
- Estado atual do usuário (IND, EXD, U) por meio de serviços internos.
- Histórico ou eventos dimensionais quando agregados.
- Parâmetros que identificam usuário e operação.

---

### Dados que este módulo pode ESCREVER
- Atualizações em IND.
- Atualizações em EXD.
- Atualizações em U.
- Registros de eventos dimensionais quando aplicável.

---

### Módulos dos quais este módulo DEPENDE
- `userdimensions/ind.service`
- `userdimensions/exd.service`
- `userdimensions/u.service`
- `userdimensions/state.service`
- `userdimensions/event.service`
- `db/prisma` (direta ou indiretamente via serviços)

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `feed`
- `media`
- `profile`
- `identity`
- `capacity`
- `cost`
- `networkai`
- `tokenengine`

---

### Este módulo pode chamar IA?
**Não.**  
Qualquer delegação para IA ocorre fora deste serviço, se existir.

---

### Este módulo pode alterar IND ou U?
**Sim.**  
Este serviço é o ponto central de orquestração para mutações explícitas de IND, EXD e U.

---

### Estado do módulo
Ponto de risco estrutural (atenção, não é acusação) Se userdimensions.service expõe métodos que chamam serviços específicos sem validação consistente (validators/policies), você pode ter: update de IND permitido por um controller e negado por outro regras duplicadas/contraditórias Eu só consigo confirmar isso quando analisarmos: validators/* controllers/* (quais rotas realmente expõem quais métodos)
**Contrato arquitetural fechado.**

## Module: userdimensions/ind.pipeline

### Responsabilidade principal
Executar o pipeline determinístico de atualização do estado IND (Inner Dimension) a partir de entradas explícitas.

---

### O que este módulo FAZ hoje
- Executa o pipeline de atualização do IND.
- Orquestra transformações internas relacionadas ao estado Inner Dimension.
- Aplica regras determinísticas para evolução do IND.
- Consolida entradas explícitas (eventos, estados ou parâmetros) em um novo estado de IND.

---

### O que este módulo explicitamente NÃO FAZ
- Não expõe rotas HTTP.
- Não acessa controllers.
- Não chama IA diretamente.
- Não executa análise semântica livre.
- Não infere intenção humana fora dos inputs explícitos.
- Não escreve em feed.
- Não processa mídia.

---

### Dados que este módulo pode LER
- Estado atual de IND.
- Entradas explícitas provenientes de eventos.
- Entradas de serviços internos do módulo `userdimensions`.
- Tipos e estruturas definidos em `engine/types`.

---

### Dados que este módulo pode ESCREVER
- Novo estado de IND (persistido via service chamador).
- Estruturas intermediárias do pipeline mantidas em memória.

---

### Módulos dos quais este módulo DEPENDE
- `userdimensions/engine/types`
- Funções utilitárias internas do módulo `userdimensions`

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `ai.service`
- `networkai`
- `feed`
- `media`
- `profile`

---

### Este módulo pode chamar IA?
**Não.**  
Aplica apenas regras determinísticas sobre entradas explícitas.

---

### Este módulo pode alterar IND ou U?
**Sim.**  
Este pipeline é responsável por gerar o novo estado de IND a partir das entradas recebidas.

---

### Estado do módulo
Ponto crítico (arquitetural, não erro) Sem engine central: Não há garantia formal de consistência entre IND / EXD / U Cada pipeline pode evoluir com semântica própria Isso não viola o contrato atual, mas define um risco aceito
**Contrato arquitet

## Module: userdimensions/exd.pipeline

### Responsabilidade principal
Executar o pipeline determinístico de atualização do estado EXD (Expression Dimension) a partir de entradas explícitas.

---

### O que este módulo FAZ hoje
- Executa o pipeline de atualização do EXD.
- Consolida entradas explícitas relacionadas à expressão do usuário.
- Aplica regras determinísticas para evolução do estado de EXD.
- Mapeia inputs estruturados para um novo estado expressivo.

---

### O que este módulo explicitamente NÃO FAZ
- Não expõe rotas HTTP.
- Não acessa controllers.
- Não chama IA.
- Não executa análise semântica ou emocional.
- Não infere intenção ou significado humano.
- Não altera IND ou U.
- Não escreve em feed.
- Não processa mídia.

---

### Dados que este módulo pode LER
- Estado atual de EXD.
- Entradas explícitas provenientes de serviços ou eventos do módulo `userdimensions`.
- Tipos e estruturas definidos em `engine/types`.

---

### Dados que este módulo pode ESCREVER
- Novo estado de EXD (persistido via service chamador).
- Estruturas intermediárias do pipeline mantidas em memória.

---

### Módulos dos quais este módulo DEPENDE
- `userdimensions/engine/types`
- Funções utilitárias internas do módulo `userdimensions`

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `ai.service`
- `networkai`
- `feed`
- `media`
- `profile`
- `userdimensions/ind.pipeline`

---

### Este módulo pode chamar IA?
**Não.**  
Aplica apenas regras determinísticas sobre entradas explícitas.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Este pipeline atua exclusivamente sobre o estado EXD.

---

### Estado do módulo
Ponto estrutural (não é erro) Sem engine central, EXD evolui de forma independente. A coerência longitudinal entre IND / EXD / U não é garantida globalmente hoje.
**Contrato arquitetural fechado.**

## Module: userdimensions/ind.service

### Responsabilidade principal
Gerenciar leitura e escrita direta do estado IND do usuário, atuando como fonte operacional de verdade para esta dimensão.

---

### O que este módulo FAZ hoje
- Gerencia leitura e escrita direta do estado IND do usuário.
- Aplica atualizações explícitas de IND a partir de comandos ou inputs válidos.
- Persiste alterações de IND no banco de dados.
- Aciona o `ind.pipeline` quando aplicável.
- Atua como fonte operacional de verdade para IND.

---

### O que este módulo explicitamente NÃO FAZ
- Não expõe rotas HTTP.
- Não chama IA diretamente.
- Não executa análise semântica livre.
- Não infere estado interno sem input explícito.
- Não altera EXD ou U.
- Não escreve em feed.
- Não processa mídia.

---

### Dados que este módulo pode LER
- Estado atual de IND do usuário.
- Inputs explícitos provenientes de controllers ou eventos.
- Tipos definidos em `engine/types`.

---

### Dados que este módulo pode ESCREVER
- Novo estado persistido de IND.
- Registros auxiliares ligados à evolução do IND, quando existirem.

---

### Módulos dos quais este módulo DEPENDE
- `db/prisma`
- `userdimensions/ind.pipeline`
- Tipos internos do módulo `userdimensions`
- `userdimensions/state.service` (se presente para leitura de estado agregado)

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `ai.service`
- `networkai`
- `feed`
- `media`
- `profile`

---

### Este módulo pode chamar IA?
**Não.**  
Opera exclusivamente sobre inputs explícitos e regras determinísticas.

---

### Este módulo pode alterar IND ou U?
**Sim.**  
Este serviço é responsável por aplicar e persistir mutações explícitas do estado IND.

---

### Estado do módulo
Ponto crítico (arquitetural) Como não existe engine central: ind.service é a autoridade final Se outro service escrever IND no futuro, a arquitetura quebra Isso exige disciplina, não código adicional agora.
**Contrato arquitetural fechado.**

## Module: userdimensions/exd.service

### Responsabilidade principal
Gerenciar leitura e escrita direta do estado EXD do usuário, atuando como autoridade operacional da dimensão expressiva.

---

### O que este módulo FAZ hoje
- Gerencia leitura e escrita direta do estado EXD do usuário.
- Aplica atualizações explícitas de EXD a partir de comandos válidos.
- Persiste alterações de EXD no banco de dados.
- Aciona o `exd.pipeline` quando aplicável.
- Atua como autoridade operacional do estado expressivo.

---

### O que este módulo explicitamente NÃO FAZ
- Não expõe rotas HTTP.
- Não chama IA diretamente.
- Não executa análise semântica ou emocional.
- Não infere intenção humana.
- Não altera IND ou U.
- Não escreve em feed.
- Não processa mídia.

---

### Dados que este módulo pode LER
- Estado atual de EXD do usuário.
- Inputs explícitos provenientes de controllers ou eventos.
- Tipos definidos em `engine/types`.

---

### Dados que este módulo pode ESCREVER
- Novo estado persistido de EXD.
- Registros auxiliares relacionados à evolução expressiva, quando existirem.

---

### Módulos dos quais este módulo DEPENDE
- `db/prisma`
- `userdimensions/exd.pipeline`
- Tipos internos do módulo `userdimensions`
- `userdimensions/state.service` (se presente para leitura de estado agregado)

---

### Módulos dos quais este módulo NUNCA deve depender
- `analysis`
- `analysis/*`
- `ai.service`
- `networkai`
- `feed`
- `media`
- `profile`
- `userdimensions/ind.service`

---

### Este módulo pode chamar IA?
**Não.**  
Opera exclusivamente sobre inputs explícitos e regras determinísticas.

---

### Este módulo pode alterar IND ou U?
**Não.**  
Este serviço atua apenas sobre a dimensão EXD, sem mutar IND ou U.

---

### Estado do módulo
Ponto crítico (arquitetural) Assim como IND: Se qualquer outro ponto do sistema escrever EXD diretamente, o contrato quebra A ausência de engine central torna disciplina arquitetural obrigatória
**Contrato arquitetural fechado.**

## Module: userdimensions/u.service

### Responsabilidade principal
Gerenciar leitura, escrita e persistência do estado U (Unity) do usuário como autoridade operacional única.

---

### O que este módulo FAZ hoje
- Lê o estado atual de U do usuário
- Aplica atualizações explícitas de U a partir de comandos ou inputs
- Persiste alterações de U no banco de dados
- Atua como fonte de verdade operacional do estado U

---

### O que este módulo explicitamente NÃO FAZ
- Não chama u.pipeline
- Não consolida ou deriva U a partir de IND ou EXD
- Não executa cálculo longitudinal
- Não executa análise semântica ou emocional
- Não chama IA diretamente
- Não escreve em feed
- Não processa mídia

---

### Dados que este módulo pode LER
- Estado atual de U do usuário
- Inputs explícitos vindos de controllers ou eventos
- Tipos definidos em engine/types.ts
- (Opcionalmente) dados auxiliares via state.service

---

### Dados que este módulo pode ESCREVER
- Novo estado persistido de U
- Campos relacionados à unidade ou coerência do usuário

---

### Módulos dos quais este módulo DEPENDE
- db/prisma
- engine/types
- (Possivelmente) state.service

---

### Módulos dos quais este módulo NUNCA deve depender
- engine/u.pipeline
- analysis/*
- ai.service
- networkai
- feed
- media
- profile
- ind.service
- exd.service

---

### Este módulo pode chamar IA?
**Não.** Não possui qualquer integração ou inferência via IA.

---

### Este módulo pode alterar IND ou U?
**Sim (U apenas).** Atua como autoridade exclusiva de escrita de U, sem alterar IND.

---

### Estado do módulo
Ponto arquitetural CRÍTICO (não é erro)
Hoje, U é:

manual

desacoplado

não derivado

não garantido coerente ao longo do tempo

Isso define um contrato claro, mas frágil:

“U existe como estado declarado, não como síntese.”

Isso não viola o sistema atual — mas define exatamente o que ele é hoje.
**Contrato arquitetural fechado.**

## Module: userdimensions/state.service

### Responsabilidade principal
Fornecer leitura agregada do estado dimensional do usuário (IND, EXD e U) como facade de consulta.

---

### O que este módulo FAZ hoje
- Consolida acesso de leitura a IND, EXD e U
- Agrega estados dimensionais para consumo por controllers e serviços internos
- Retorna estruturas agregadas em memória para resposta

---

### O que este módulo explicitamente NÃO FAZ
- Não escreve ou altera IND, EXD ou U
- Não executa pipelines
- Não chama IA
- Não executa análise semântica ou emocional
- Não gera eventos
- Não escreve em feed
- Não processa mídia

---

### Dados que este módulo pode LER
- Estado atual de IND (via ind.service ou repositório)
- Estado atual de EXD (via exd.service ou repositório)
- Estado atual de U (via u.service ou repositório)
- Tipos internos do userdimensions

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas estruturas agregadas em memória para resposta

---

### Módulos dos quais este módulo DEPENDE
- userdimensions/ind.service
- userdimensions/exd.service
- userdimensions/u.service
- engine/types

---

### Módulos dos quais este módulo NUNCA deve depender
- analysis/*
- engine/*
- ai.service
- networkai
- feed
- media
- profile

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como camada de leitura agregada.

---

### Este módulo pode alterar IND ou U?
**Não.** É estritamente leitura, sem mutação de estado.

---

### Estado do módulo
Ponto estrutural (aceito hoje) Como não existe engine central: O estado agregado não garante coerência semântica É uma fotografia, não uma interpretação Isso está correto para o contrato atual.
**Contrato arquitetural fechado.**

## Module: userdimensions/event.service

### Responsabilidade principal
Registrar e persistir eventos dimensionais explícitos do usuário como fatos históricos auditáveis.

---

### O que este módulo FAZ hoje
- Registra eventos dimensionais explícitos associados ao usuário
- Persiste eventos como registros históricos
- Encaminha eventos para consumo interno quando explicitamente acionado
- Atua como fonte de verdade de eventos, não de estados

---

### O que este módulo explicitamente NÃO FAZ
- Não altera diretamente IND, EXD ou U
- Não executa pipelines
- Não chama IA
- Não executa análise semântica ou emocional
- Não escreve em feed
- Não processa mídia
- Não interpreta o significado do evento

---

### Dados que este módulo pode LER
- Payloads de eventos recebidos dos controllers
- Identificador do usuário
- Tipos de evento definidos no domínio

---

### Dados que este módulo pode ESCREVER
- Registros persistidos de eventos dimensionais
- Metadados associados ao evento (timestamp, tipo, origem)

---

### Módulos dos quais este módulo DEPENDE
- db/prisma
- engine/types

---

### Módulos dos quais este módulo NUNCA deve depender
- analysis/*
- engine/*
- userdimensions/ind.service
- userdimensions/exd.service
- userdimensions/u.service
- ai.service
- networkai
- feed
- media
- profile

---

### Este módulo pode chamar IA?
**Não.** Atua apenas no registro e persistência de eventos.

---

### Este módulo pode alterar IND ou U?
**Não.** Eventos não executam mutações de estado.

---

### Estado do módulo
Ponto de atenção (arquitetural) Se no futuro eventos passarem a disparar mutações automáticas, isso não pode acontecer aqui; deve ocorrer em engine/pipeline explícito. Hoje isso não ocorre.
**Contrato arquitetural fechado.**

## Module: userdimensions/ai.service

### Responsabilidade principal
Encapsular chamadas controladas a serviços de IA para apoio ao domínio userdimensions, sem autoridade sobre estados.

---

### O que este módulo FAZ hoje
- Recebe inputs estruturados relacionados a userdimensions
- Executa chamadas controladas a modelos ou serviços de IA
- Retorna resultados auxiliares em memória para consumo por services ou controllers
- Atua exclusivamente como serviço de apoio

---

### O que este módulo explicitamente NÃO FAZ
- Não altera diretamente IND, EXD ou U
- Não persiste estados dimensionais
- Não executa pipelines de IND, EXD ou U
- Não toma decisões finais sobre estado humano
- Não escreve em feed
- Não processa mídia
- Não executa análise semântica livre fora do escopo declarado

---

### Dados que este módulo pode LER
- Inputs estruturados fornecidos por services ou controllers
- Configurações de IA (modelo, parâmetros)
- Contexto mínimo necessário para execução da chamada

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas respostas retornadas em memória ao chamador

---

### Módulos dos quais este módulo DEPENDE
- Cliente/SDK de IA configurado no projeto
- engine/types
- Configurações de ambiente

---

### Módulos dos quais este módulo NUNCA deve depender
- db/prisma
- engine/*
- analysis/*
- userdimensions/ind.service
- userdimensions/exd.service
- userdimensions/u.service
- networkai
- feed
- media
- profile

---

### Este módulo pode chamar IA?
**Sim.** É o ponto único de encapsulamento de chamadas de IA no domínio userdimensions.

---

### Este módulo pode alterar IND ou U?
**Não.** IA não possui autoridade de mutação de estado.

---

### Estado do módulo
Ponto de risco (arquitetural, não erro) Se algum controller/service: aceitar resposta da IA sem validação humana ou regra explícita ou passar texto livre sensível diretamente isso quebraria o contrato — mas não ocorre neste arquivo.
**Contrato arquitetural fechado.**

## Module: userdimensions/userdimensions.controller

### Responsabilidade principal
Expor endpoint HTTP de leitura agregada das dimensões do usuário como camada de borda sem lógica de domínio.

---

### O que este módulo FAZ hoje
- Expõe endpoint HTTP de leitura agregada das dimensões do usuário
- Recebe userId via parâmetros de rota
- Valida exclusivamente a presença do userId
- Delegada integralmente a obtenção dos dados ao UserDimensionsService
- Retorna o resultado diretamente como resposta HTTP

---

### O que este módulo explicitamente NÃO FAZ
- Não altera estado (IND, EXD, U ou qualquer outro)
- Não executa pipelines
- Não chama IA
- Não processa eventos
- Não executa regras de negócio
- Não valida formato, existência ou elegibilidade do usuário
- Não executa autenticação ou autorização
- Não persiste dados
- Não realiza agregação interna

---

### Dados que este módulo pode LER
- userId proveniente de req.params

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas resposta HTTP (status e payload)

---

### Módulos dos quais este módulo DEPENDE
- express (Request, Response)
- userdimensions/userdimensions.service

---

### Módulos dos quais este módulo NUNCA deve depender
- userdimensions/ind.service
- userdimensions/exd.service
- userdimensions/u.service
- engine/*
- pipelines/*
- ai.service
- camada de autenticação/autorização
- validators dedicados
- logger explícito

---

### Este módulo pode chamar IA?
**Não.** Atua apenas como camada de borda HTTP.

---

### Este módulo pode alterar IND ou U?
**Não.** Controllers não possuem autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato

Caracterizam violação arquitetural direta deste controller:

Introduzir qualquer mutação de estado (IND, EXD, U ou derivados).

Executar ou disparar pipelines (IND, EXD, U ou engine central).

Realizar inferência semântica, emocional ou contextual, incluindo uso de IA.

Processar ou interpretar eventos.

Implementar regras de negócio, validações de domínio ou decisões condicionais.

Centralizar autenticação, autorização ou elegibilidade como lógica interna.

Agregar dados manualmente fora do UserDimensionsService.

Este controller é exclusivamente de leitura agregada e de borda HTTP.
Qualquer expansão funcional além disso quebra o contrato.

Responsabilidades corretamente direcionadas para outras camadas:

Leitura agregada / composição: UserDimensionsService / state.service.

Mutação explícita de estado: ind.service, exd.service, u.service.

Execução determinística: engine/*.pipeline.ts.

Eventos: event.service.

IA (apoio, não autoridade): ai.service.

Autenticação / autorização: camada externa (middlewares / infra), AMBÍGUO neste módulo.

Este alerta é cláusula de proteção arquitetural e invalida qualquer alteração que desloque essas responsabilidades para este controller.
**Contrato arquitetural fechado.**


## Module: userdimensions/ind.controller

### Responsabilidade principal
Expor endpoints HTTP para leitura e escrita explícita da camada IND, atuando como gateway entre HTTP e o serviço de domínio.

---

### O que este módulo FAZ hoje
- Expõe endpoints HTTP relacionados exclusivamente à camada IND
- Recebe userId e payloads explícitos via request
- Encaminha comandos e consultas diretamente para IndService
- Permite leitura e escrita explícita de estado IND conforme métodos expostos

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não interpreta significado, intenção ou estado humano
- Não executa pipelines diretamente
- Não chama IA
- Não agrega dados de outras dimensões (EXD, U)
- Não persiste dados diretamente
- Não reage a eventos
- Não executa validações complexas de domínio

---

### Dados que este módulo pode LER
- userId proveniente de parâmetros de rota ou body
- Payloads explícitos enviados pelo cliente

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido diretamente
- Apenas respostas HTTP
- Escrita indireta ocorre somente via IndService

---

### Módulos dos quais este módulo DEPENDE
- express (Request, Response)
- userdimensions/ind.service

---

### Módulos dos quais este módulo NUNCA deve depender
- userdimensions/exd.service
- userdimensions/u.service
- userdimensions/state.service
- engine/*
- pipelines/*
- ai.service
- event services
- validators dedicados
- logger explícito

---

### Este módulo pode chamar IA?
**Não.** Atua apenas como camada de borda HTTP.

---

### Este módulo pode alterar IND ou U?
**Sim (IND apenas).** A mutação ocorre exclusivamente por delegação explícita ao IndService.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste controller: Introduzir inferência, interpretação ou decisão automática sobre estados internos. Executar pipelines ou engines diretamente. Agregar ou correlacionar IND com EXD ou U. Chamar IA ou serviços externos de análise. Persistir dados fora do IndService. Responsabilidades corretas: Mutação de IND: IndService. Execução determinística: engine/ind.pipeline.ts. Leitura agregada: state.service / UserDimensionsService. IA: ai.service.
**Contrato arquitetural fechado.**

## Module: userdimensions/exd.controller

### Responsabilidade principal
Expor endpoints HTTP para leitura e escrita explícita da camada EXD, atuando como gateway entre HTTP e o serviço de domínio.

---

### O que este módulo FAZ hoje
- Expõe endpoints HTTP relacionados exclusivamente à camada EXD
- Recebe userId e payloads explícitos via request
- Encaminha comandos e consultas diretamente para ExdService
- Permite leitura e escrita explícita de estado EXD conforme métodos expostos

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não interpreta significado expressivo ou intenção humana
- Não executa pipelines diretamente
- Não chama IA
- Não agrega dados de IND ou U
- Não persiste dados diretamente
- Não processa eventos
- Não executa validações complexas de domínio

---

### Dados que este módulo pode LER
- userId proveniente de parâmetros de rota ou body
- Payloads explícitos enviados pelo cliente

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido diretamente
- Apenas respostas HTTP
- Escrita indireta ocorre somente via ExdService

---

### Módulos dos quais este módulo DEPENDE
- express (Request, Response)
- userdimensions/exd.service

---

### Módulos dos quais este módulo NUNCA deve depender
- userdimensions/ind.service
- userdimensions/u.service
- userdimensions/state.service
- engine/*
- pipelines/*
- ai.service
- event services
- validators dedicados
- logger explícito

---

### Este módulo pode chamar IA?
**Não.** Atua apenas como camada de borda HTTP.

---

### Este módulo pode alterar IND ou U?
**Não.** Apenas delega mutação explícita de EXD ao ExdService.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste controller: Executar inferência, interpretação ou decisão sobre expressão humana. Executar pipelines EXD ou engines diretamente. Agregar EXD com IND ou U. Chamar IA ou serviços de análise. Persistir dados fora do ExdService. Responsabilidades corretamente direcionadas: Mutação de EXD: ExdService. Execução determinística: engine/exd.pipeline.ts. Leitura agregada: state.service / UserDimensionsService. IA de apoio: ai.service.
**Contrato arquitetural fechado.**

## Module: userdimensions/u.controller

### Responsabilidade principal
Expor endpoints HTTP para leitura e escrita explícita da camada U (Unity), atuando como gateway entre HTTP e o serviço de domínio.

---

### O que este módulo FAZ hoje
- Expõe endpoints HTTP relacionados exclusivamente à camada U
- Recebe userId e comandos explícitos via request
- Encaminha chamadas diretamente para UService
- Permite leitura e escrita explícita do estado U conforme métodos expostos no service

---

### O que este módulo explicitamente NÃO FAZ
- Não executa inferência automática de transições de consciência
- Não executa pipelines
- Não chama IA
- Não interpreta ou correlaciona IND e EXD
- Não executa lógica de domínio própria
- Não persiste dados diretamente
- Não processa eventos
- Não agrega dados de outras dimensões

---

### Dados que este módulo pode LER
- userId proveniente de parâmetros de rota ou body
- Payloads explícitos enviados pelo cliente

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido diretamente
- Apenas respostas HTTP
- Escrita indireta ocorre somente via UService

---

### Módulos dos quais este módulo DEPENDE
- express (Request, Response)
- userdimensions/u.service

---

### Módulos dos quais este módulo NUNCA deve depender
- userdimensions/ind.service
- userdimensions/exd.service
- userdimensions/state.service
- engine/*
- pipelines/*
- ai.service
- event services
- validators dedicados
- logger explícito

---

### Este módulo pode chamar IA?
**Não.** Atua apenas como camada de borda HTTP.

---

### Este módulo pode alterar IND ou U?
**Sim (U apenas).** A mutação ocorre exclusivamente por delegação explícita ao UService.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste controller: Implementar inferência de estado U. Executar qualquer forma de pipeline. Correlacionar U com IND ou EXD. Chamar IA ou serviços de análise. Persistir dados fora do UService. Responsabilidades corretamente direcionadas: Escrita e leitura de U: UService. Leitura agregada longitudinal: state.service / UserDimensionsService. IA de apoio (não autoridade): ai.service.
**Contrato arquitetural fechado.**

## Module: userdimensions/state.controller

### Responsabilidade principal
Expor endpoint(s) HTTP para leitura agregada do estado dimensional do usuário como camada de borda.

---

### O que este módulo FAZ hoje
- Expõe endpoint(s) HTTP para leitura agregada do estado do usuário
- Recebe userId via parâmetros de rota
- Encaminha a consulta diretamente para StateService
- Retorna o estado agregado como resposta HTTP

---

### O que este módulo explicitamente NÃO FAZ
- Não altera estado (IND, EXD, U)
- Não executa pipelines
- Não chama IA
- Não interpreta eventos
- Não executa lógica de domínio
- Não persiste dados
- Não realiza agregação própria

---

### Dados que este módulo pode LER
- userId proveniente de req.params

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas resposta HTTP (status e payload)

---

### Módulos dos quais este módulo DEPENDE
- express (Request, Response)
- userdimensions/state.service

---

### Módulos dos quais este módulo NUNCA deve depender
- userdimensions/ind.service
- userdimensions/exd.service
- userdimensions/u.service
- engine/*
- pipelines/*
- ai.service
- userdimensions/event.service
- validators dedicados
- logger explícito

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como fachada HTTP de leitura.

---

### Este módulo pode alterar IND ou U?
**Não.** É estritamente leitura agregada.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste controller: Introduzir qualquer mutação de estado. Executar pipelines ou engines. Inferir, classificar ou decidir estado humano. Agregar manualmente IND, EXD e U fora do StateService. Chamar IA ou serviços de análise. Responsabilidades corretamente direcionadas: Agregação de estado: StateService. Mutação explícita: IndService, ExdService, UService. Execução determinística: engine/*.pipeline.ts. IA de apoio: ai.service.
**Contrato arquitetural fechado.**

## Module: userdimensions/event.controller

### Responsabilidade principal
Expor endpoint(s) HTTP para entrada explícita de eventos do usuário, atuando como camada de borda para o EventService.

---

### O que este módulo FAZ hoje
- Expõe endpoint(s) HTTP para registro de eventos relacionados ao usuário
- Recebe userId e payload de evento via request
- Delegada integralmente o processamento ao EventService
- Retorna resposta HTTP baseada no resultado do encaminhamento

---

### O que este módulo explicitamente NÃO FAZ
- Não altera estado diretamente (IND, EXD, U)
- Não executa pipelines
- Não interpreta ou classifica eventos
- Não executa inferência semântica ou emocional
- Não chama IA
- Não executa lógica de domínio
- Não persiste dados diretamente
- Não agrega estado

---

### Dados que este módulo pode LER
- userId proveniente de parâmetros de rota ou body
- Payload explícito do evento enviado pelo cliente

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas resposta HTTP
- Escrita indireta ocorre somente via EventService

---

### Módulos dos quais este módulo DEPENDE
- express (Request, Response)
- userdimensions/event.service

---

### Módulos dos quais este módulo NUNCA deve depender
- userdimensions/ind.service
- userdimensions/exd.service
- userdimensions/u.service
- engine/*
- pipelines/*
- userdimensions/state.service
- ai.service
- validators dedicados
- logger explícito

---

### Este módulo pode chamar IA?
**Não.** Atua apenas como gateway HTTP para eventos explícitos.

---

### Este módulo pode alterar IND ou U?
**Não.** Eventos não produzem mutação direta nesta camada.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste controller: Mutar estado diretamente a partir de eventos. Inferir significado ou intenção a partir do evento. Executar pipelines ou engines. Correlacionar eventos com estado dimensional. Chamar IA ou serviços analíticos. Responsabilidades corretamente direcionadas: Tratamento e roteamento de eventos: EventService. Mutação explícita de estado: IndService, ExdService, UService (se acionados). Execução determinística: engine/*.pipeline.ts. IA de apoio (não autoridade): ai.service.
**Contrato arquitetural fechado.**

## Module: userdimensions/ai.controller

### Responsabilidade principal
Expor endpoint(s) HTTP para chamadas controladas de IA no contexto de userdimensions, como camada de borda consultiva.

---

### O que este módulo FAZ hoje
- Expõe endpoint(s) HTTP para chamadas auxiliares de IA
- Recebe userId e inputs estruturados via request
- Encaminha a requisição diretamente para AIService
- Retorna o resultado da chamada de IA como resposta HTTP
- Opera exclusivamente em modo consultivo, sem autoridade de estado

---

### O que este módulo explicitamente NÃO FAZ
- Não altera estado (IND, EXD, U)
- Não executa pipelines
- Não decide transições de estado humano
- Não interpreta resultados de IA como verdade de domínio
- Não persiste dados
- Não agrega estado
- Não processa eventos
- Não executa lógica de domínio própria

---

### Dados que este módulo pode LER
- userId proveniente de parâmetros de rota ou body
- Inputs estruturados enviados pelo cliente
- Contexto mínimo necessário para a chamada de IA

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas resposta HTTP contendo o resultado da IA

---

### Módulos dos quais este módulo DEPENDE
- express (Request, Response)
- userdimensions/ai.service

---

### Módulos dos quais este módulo NUNCA deve depender
- userdimensions/ind.service
- userdimensions/exd.service
- userdimensions/u.service
- userdimensions/state.service
- engine/*
- pipelines/*
- userdimensions/event.service
- validators dedicados
- logger explícito

---

### Este módulo pode chamar IA?
**Sim.** Atua como camada de borda HTTP para acesso controlado à IA auxiliar.

---

### Este módulo pode alterar IND ou U?
**Não.** Resultados de IA não possuem autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste controller: Tratar IA como fonte de verdade ou autoridade de estado. Persistir resultados de IA automaticamente. Disparar pipelines ou mutações com base direta em IA. Correlacionar resultados de IA com IND, EXD ou U. Executar decisões de domínio ou classificação humana. Responsabilidades corretamente direcionadas: IA auxiliar e controlada: AIService. Decisão e mutação explícita: services de domínio (IndService, ExdService, UService). Execução determinística: engine/*.pipeline.ts. Agregação de leitura: state.service / UserDimensionsService
**Contrato arquitetural fechado.**

## Module: userdimensions/userdimensions.routes

### Responsabilidade principal
Definir e registrar as rotas HTTP base do módulo userdimensions, mapeando paths para controllers.

---

### O que este módulo FAZ hoje
- Define as rotas HTTP base do módulo userdimensions
- Associa paths e métodos HTTP ao userdimensions.controller
- Atua como ponto de entrada agregado para operações gerais do módulo
- Registra rotas no Router do Express

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não altera estado (IND, EXD, U)
- Não executa pipelines
- Não chama IA
- Não valida payloads
- Não executa autenticação ou autorização
- Não agrega dados manualmente
- Não persiste dados

---

### Dados que este módulo pode LER
- Definições estáticas de rotas (paths e métodos)
- Controller importado (userdimensions.controller)

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas registro de rotas no Router do Express

---

### Módulos dos quais este módulo DEPENDE
- express (Router)
- userdimensions/userdimensions.controller

---

### Módulos dos quais este módulo NUNCA deve depender
- userdimensions/ind.service
- userdimensions/exd.service
- userdimensions/u.service
- engine/*
- pipelines/*
- validators
- middlewares de autenticação/autorização
- logger explícito
- ai.service

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como camada de roteamento HTTP.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui qualquer autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Inserir lógica de negócio ou decisões condicionais. Executar validações de domínio. Mutar estado ou disparar pipelines. Tratar a rota como camada de orquestração. Responsabilidades corretamente direcionadas: Roteamento HTTP: routes. Controle de fluxo: controllers. Lógica e decisão: services. Execução determinística: engine.
**Contrato arquitetural fechado.**

## Module: userdimensions/ind.routes

### Responsabilidade principal
Definir e registrar as rotas HTTP específicas da dimensão IND, mapeando paths para o ind.controller.

---

### O que este módulo FAZ hoje
- Define as rotas HTTP específicas da dimensão IND
- Mapeia métodos e paths HTTP para o ind.controller
- Separa a superfície HTTP da dimensão IND das demais dimensões
- Registra rotas no Router do Express

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não altera estado diretamente
- Não executa pipelines
- Não chama IA
- Não valida payloads
- Não executa autenticação ou autorização
- Não agrega dados entre dimensões
- Não persiste dados

---

### Dados que este módulo pode LER
- Definições estáticas de rotas (paths e métodos HTTP)
- ind.controller

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas registro de rotas no Router do Express

---

### Módulos dos quais este módulo DEPENDE
- express (Router)
- userdimensions/ind.controller

---

### Módulos dos quais este módulo NUNCA deve depender
- userdimensions/ind.service
- engine/*
- pipelines/*
- validators
- ai.service
- middlewares explícitos
- logger explícito

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como camada de roteamento HTTP.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Inserir lógica de domínio ou validação. Executar mutações ou pipelines. Tratar rotas como camada de decisão. Introduzir inferência ou agregação. Responsabilidades corretamente direcionadas: Roteamento HTTP: ind.routes.ts. Controle de fluxo: ind.controller. Lógica e mutação: ind.service. Execução determinística: engine/ind.pipeline.ts.
**Contrato arquitetural fechado.**

## Module: userdimensions/exd.routes

### Responsabilidade principal
Definir e registrar as rotas HTTP específicas da dimensão EXD, mapeando paths para o exd.controller.

---

### O que este módulo FAZ hoje
- Define as rotas HTTP específicas da dimensão EXD
- Mapeia métodos e paths HTTP para o exd.controller
- Mantém a superfície HTTP de EXD isolada das demais dimensões
- Registra rotas no Router do Express

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não altera estado diretamente
- Não executa pipelines
- Não chama IA
- Não valida payloads
- Não executa autenticação ou autorização
- Não agrega dados entre dimensões
- Não persiste dados

---

### Dados que este módulo pode LER
- Definições estáticas de rotas (paths e métodos HTTP)
- exd.controller

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas registro de rotas no Router do Express

---

### Módulos dos quais este módulo DEPENDE
- express (Router)
- userdimensions/exd.controller

---

### Módulos dos quais este módulo NUNCA deve depender
- userdimensions/exd.service
- engine/*
- pipelines/*
- validators
- ai.service
- middlewares explícitos
- logger explícito

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como camada de roteamento HTTP.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Inserir lógica de domínio ou validações. Executar mutações ou pipelines. Introduzir inferência ou correlação. Tratar rotas como camada de decisão. Responsabilidades corretamente direcionadas: Roteamento HTTP: exd.routes.ts. Controle de fluxo: exd.controller. Mutação explícita: exd.service. Execução determinística: engine/exd.pipeline.ts.
**Contrato arquitetural fechado.**

## Module: userdimensions/u.routes

### Responsabilidade principal
Definir e registrar as rotas HTTP específicas da camada U (Unity), mapeando paths para o u.controller.

---

### O que este módulo FAZ hoje
- Define as rotas HTTP específicas da camada U
- Mapeia métodos e paths HTTP para o u.controller
- Isola a superfície HTTP da camada U das demais dimensões
- Registra rotas no Router do Express

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não altera estado diretamente
- Não executa pipelines
- Não chama IA
- Não valida payloads
- Não executa autenticação ou autorização
- Não agrega dados entre dimensões
- Não persiste dados

---

### Dados que este módulo pode LER
- Definições estáticas de rotas (paths e métodos HTTP)
- u.controller

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas registro de rotas no Router do Express

---

### Módulos dos quais este módulo DEPENDE
- express (Router)
- userdimensions/u.controller

---

### Módulos dos quais este módulo NUNCA deve depender
- userdimensions/u.service
- engine/*
- pipelines/*
- validators
- ai.service
- middlewares explícitos
- logger explícito

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como camada de roteamento HTTP.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Inserir lógica de domínio ou validação. Executar mutações ou inferências. Tratar rotas como camada de decisão. Introduzir pipelines ou correlação entre dimensões. Responsabilidades corretamente direcionadas: Roteamento HTTP: u.routes.ts. Controle de fluxo: u.controller. Escrita e leitura de U: u.service.
**Contrato arquitetural fechado.**

## Module: userdimensions/state.routes

### Responsabilidade principal
Definir e registrar as rotas HTTP específicas para leitura do estado dimensional agregado do usuário.

---

### O que este módulo FAZ hoje
- Define as rotas HTTP específicas para leitura de estado agregado
- Mapeia métodos e paths HTTP para o state.controller
- Expõe a superfície HTTP de acesso ao estado consolidado
- Registra rotas no Router do Express

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não altera estado (IND, EXD, U)
- Não executa pipelines
- Não chama IA
- Não valida payloads
- Não executa autenticação ou autorização
- Não agrega dados manualmente
- Não persiste dados

---

### Dados que este módulo pode LER
- Definições estáticas de rotas (paths e métodos HTTP)
- state.controller

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas registro de rotas no Router do Express

---

### Módulos dos quais este módulo DEPENDE
- express (Router)
- userdimensions/state.controller

---

### Módulos dos quais este módulo NUNCA deve depender
- userdimensions/ind.service
- userdimensions/exd.service
- userdimensions/u.service
- engine/*
- pipelines/*
- validators
- ai.service
- middlewares explícitos
- logger explícito

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como camada de roteamento HTTP.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Inserir lógica de domínio ou validações. Executar mutações ou pipelines. Agregar manualmente estado dimensional. Tratar rotas como camada de decisão. Responsabilidades corretamente direcionadas: Roteamento HTTP: state.routes.ts. Controle de fluxo: state.controller. Agregação de leitura: state.service.
**Contrato arquitetural fechado.**

## Module: userdimensions/event.routes

### Responsabilidade principal
Definir e registrar as rotas HTTP específicas para entrada e encaminhamento de eventos no módulo userdimensions.

---

### O que este módulo FAZ hoje
- Define as rotas HTTP específicas para registro de eventos
- Mapeia métodos e paths HTTP para o event.controller
- Expõe a superfície HTTP de entrada de eventos
- Registra rotas no Router do Express

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não altera estado (IND, EXD, U)
- Não executa pipelines
- Não interpreta ou classifica eventos
- Não chama IA
- Não valida payloads
- Não executa autenticação ou autorização
- Não persiste dados

---

### Dados que este módulo pode LER
- Definições estáticas de rotas (paths e métodos HTTP)
- event.controller

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas registro de rotas no Router do Express

---

### Módulos dos quais este módulo DEPENDE
- express (Router)
- userdimensions/event.controller

---

### Módulos dos quais este módulo NUNCA deve depender
- userdimensions/ind.service
- userdimensions/exd.service
- userdimensions/u.service
- userdimensions/state.service
- engine/*
- pipelines/*
- validators
- ai.service
- middlewares explícitos
- logger explícito

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como camada de roteamento HTTP.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui qualquer autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Processar ou interpretar eventos. Executar mutações de estado. Disparar pipelines. Inferir significado ou intenção humana. Tratar rotas como camada de decisão. Responsabilidades corretamente direcionadas: Roteamento HTTP: event.routes.ts. Encaminhamento de eventos: event.controller. Processamento de eventos: event.service. Mutação explícita (se houver): services de domínio. Execução determinística: engines.
**Contrato arquitetural fechado.**

## Module: userdimensions/ai.routes

### Responsabilidade principal
Definir e registrar as rotas HTTP específicas para acesso às capacidades auxiliares de IA no contexto de userdimensions.

---

### O que este módulo FAZ hoje
- Define as rotas HTTP específicas para chamadas auxiliares de IA
- Mapeia métodos e paths HTTP para o ai.controller
- Expõe a superfície HTTP para acesso às capacidades de IA
- Registra rotas no Router do Express

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não altera estado (IND, EXD, U)
- Não executa pipelines
- Não interpreta resultados de IA
- Não valida payloads
- Não executa autenticação ou autorização
- Não persiste dados
- Não agrega estado

---

### Dados que este módulo pode LER
- Definições estáticas de rotas (paths e métodos HTTP)
- ai.controller

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas registro de rotas no Router do Express

---

### Módulos dos quais este módulo DEPENDE
- express (Router)
- userdimensions/ai.controller

---

### Módulos dos quais este módulo NUNCA deve depender
- userdimensions/ind.service
- userdimensions/exd.service
- userdimensions/u.service
- userdimensions/state.service
- engine/*
- pipelines/*
- validators
- middlewares explícitos
- logger explícito

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como camada de roteamento HTTP.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui qualquer autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Processar ou interpretar resultados de IA. Executar mutações de estado. Disparar pipelines. Introduzir inferência ou decisão de domínio. Tratar rotas como camada de decisão. Responsabilidades corretamente direcionadas: Roteamento HTTP: ai.routes.ts. Controle de fluxo: ai.controller. IA auxiliar (não autoridade): ai.service. Decisão e mutação: services de domínio.
**Contrato arquitetural fechado.**

## Module: userdimensions/userdimensions.dto

### Responsabilidade principal
Definir contratos estruturais de dados (DTOs) utilizados nas interfaces do módulo userdimensions, sem lógica executável.

---

### O que este módulo FAZ hoje
- Define a forma estrutural dos payloads aceitos e retornados no módulo userdimensions
- Padroniza a comunicação entre routes, controllers e services
- Atua como camada passiva de tipagem em tempo de compilação

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não valida regras de negócio
- Não altera estado
- Não executa pipelines
- Não chama IA
- Não persiste dados
- Não processa eventos
- Não executa validação em runtime

---

### Dados que este módulo pode LER
- Nenhum dado em runtime
- Utilizado apenas em tempo de compilação (TypeScript)

---

### Dados que este módulo pode ESCREVER
- Nenhum
- Não possui efeitos colaterais

---

### Módulos dos quais este módulo DEPENDE
- Tipos internos do TypeScript

---

### Módulos dos quais este módulo NUNCA deve depender
- userdimensions/services/*
- engine/*
- controllers/*
- ai.service
- validators de runtime
- qualquer dependência de infraestrutura

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como contrato estrutural de dados.

---

### Este módulo pode alterar IND ou U?
**Não.** DTOs não possuem qualquer autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Inserir lógica de domínio em DTOs. Realizar validações semânticas ou condicionais. Acoplar DTOs a services, engines ou IA. Utilizar DTOs como objetos de persistência. Responsabilidades corretamente direcionadas: Validação de runtime: validators/*. Decisão e lógica: services. Execução determinística: engines. Persistência: camada de dados.
**Contrato arquitetural fechado.**

## Module: userdimensions/validators/createEvent.validator

### Responsabilidade principal
Nenhuma. Arquivo presente apenas como placeholder estrutural, sem implementação ativa.

---

### O que este módulo FAZ hoje
- Nada
- Existe fisicamente sem expor funções, schemas ou validações

---

### O que este módulo explicitamente NÃO FAZ
- Não valida payload de evento
- Não executa validação estrutural
- Não executa validação semântica
- Não bloqueia requests inválidos
- Não interage com controllers, services ou routes
- Não produz efeitos colaterais

---

### Dados que este módulo pode LER
- Nenhum

---

### Dados que este módulo pode ESCREVER
- Nenhum

---

### Módulos dos quais este módulo DEPENDE
- Nenhum

---

### Módulos dos quais este módulo NUNCA deve depender
- Libraries de validação (zod, joi, yup)
- controllers/*
- services/*
- engine/*
- ai.service
- infraestrutura

---

### Este módulo pode chamar IA?
**Não.** Não possui qualquer implementação.

---

### Este módulo pode alterar IND ou U?
**Não.** Não executa código nem possui autoridade de mutação.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural futura deste subdomínio: Introduzir lógica de domínio neste arquivo. Executar mutação de estado a partir de validação. Acoplar validator a services, engines ou IA. Inferir significado do evento durante validação. Responsabilidades corretamente direcionadas: Validação estrutural de eventos (se existir): validators. Decisão e mutação: services. Processamento de eventos: event.service. Execução determinística: engines.
**Contrato arquitetural fechado.**

## Module: userdimensions/validators/updateInd.validator

### Responsabilidade principal
Nenhuma. Arquivo presente apenas como placeholder estrutural, sem implementação ativa.

---

### O que este módulo FAZ hoje
- Nada
- Existe fisicamente sem definir funções, schemas ou regras de validação

---

### O que este módulo explicitamente NÃO FAZ
- Não valida payloads de atualização de IND
- Não executa validação estrutural ou semântica
- Não bloqueia requests inválidos
- Não interage com controllers, services ou routes
- Não produz efeitos colaterais

---

### Dados que este módulo pode LER
- Nenhum

---

### Dados que este módulo pode ESCREVER
- Nenhum

---

### Módulos dos quais este módulo DEPENDE
- Nenhum

---

### Módulos dos quais este módulo NUNCA deve depender
- Bibliotecas de validação
- controllers/*
- services/*
- engine/*
- ai.service
- infraestrutura

---

### Este módulo pode chamar IA?
**Não.** Não possui qualquer implementação.

---

### Este módulo pode alterar IND ou U?
**Não.** Não executa código nem possui autoridade de mutação.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural futura deste subdomínio: Introduzir lógica de domínio neste arquivo. Executar mutação de estado durante validação. Acoplar validator a services, engines ou IA. Inferir significado humano na validação. Responsabilidades corretamente direcionadas: Validação estrutural (se existir): validators. Decisão e mutação: services. Execução determinística: engines.
**Contrato arquitetural fechado.**

## Module: userdimensions/validators/updateInd.validator

### Responsabilidade principal
Nenhuma. Arquivo presente apenas como placeholder estrutural, sem implementação ativa.

---

### O que este módulo FAZ hoje
- Nada
- Existe fisicamente sem definir funções, schemas ou regras de validação

---

### O que este módulo explicitamente NÃO FAZ
- Não valida payloads de atualização de IND
- Não executa validação estrutural ou semântica
- Não bloqueia requests inválidos
- Não interage com controllers, services ou routes
- Não produz efeitos colaterais

---

### Dados que este módulo pode LER
- Nenhum

---

### Dados que este módulo pode ESCREVER
- Nenhum

---

### Módulos dos quais este módulo DEPENDE
- Nenhum

---

### Módulos dos quais este módulo NUNCA deve depender
- Bibliotecas de validação
- controllers/*
- services/*
- engine/*
- ai.service
- infraestrutura

---

### Este módulo pode chamar IA?
**Não.** Não possui qualquer implementação.

---

### Este módulo pode alterar IND ou U?
**Não.** Não executa código nem possui autoridade de mutação.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural futura deste subdomínio: Introduzir lógica de domínio neste arquivo. Executar mutação de estado durante validação. Acoplar validator a services, engines ou IA. Inferir significado humano na validação. Responsabilidades corretamente direcionadas: Validação estrutural (se existir): validators. Decisão e mutação: services. Execução determinística: engines.
**Contrato arquitetural fechado.**

## Module: userdimensions/validators/updateU.validator

### Responsabilidade principal
Nenhuma. Arquivo presente apenas como placeholder estrutural, sem implementação ativa.

---

### O que este módulo FAZ hoje
- Nada
- Existe fisicamente sem definir funções, schemas ou regras de validação

---

### O que este módulo explicitamente NÃO FAZ
- Não valida payloads de atualização da camada U
- Não executa validação estrutural ou semântica
- Não bloqueia requests inválidos
- Não interage com controllers, services ou routes
- Não produz efeitos colaterais

---

### Dados que este módulo pode LER
- Nenhum

---

### Dados que este módulo pode ESCREVER
- Nenhum

---

### Módulos dos quais este módulo DEPENDE
- Nenhum

---

### Módulos dos quais este módulo NUNCA deve depender
- Bibliotecas de validação
- controllers/*
- services/*
- engine/*
- ai.service
- infraestrutura

---

### Este módulo pode chamar IA?
**Não.** Não possui qualquer implementação.

---

### Este módulo pode alterar IND ou U?
**Não.** Não executa código nem possui autoridade de mutação.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural futura deste subdomínio: Inserir lógica de domínio neste arquivo. Executar mutação de estado durante validação. Acoplar validator a services, engines ou IA. Inferir significado existencial ou decisões de estado. Responsabilidades corretamente direcionadas: Validação estrutural (se existir): validators. Decisão e mutação: services. Execução determinística: engines.
**Contrato arquitetural fechado.**

## Module: userdimensions/index

### Responsabilidade principal
Centralizar e expor publicamente os exports do módulo userdimensions como ponto único de entrada.

---

### O que este módulo FAZ hoje
- Centraliza a exportação pública do módulo userdimensions
- Reexporta rotas, services, controllers e demais artefatos internos
- Atua como ponto único de entrada do módulo para consumo externo
- Agrega exports sem executar lógica

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não altera estado
- Não executa pipelines
- Não chama IA
- Não valida dados
- Não persiste dados
- Não processa eventos
- Não define contratos novos além dos já existentes

---

### Dados que este módulo pode LER
- Definições estáticas de exports dos subdomínios internos

---

### Dados que este módulo pode ESCREVER
- Nenhum
- Não possui efeitos colaterais em runtime

---

### Módulos dos quais este módulo DEPENDE
- userdimensions/routes/*
- userdimensions/controllers/*
- userdimensions/services/*
- userdimensions/engine/*
- userdimensions/dtos/*
- userdimensions/validators/*
- sistema de módulos TypeScript

---

### Módulos dos quais este módulo NUNCA deve depender
- infraestrutura
- engines externos
- ai.service
- serviços externos
- persistência
- validação em runtime

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como agregador passivo de exports.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui qualquer comportamento executável.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Introduzir código executável. Chamar services ou controllers. Implementar lógica de inicialização. Criar dependências externas diretas. Responsabilidades corretamente direcionadas: Organização e exposição: index.ts. Comportamento e decisão: services / controllers. Execução determinística: engines.
**Contrato arquitetural fechado.**

## Module: userdimensions/README

### Responsabilidade principal
Fornecer documentação mínima e descritiva do módulo userdimensions, sem caráter normativo.

---

### O que este módulo FAZ hoje
- Define uma descrição mínima do módulo userdimensions
- Declara o módulo como stub
- Indica genericamente a existência de controllers, services e routes
- Atua como documentação passiva para leitura humana

---

### O que este módulo explicitamente NÃO FAZ
- Não descreve contratos arquiteturais detalhados
- Não documenta invariantes
- Não define responsabilidades formais por camada
- Não descreve fluxos de dados
- Não define regras de negócio
- Não influencia comportamento em runtime

---

### Dados que este módulo pode LER
- Nenhum dado em runtime
- Apenas leitura humana

---

### Dados que este módulo pode ESCREVER
- Nenhum
- Não possui efeito colateral técnico

---

### Módulos dos quais este módulo DEPENDE
- Nenhum

---

### Módulos dos quais este módulo NUNCA deve depender
- Código executável
- Módulos internos
- Infraestrutura
- IA
- Engines ou pipelines

---

### Este módulo pode chamar IA?
**Não.** Trata-se apenas de documentação estática.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui qualquer efeito técnico.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Tratar o README como especificação normativa. Derivar decisões técnicas a partir deste texto. Atualizar código para “obedecer” o README. Responsabilidades corretamente direcionadas: Fonte de verdade arquitetural: código. Documentação futura (se existir): README revisado a posteriori. Contratos formais: MODULES.md.
**Contrato arquitetural fechado.**

## Module: meta/meta.controller

### Responsabilidade principal
Expor endpoints HTTP informativos com metadados operacionais da aplicação, sem lógica de domínio.

---

### O que este módulo FAZ hoje
- Expõe endpoint(s) HTTP relacionados a metadados operacionais
- Responde com informações estáticas ou diagnósticas (status, versão, identificação)
- Atua como camada de borda entre HTTP e informações internas não sensíveis

---

### O que este módulo explicitamente NÃO FAZ
- Não altera estado
- Não executa lógica de negócio
- Não executa pipelines
- Não chama IA
- Não processa eventos
- Não persiste dados
- Não realiza agregações complexas
- Não executa autenticação ou autorização explícita

---

### Dados que este módulo pode LER
- Constantes internas
- Variáveis de ambiente expostas no código
- Informações estáticas do processo

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas resposta HTTP com payload informativo

---

### Módulos dos quais este módulo DEPENDE
- Framework HTTP (controller)
- process.env ou constantes locais

---

### Módulos dos quais este módulo NUNCA deve depender
- Services de domínio
- Engines ou pipelines
- IA
- Persistência
- Validators
- Filas, workers ou queues

---

### Este módulo pode chamar IA?
**Não.** Atua apenas como endpoint informativo.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui qualquer autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste controller: Expor informações sensíveis de infraestrutura. Introduzir lógica de negócio. Mutar estado ou disparar pipelines. Integrar com IA, services ou persistência. Tornar-se ponto de decisão da aplicação. Responsabilidades corretamente direcionadas: Informações operacionais: meta.controller. Lógica de domínio: outros módulos. Configuração de módulo: meta.module.ts.
**Contrato arquitetural fechado.**

## Module: meta/meta.routes

### Responsabilidade principal
Definir e registrar as rotas HTTP do módulo meta, mapeando endpoints informativos para o meta.controller.

---

### O que este módulo FAZ hoje
- Define o mapeamento de rotas HTTP do módulo meta
- Associa paths e métodos HTTP ao meta.controller
- Expõe endpoints de informação e metadados operacionais
- Registra rotas no Router do framework HTTP

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não altera estado
- Não executa pipelines
- Não chama IA
- Não valida payloads
- Não processa eventos
- Não persiste dados
- Não executa autenticação ou autorização

---

### Dados que este módulo pode LER
- Definições estáticas de rotas (paths e métodos HTTP)
- meta.controller

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas registro das rotas no Router do framework HTTP

---

### Módulos dos quais este módulo DEPENDE
- Framework HTTP (Router)
- meta/meta.controller

---

### Módulos dos quais este módulo NUNCA deve depender
- Services de domínio
- Engines ou pipelines
- IA
- Persistência
- Validators
- Filas, workers ou queues

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como camada declarativa de roteamento.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui qualquer autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Inserir lógica de negócio nas rotas. Expor dados sensíveis via roteamento. Executar mutações ou pipelines. Tratar rotas como camada de decisão. Responsabilidades corretamente direcionadas: Roteamento HTTP: meta.routes.ts. Resposta informativa: meta.controller. Configuração do módulo: meta.module.ts.
**Contrato arquitetural fechado.**

## Module: meta/meta.module

### Responsabilidade principal
Configurar e registrar o módulo meta no framework de aplicação, conectando controllers e rotas sem comportamento próprio.

---

### O que este módulo FAZ hoje
- Define o módulo de composição do domínio meta
- Registra controllers e rotas associados
- Integra o módulo meta ao framework de aplicação
- Atua exclusivamente como camada de configuração

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não altera estado
- Não executa pipelines
- Não chama IA
- Não processa eventos
- Não persiste dados
- Não executa validações
- Não contém regras de negócio

---

### Dados que este módulo pode LER
- Metadados de configuração do framework
- Referências estáticas a controllers e rotas

---

### Dados que este módulo pode ESCREVER
- Nenhum
- Não possui efeitos colaterais em runtime

---

### Módulos dos quais este módulo DEPENDE
- Framework de módulos da aplicação
- meta/meta.controller
- meta/meta.routes

---

### Módulos dos quais este módulo NUNCA deve depender
- Services de domínio
- Engines ou pipelines
- IA
- Persistência
- Validators
- Filas, workers ou queues

---

### Este módulo pode chamar IA?
**Não.** É puramente configuracional.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui comportamento executável.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Inserir lógica de negócio no módulo. Registrar services com comportamento ativo. Executar código no momento de inicialização. Acoplar o módulo a domínios não relacionados. Responsabilidades corretamente direcionadas: Configuração do módulo: meta.module.ts. Resposta informativa: meta.controller. Roteamento HTTP: meta.routes.ts.
**Contrato arquitetural fechado.**


## Module: networkai/networkai.controller

### Responsabilidade principal
Expor endpoints HTTP para acesso às capacidades de IA em rede, atuando como camada de borda entre HTTP e o NetworkAIService.

---

### O que este módulo FAZ hoje
- Expõe endpoint(s) HTTP relacionados às capacidades de IA em rede
- Encaminha comandos explícitos para o NetworkAIService
- Retorna respostas HTTP baseadas no resultado do service
- Atua exclusivamente como gateway HTTP → service

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio própria
- Não toma decisões autônomas de moderação ou ação em rede
- Não executa pipelines
- Não chama diretamente agentes internos
- Não persiste dados
- Não processa eventos de forma autônoma
- Não executa validações complexas de domínio

---

### Dados que este módulo pode LER
- Payloads explícitos enviados via request
- Parâmetros de rota, se existentes

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido diretamente
- Apenas respostas HTTP

---

### Módulos dos quais este módulo DEPENDE
- Framework HTTP (controller)
- networkai/networkai.service

---

### Módulos dos quais este módulo NUNCA deve depender
- Persistência direta
- engine/*
- pipelines/*
- IA externa direta
- validators dedicados
- filas, workers ou queues

---

### Este módulo pode chamar IA?
**Não.** Atua apenas como camada de borda, delegando chamadas ao service.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste controller: Executar lógica de moderação diretamente. Chamar agentes internos ou IA diretamente. Persistir dados. Tornar-se autoridade de decisão em rede. Responsabilidades corretamente direcionadas: Orquestração e decisão: networkai.service. Execução especializada: agents/*. Exposição HTTP: networkai.controller.
**Contrato arquitetural fechado.**

## Module: networkai/networkai.routes

### Responsabilidade principal
Definir e registrar as rotas HTTP do módulo networkai, mapeando endpoints para o networkai.controller.

---

### O que este módulo FAZ hoje
- Define o mapeamento de rotas HTTP do módulo networkai
- Associa paths e métodos HTTP ao networkai.controller
- Expõe a superfície HTTP para acesso às capacidades de IA em rede
- Registra rotas no Router do framework HTTP

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não altera estado
- Não executa pipelines
- Não chama IA ou agentes diretamente
- Não valida payloads
- Não processa eventos
- Não persiste dados
- Não executa autenticação ou autorização explícita

---

### Dados que este módulo pode LER
- Definições estáticas de rotas (paths e métodos HTTP)
- networkai.controller

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas registro das rotas no Router do framework HTTP

---

### Módulos dos quais este módulo DEPENDE
- Framework HTTP (Router)
- networkai/networkai.controller

---

### Módulos dos quais este módulo NUNCA deve depender
- Services de domínio
- Engines ou pipelines
- Agentes de IA
- Persistência
- Validators
- Filas, workers ou queues

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como camada declarativa de roteamento.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui qualquer autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Inserir lógica de negócio nas rotas. Executar decisões de moderação. Chamar agentes ou IA diretamente. Persistir dados ou disparar pipelines. Tratar rotas como camada de decisão. Responsabilidades corretamente direcionadas: Roteamento HTTP: networkai.routes.ts. Exposição de endpoints: networkai.controller. Orquestração e decisão: networkai.service.
**Contrato arquitetural fechado.**

## Module: networkai/networkai.service

### Responsabilidade principal
Orquestrar as capacidades de IA em rede, coordenando a execução de agentes especializados a partir de comandos explícitos.

---

### O que este módulo FAZ hoje
- Recebe comandos explícitos encaminhados pelo networkai.controller
- Centraliza a orquestração das capacidades de IA em rede
- Coordena a execução de agentes especializados quando aplicável
- Retorna resultados estruturados ao controller

---

### O que este módulo explicitamente NÃO FAZ
- Não expõe endpoints HTTP
- Não persiste dados diretamente
- Não executa pipelines determinísticos de engine
- Não executa validações estruturais de payload
- Não atua como camada de roteamento
- Não é um agente especializado em si

---

### Dados que este módulo pode LER
- Payloads explícitos encaminhados pelo controller
- Configurações internas necessárias para execução dos agentes

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas resultados retornados em memória para o controller

---

### Módulos dos quais este módulo DEPENDE
- networkai/agents/moderation.agent
- Tipos internos do módulo networkai

---

### Módulos dos quais este módulo NUNCA deve depender
- Camada HTTP
- Persistência direta
- engine/*
- validators
- filas, workers ou queues

---

### Este módulo pode chamar IA?
**Sim.** Atua indiretamente por meio da coordenação de agentes especializados.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui autoridade de mutação de estado dimensional.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Tornar o service um agente especializado. Persistir dados diretamente. Executar pipelines determinísticos. Expor lógica HTTP. Centralizar decisão normativa de domínio humano. Responsabilidades corretamente direcionadas: Orquestração: networkai.service. Execução especializada: agents/*. Exposição HTTP: networkai.controller. Configuração: index.ts.
**Contrato arquitetural fechado.**

## Module: networkai/agents/moderation.agent

### Responsabilidade principal
Executar lógica especializada de moderação como agente isolado, acionado pelo networkai.service.

---

### O que este módulo FAZ hoje
- Implementa um agente especializado de moderação no contexto de IA em rede
- Executa lógica encapsulada quando acionado pelo networkai.service
- Retorna resultados estruturados ao service chamador
- Atua exclusivamente como unidade executora

---

### O que este módulo explicitamente NÃO FAZ
- Não expõe endpoints HTTP
- Não decide fluxo de orquestração global
- Não persiste dados diretamente
- Não executa pipelines de engine
- Não atua como camada de roteamento
- Não executa autenticação ou autorização

---

### Dados que este módulo pode LER
- Payloads explícitos recebidos do networkai.service
- Configurações internas necessárias à execução da moderação

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas resultados retornados em memória para o service

---

### Módulos dos quais este módulo DEPENDE
- Tipos internos do módulo networkai
- (Possíveis) utilitários ou clientes de IA, se importados no código

---

### Módulos dos quais este módulo NUNCA deve depender
- Camada HTTP
- Persistência direta
- Services de domínio externos
- Engines determinísticas
- Validators de payload
- Filas, workers ou queues

---

### Este módulo pode chamar IA?
**Sim.** Pode utilizar clientes ou utilitários de IA como parte da execução especializada.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui autoridade de mutação de estado dimensional.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Tornar o agente um ponto de decisão global. Persistir dados diretamente. Expor endpoints HTTP. Executar orquestração de múltiplos agentes. Incorporar lógica de domínio humano fora do escopo de moderação. Responsabilidades corretamente direcionadas: Execução especializada: agents/moderation.agent.ts. Orquestração: networkai.service. Exposição HTTP: networkai.controller. Configuração do módulo: index.ts.
**Contrato arquitetural fechado.**

## Module: networkai/index

### Responsabilidade principal
Centralizar e expor publicamente os artefatos do módulo networkai como ponto único de integração com a aplicação.

---

### O que este módulo FAZ hoje
- Centraliza a exposição pública do módulo networkai
- Reexporta controllers, routes, services e agents internos
- Atua como ponto único de entrada do módulo para o app principal
- Agrega exports sem executar lógica

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não altera estado
- Não executa pipelines
- Não chama IA ou agentes
- Não processa eventos
- Não persiste dados
- Não define regras de negócio

---

### Dados que este módulo pode LER
- Definições estáticas de exports dos subdomínios internos

---

### Dados que este módulo pode ESCREVER
- Nenhum
- Não possui efeitos colaterais em runtime

---

### Módulos dos quais este módulo DEPENDE
- networkai/controllers/*
- networkai/routes/*
- networkai/services/*
- networkai/agents/*
- sistema de módulos TypeScript

---

### Módulos dos quais este módulo NUNCA deve depender
- infraestrutura
- persistência
- engines determinísticas
- IA direta
- validators
- filas ou workers

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como agregador passivo de exports.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui comportamento executável.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Inserir código executável. Chamar services ou agentes diretamente. Implementar lógica de inicialização. Criar dependências externas diretas. Responsabilidades corretamente direcionadas: Organização e exposição: index.ts. Orquestração: networkai.service. Execução especializada: agents/*. Exposição HTTP: networkai.controller.
**Contrato arquitetural fechado.**

## Module: networkai/README

### Responsabilidade principal
Fornecer documentação mínima e descritiva do módulo networkai, sem caráter normativo ou técnico.

---

### O que este módulo FAZ hoje
- Declara o módulo networkai como stub
- Define de forma mínima a intenção nominal do módulo
- Indica genericamente a existência de controllers, services e routes
- Atua exclusivamente como documentação passiva para leitura humana

---

### O que este módulo explicitamente NÃO FAZ
- Não define contratos arquiteturais
- Não descreve invariantes
- Não documenta fluxos, agentes ou decisões
- Não impõe regras ao código existente
- Não influencia comportamento em runtime

---

### Dados que este módulo pode LER
- Nenhum dado em runtime
- Apenas leitura humana

---

### Dados que este módulo pode ESCREVER
- Nenhum
- Não possui qualquer efeito colateral técnico

---

### Módulos dos quais este módulo DEPENDE
- Nenhum

---

### Módulos dos quais este módulo NUNCA deve depender
- Código executável
- Módulos internos
- IA
- Agentes
- Services
- Persistência
- Infraestrutura

---

### Este módulo pode chamar IA?
**Não.** Trata-se apenas de documentação estática.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui qualquer efeito técnico.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Tratar o README como especificação normativa. Alterar código para “obedecer” o README. Derivar decisões técnicas a partir deste texto. Responsabilidades corretamente direcionadas: Fonte de verdade: código. Documentação futura: README revisado a posteriori. Contrato formal: MODULES.md.
**Contrato arquitetural fechado.**


## Module: participation/participation.controller

### Responsabilidade principal
Expor endpoints HTTP relacionados à participação do usuário, atuando como camada de borda entre HTTP e o participation.service.

---

### O que este módulo FAZ hoje
- Expõe endpoint(s) HTTP relacionados à participação do usuário
- Encaminha comandos explícitos para o participation.service
- Retorna respostas HTTP baseadas no resultado do service
- Atua exclusivamente como gateway HTTP → service

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio própria
- Não altera estado diretamente
- Não executa pipelines
- Não chama IA
- Não persiste dados
- Não executa validações complexas de domínio
- Não toma decisões normativas sobre participação

---

### Dados que este módulo pode LER
- Payloads explícitos enviados via request
- Parâmetros de rota, se existentes

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido diretamente
- Apenas respostas HTTP

---

### Módulos dos quais este módulo DEPENDE
- Framework HTTP (controller)
- participation/participation.service

---

### Módulos dos quais este módulo NUNCA deve depender
- Persistência direta
- engine/*
- IA ou agentes
- validators dedicados
- filas, workers ou queues

---

### Este módulo pode chamar IA?
**Não.** Atua apenas como camada de borda HTTP.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste controller: Implementar lógica de participação diretamente. Persistir dados. Executar decisões normativas de domínio. Integrar com IA ou pipelines. Tornar-se autoridade de negócio. Responsabilidades corretamente direcionadas: Exposição HTTP: participation.controller. Orquestração e decisão: participation.service. Configuração do módulo: index.ts.
**Contrato arquitetural fechado.**

## Module: participation/participation.routes

### Responsabilidade principal
Definir e registrar as rotas HTTP do módulo participation, mapeando endpoints para o participation.controller.

---

### O que este módulo FAZ hoje
- Define o mapeamento de rotas HTTP do módulo participation
- Associa paths e métodos HTTP ao participation.controller
- Expõe a superfície HTTP para ações relacionadas à participação
- Registra rotas no Router do framework HTTP

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não altera estado
- Não executa pipelines
- Não chama IA
- Não valida payloads
- Não processa eventos
- Não persiste dados
- Não executa autenticação ou autorização explícita

---

### Dados que este módulo pode LER
- Definições estáticas de rotas (paths e métodos HTTP)
- participation.controller

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas registro das rotas no Router do framework HTTP

---

### Módulos dos quais este módulo DEPENDE
- Framework HTTP (Router)
- participation/participation.controller

---

### Módulos dos quais este módulo NUNCA deve depender
- Services de domínio
- Engines ou pipelines
- IA ou agentes
- Persistência
- Validators
- Filas, workers ou queues

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como camada declarativa de roteamento.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui qualquer autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Inserir lógica de negócio nas rotas. Executar decisões de participação. Persistir dados ou disparar pipelines. Tratar rotas como camada de decisão. Responsabilidades corretamente direcionadas: Roteamento HTTP: participation.routes.ts. Exposição de endpoints: participation.controller. Orquestração e decisão: participation.service.
**Contrato arquitetural fechado.**

## Module: participation/participation.service

### Responsabilidade principal
Centralizar a lógica operacional relacionada à participação do usuário no sistema, atuando como autoridade do domínio participation.

---

### O que este módulo FAZ hoje
- Recebe comandos explícitos encaminhados pelo participation.controller
- Executa decisões operacionais relacionadas à participação
- Processa ações como registrar, consultar ou avaliar participação
- Retorna resultados estruturados ao controller chamador

---

### O que este módulo explicitamente NÃO FAZ
- Não expõe endpoints HTTP
- Não executa pipelines determinísticos de engine
- Não chama IA ou agentes especializados
- Não executa validação estrutural de payloads
- Não atua como camada de roteamento
- Não coordena múltiplos domínios externos

---

### Dados que este módulo pode LER
- Payloads explícitos encaminhados pelo controller
- Configurações internas necessárias à lógica de participação

---

### Dados que este módulo pode ESCREVER
- **AMBÍGUO:** dados persistidos de participação, dependendo do conteúdo real do arquivo
- Resultados retornados em memória ao controller

---

### Módulos dos quais este módulo DEPENDE
- Tipos internos do módulo participation
- (Possivelmente) camada de persistência do próprio módulo

---

### Módulos dos quais este módulo NUNCA deve depender
- Camada HTTP
- engine/*
- IA ou agentes
- filas ou workers

---

### Este módulo pode chamar IA?
**Não.** A lógica de participação não envolve IA no estado atual.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui autoridade sobre estados dimensionais.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Tornar o service uma camada HTTP. Introduzir IA ou agentes. Executar pipelines determinísticos. Centralizar decisões normativas globais fora do escopo de participação. Criar acoplamento implícito com outros domínios. Responsabilidades corretamente direcionadas: Orquestração e decisão de participação: participation.service. Exposição HTTP: participation.controller. Roteamento: participation.routes.ts. Configuração do módulo: index.ts.
**Contrato arquitetural fechado.**

## Module: participation/index

### Responsabilidade principal
Centralizar e expor publicamente os artefatos do módulo participation como ponto único de integração com a aplicação.

---

### O que este módulo FAZ hoje
- Centraliza a exposição pública do módulo participation
- Reexporta controllers, routes e services internos
- Atua como ponto único de entrada do módulo para o app principal
- Agrega exports sem executar lógica

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não altera estado
- Não executa pipelines
- Não chama IA
- Não processa eventos
- Não persiste dados
- Não define regras de negócio

---

### Dados que este módulo pode LER
- Definições estáticas de exports dos subdomínios internos

---

### Dados que este módulo pode ESCREVER
- Nenhum
- Não possui efeitos colaterais em runtime

---

### Módulos dos quais este módulo DEPENDE
- participation/controllers/*
- participation/routes/*
- participation/services/*
- sistema de módulos TypeScript

---

### Módulos dos quais este módulo NUNCA deve depender
- infraestrutura
- persistência direta
- engines determinísticas
- IA ou agentes
- validators
- filas ou workers

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como agregador passivo de exports.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui comportamento executável.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Inserir código executável. Chamar services diretamente. Implementar lógica de inicialização. Criar dependências externas diretas. Responsabilidades corretamente direcionadas: Organização e exposição: index.ts. Orquestração e decisão: participation.service. Exposição HTTP: participation.controller.
**Contrato arquitetural fechado.**

## Module: participation/README

### Responsabilidade principal
Fornecer documentação mínima e descritiva do módulo participation, sem caráter normativo.

---

### O que este módulo FAZ hoje
- Declara o módulo participation como stub
- Indica de forma mínima a existência de controllers, services e routes
- Atua exclusivamente como documentação passiva para leitura humana

---

### O que este módulo explicitamente NÃO FAZ
- Não define contratos arquiteturais
- Não descreve invariantes
- Não documenta fluxos de dados ou decisões
- Não impõe regras ao código existente
- Não influencia comportamento em runtime

---

### Dados que este módulo pode LER
- Nenhum dado em runtime
- Apenas leitura humana

---

### Dados que este módulo pode ESCREVER
- Nenhum
- Não possui efeito colateral técnico

---

### Módulos dos quais este módulo DEPENDE
- Nenhum

---

### Módulos dos quais este módulo NUNCA deve depender
- Código executável
- Módulos internos
- Persistência
- IA
- Engines ou pipelines

---

### Este módulo pode chamar IA?
**Não.** Trata-se apenas de documentação estática.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui qualquer efeito técnico.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Tratar o README como especificação normativa. Alterar código para “obedecer” o README. Derivar decisões técnicas a partir deste texto. Responsabilidades corretamente direcionadas: Fonte de verdade: código. Documentação: README revisado a posteriori. Contrato formal: MODULES.md.
**Contrato arquitetural fechado.**


## Module: reputation/reputation.controller

### Responsabilidade principal
Expor endpoints HTTP relacionados ao estado de reputação, atuando como camada de borda entre HTTP e o reputation.service.

---

### O que este módulo FAZ hoje
- Expõe endpoint(s) HTTP relacionados ao estado de reputação
- Encaminha comandos explícitos para o reputation.service
- Retorna respostas HTTP baseadas no resultado do service
- Atua exclusivamente como gateway HTTP → service

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio própria
- Não altera estado diretamente
- Não executa pipelines
- Não chama IA
- Não persiste dados
- Não executa validações complexas de domínio
- Não define regras normativas de reputação

---

### Dados que este módulo pode LER
- Payloads explícitos enviados via request
- Parâmetros de rota, se existentes

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido diretamente
- Apenas respostas HTTP

---

### Módulos dos quais este módulo DEPENDE
- Framework HTTP (controller)
- reputation/reputation.service

---

### Módulos dos quais este módulo NUNCA deve depender
- Persistência direta
- engine/*
- IA ou agentes
- validators dedicados
- filas, workers ou queues

---

### Este módulo pode chamar IA?
**Não.** Atua apenas como camada de borda HTTP.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste controller: Implementar lógica de reputação diretamente. Persistir dados. Executar decisões normativas globais. Integrar com IA ou pipelines. Tornar-se autoridade de domínio. Responsabilidades corretamente direcionadas: Exposição HTTP: reputation.controller. Orquestração e decisão: reputation.service. Configuração do módulo: index.ts.
**Contrato arquitetural fechado.**

## Module: reputation/reputation.routes

### Responsabilidade principal
Definir e registrar as rotas HTTP do módulo reputation, mapeando endpoints para o reputation.controller.

---

### O que este módulo FAZ hoje
- Define o mapeamento de rotas HTTP do módulo reputation
- Associa paths e métodos HTTP ao reputation.controller
- Expõe a superfície HTTP para leitura e/ou ações relacionadas à reputação
- Registra rotas no Router do framework HTTP

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não altera estado
- Não executa pipelines
- Não chama IA
- Não valida payloads
- Não processa eventos
- Não persiste dados
- Não executa autenticação ou autorização explícita

---

### Dados que este módulo pode LER
- Definições estáticas de rotas (paths e métodos HTTP)
- reputation.controller

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas registro das rotas no Router do framework HTTP

---

### Módulos dos quais este módulo DEPENDE
- Framework HTTP (Router)
- reputation/reputation.controller

---

### Módulos dos quais este módulo NUNCA deve depender
- Services de domínio
- Engines ou pipelines
- IA ou agentes
- Persistência
- Validators
- Filas, workers ou queues

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como camada declarativa de roteamento.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui qualquer autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Inserir lógica de negócio nas rotas. Executar decisões de reputação. Persistir dados ou disparar pipelines. Tratar rotas como camada de decisão. Responsabilidades corretamente direcionadas: Roteamento HTTP: reputation.routes.ts. Exposição de endpoints: reputation.controller. Orquestração e decisão: reputation.service.
**Contrato arquitetural fechado.**

## Module: reputation/reputation.service

### Responsabilidade principal
Centralizar a lógica operacional de reputação do sistema, atuando como autoridade do domínio reputation.

---

### O que este módulo FAZ hoje
- Recebe comandos explícitos encaminhados pelo reputation.controller
- Executa operações de consulta, cálculo ou atualização de reputação
- Processa regras operacionais internas do domínio de reputação
- Retorna resultados estruturados ao controller chamador

---

### O que este módulo explicitamente NÃO FAZ
- Não expõe endpoints HTTP
- Não executa pipelines determinísticos de engine
- Não chama IA ou agentes especializados
- Não executa validação estrutural de payloads
- Não atua como camada de roteamento
- Não coordena múltiplos domínios externos

---

### Dados que este módulo pode LER
- Payloads explícitos encaminhados pelo controller
- Configurações internas necessárias à lógica de reputação
- **AMBÍGUO:** estado persistido de reputação, dependendo do código existente

---

### Dados que este módulo pode ESCREVER
- **AMBÍGUO:** escrita/persistência de estado de reputação, dependendo do código existente
- Resultados retornados em memória ao controller

---

### Módulos dos quais este módulo DEPENDE
- Tipos internos do módulo reputation
- (Possivelmente) camada de persistência do próprio módulo

---

### Módulos dos quais este módulo NUNCA deve depender
- Camada HTTP
- engine/*
- IA ou agentes
- validators dedicados
- filas ou workers

---

### Este módulo pode chamar IA?
**Não.** A lógica de reputação não envolve IA no estado atual.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui autoridade sobre estados dimensionais.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Tornar o service uma camada HTTP. Introduzir IA ou agentes especializados. Executar pipelines determinísticos. Centralizar decisões normativas globais fora do escopo de reputação. Criar acoplamento implícito com outros domínios. Responsabilidades corretamente direcionadas: Orquestração e decisão de reputação: reputation.service. Exposição HTTP: reputation.controller. Roteamento: reputation.routes.ts. Configuração do módulo: index.ts.
**Contrato arquitetural fechado.**

## Module: reputation/index

### Responsabilidade principal
Centralizar e expor publicamente os artefatos do módulo reputation como ponto único de integração com a aplicação.

---

### O que este módulo FAZ hoje
- Centraliza a exposição pública do módulo reputation
- Reexporta controllers, routes e services internos
- Atua como ponto único de entrada do módulo para o app principal
- Agrega exports sem executar lógica

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não altera estado
- Não executa pipelines
- Não chama IA
- Não processa eventos
- Não persiste dados
- Não define regras de negócio

---

### Dados que este módulo pode LER
- Definições estáticas de exports dos subdomínios internos

---

### Dados que este módulo pode ESCREVER
- Nenhum
- Não possui efeitos colaterais em runtime

---

### Módulos dos quais este módulo DEPENDE
- reputation/controllers/*
- reputation/routes/*
- reputation/services/*
- sistema de módulos TypeScript

---

### Módulos dos quais este módulo NUNCA deve depender
- infraestrutura
- persistência direta
- engines determinísticas
- IA ou agentes
- validators
- filas ou workers

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como agregador passivo de exports.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui comportamento executável.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Inserir código executável. Chamar services diretamente. Implementar lógica de inicialização. Criar dependências externas diretas. Responsabilidades corretamente direcionadas: Organização e exposição: index.ts. Orquestração e decisão: reputation.service. Exposição HTTP: reputation.controller.
**Contrato arquitetural fechado.**

## Module: reputation/README

### Responsabilidade principal
Fornecer documentação mínima e descritiva do módulo reputation, sem caráter normativo.

---

### O que este módulo FAZ hoje
- Declara o módulo reputation como stub
- Indica de forma mínima a existência de controllers, services e routes
- Atua exclusivamente como documentação passiva para leitura humana

---

### O que este módulo explicitamente NÃO FAZ
- Não define contratos arquiteturais
- Não descreve invariantes
- Não documenta fluxos, regras ou decisões
- Não impõe regras ao código existente
- Não influencia comportamento em runtime

---

### Dados que este módulo pode LER
- Nenhum dado em runtime
- Apenas leitura humana

---

### Dados que este módulo pode ESCREVER
- Nenhum
- Não possui efeito colateral técnico

---

### Módulos dos quais este módulo DEPENDE
- Nenhum

---

### Módulos dos quais este módulo NUNCA deve depender
- Código executável
- Módulos internos
- Persistência
- IA
- Engines ou pipelines

---

### Este módulo pode chamar IA?
**Não.** Trata-se apenas de documentação estática.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui qualquer efeito técnico.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Tratar o README como especificação normativa. Alterar código para “obedecer” o README. Derivar decisões técnicas a partir deste texto. Responsabilidades corretamente direcionadas: Fonte de verdade: código. Documentação futura: README revisado a posteriori. Contrato formal: MODULES.md.
**Contrato arquitetural fechado.**


## Module: tokenengine/tokenengine.controller

### Responsabilidade principal
Expor endpoints HTTP para operações do motor de tokens, atuando como camada de borda entre HTTP e o tokenengine.service.

---

### O que este módulo FAZ hoje
- Expõe endpoint(s) HTTP relacionados às operações do motor de tokens
- Encaminha comandos explícitos para o tokenengine.service
- Retorna respostas HTTP baseadas no resultado do service
- Atua exclusivamente como gateway HTTP → service

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de negócio própria
- Não altera estado diretamente
- Não executa pipelines
- Não chama IA
- Não interage diretamente com contratos on-chain
- Não persiste dados
- Não executa validações complexas de domínio

---

### Dados que este módulo pode LER
- Payloads explícitos enviados via request
- Parâmetros de rota, se existentes

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido diretamente
- Apenas respostas HTTP

---

### Módulos dos quais este módulo DEPENDE
- Framework HTTP (controller)
- tokenengine/tokenengine.service

---

### Módulos dos quais este módulo NUNCA deve depender
- Persistência direta
- engine/*
- IA ou agentes
- validators dedicados
- filas, workers ou queues
- clientes blockchain diretos

---

### Este módulo pode chamar IA?
**Não.** Atua apenas como camada de borda HTTP.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste controller: Executar lógica de tokenomics diretamente. Interagir diretamente com contratos blockchain. Persistir dados. Tornar-se autoridade de domínio econômico. Integrar com IA ou pipelines. Responsabilidades corretamente direcionadas: Exposição HTTP: tokenengine.controller. Orquestração e decisão: tokenengine.service. Execução on-chain/off-chain: contracts/*. Configuração do módulo: index.ts.
**Contrato arquitetural fechado.**

## Module: tokenengine/tokenengine.routes

### Responsabilidade principal
Definir e registrar as rotas HTTP do módulo tokenengine, mapeando endpoints para o tokenengine.controller.

---

### O que este módulo FAZ hoje
- Define o mapeamento de rotas HTTP do módulo tokenengine
- Associa paths e métodos HTTP ao tokenengine.controller
- Expõe a superfície HTTP para operações relacionadas ao motor de tokens
- Registra rotas no Router do framework HTTP

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não altera estado
- Não executa pipelines
- Não interage com contratos on-chain
- Não chama IA
- Não valida payloads
- Não persiste dados
- Não executa autenticação ou autorização explícita

---

### Dados que este módulo pode LER
- Definições estáticas de rotas (paths e métodos HTTP)
- tokenengine.controller

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas registro das rotas no Router do framework HTTP

---

### Módulos dos quais este módulo DEPENDE
- Framework HTTP (Router)
- tokenengine/tokenengine.controller

---

### Módulos dos quais este módulo NUNCA deve depender
- Services de domínio
- Engines determinísticas
- Contratos blockchain
- IA ou agentes
- Persistência
- Validators
- Filas, workers ou queues

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como camada declarativa de roteamento.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui qualquer autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Inserir lógica de negócio nas rotas. Executar decisões econômicas. Interagir com contratos on-chain. Persistir dados ou disparar pipelines. Tratar rotas como camada de decisão. Responsabilidades corretamente direcionadas: Roteamento HTTP: tokenengine.routes.ts. Exposição de endpoints: tokenengine.controller. Orquestração e decisão: tokenengine.service. Execução on-chain/off-chain: contracts/*.
**Contrato arquitetural fechado.**

## Module: tokenengine/tokenengine.service

### Responsabilidade principal
Orquestrar operacionalmente o motor de tokens, coordenando decisões e fluxos lógicos relacionados a tokens sem execução on-chain direta.

---

### O que este módulo FAZ hoje
- Recebe comandos explícitos encaminhados pelo tokenengine.controller
- Centraliza a orquestração operacional do motor de tokens
- Coordena operações de emissão, leitura ou atualização lógica de tokens
- Retorna resultados estruturados ao controller chamador

---

### O que este módulo explicitamente NÃO FAZ
- Não expõe endpoints HTTP
- Não executa contratos blockchain diretamente
- Não executa pipelines determinísticos de engine
- Não chama IA
- Não executa validação estrutural de payloads
- Não atua como camada de roteamento
- Não coordena múltiplos domínios fora do escopo de tokens

---

### Dados que este módulo pode LER
- Payloads explícitos encaminhados pelo controller
- Configurações internas relacionadas ao motor de tokens
- **AMBÍGUO:** estado persistido de tokens, dependendo do conteúdo do arquivo

---

### Dados que este módulo pode ESCREVER
- **AMBÍGUO:** escrita ou persistência de estado lógico de tokens, dependendo do código existente
- Resultados retornados em memória ao controller

---

### Módulos dos quais este módulo DEPENDE
- Tipos internos do módulo tokenengine
- (Possivelmente) dependências off-chain ou de acesso indireto a contratos, se existirem

---

### Módulos dos quais este módulo NUNCA deve depender
- Camada HTTP
- engine/*
- IA ou agentes
- validators dedicados
- filas ou workers

---

### Este módulo pode chamar IA?
**Não.** O motor de tokens não envolve IA no estado atual.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui autoridade sobre estados dimensionais.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Tornar o service um cliente blockchain direto. Introduzir IA ou agentes especializados. Executar pipelines determinísticos. Centralizar decisões normativas globais de economia. Criar acoplamento implícito com outros domínios. Responsabilidades corretamente direcionadas: Orquestração e decisão: tokenengine.service. Exposição HTTP: tokenengine.controller. Execução on-chain/off-chain: contracts/*. Configuração do módulo: index.ts.
**Contrato arquitetural fechado.**

## Module: tokenengine/contracts/<arquivo_vazio>

### Responsabilidade principal
Servir como placeholder estrutural para futuras definições de contratos relacionados ao motor de tokens.

---

### O que este módulo FAZ hoje
- Nada
- O arquivo existe fisicamente, mas não contém implementação
- Não é executado em runtime
- Não define contratos, ABIs ou wrappers

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de tokenomics
- Não interage com blockchain
- Não expõe funções
- Não persiste dados
- Não é invocado pelo tokenengine.service
- Não produz efeitos colaterais

---

### Dados que este módulo pode LER
- Nenhum

---

### Dados que este módulo pode ESCREVER
- Nenhum

---

### Módulos dos quais este módulo DEPENDE
- Nenhum

---

### Módulos dos quais este módulo NUNCA deve depender
- Clientes blockchain
- SDKs Web3
- Services
- Engines
- IA
- Infraestrutura

---

### Este módulo pode chamar IA?
**Não.** Arquivo vazio sem execução.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui qualquer comportamento técnico.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural futura deste subdomínio: Inserir lógica de orquestração neste arquivo. Executar chamadas HTTP ou services. Misturar decisão econômica com definição de contrato. Transformar este arquivo em camada ativa sem revisão contratual. Responsabilidades corretamente direcionadas: Definição de contratos: contracts/*. Orquestração: tokenengine.service. Exposição HTTP: tokenengine.controller.
**Contrato arquitetural fechado.**

## Module: tokenengine/index

### Responsabilidade principal
Centralizar a exposição pública do módulo tokenengine como ponto único de integração com a aplicação.

---

### O que este módulo FAZ hoje
- Centraliza a exposição pública do módulo tokenengine
- Reexporta controllers, routes, services e contracts internos
- Atua como ponto único de entrada do módulo para o app principal
- Agrega exports sem executar lógica

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não altera estado
- Não executa pipelines
- Não chama IA
- Não interage com blockchain
- Não processa eventos
- Não persiste dados
- Não define regras econômicas

---

### Dados que este módulo pode LER
- Definições estáticas de exports dos subdomínios internos

---

### Dados que este módulo pode ESCREVER
- Nenhum
- Não possui efeitos colaterais em runtime

---

### Módulos dos quais este módulo DEPENDE
- tokenengine/controllers/*
- tokenengine/routes/*
- tokenengine/services/*
- tokenengine/contracts/*
- sistema de módulos TypeScript

---

### Módulos dos quais este módulo NUNCA deve depender
- infraestrutura
- persistência direta
- engines determinísticas
- IA ou agentes
- validators
- filas ou workers

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como agregador passivo de exports.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui comportamento executável.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Inserir código executável. Chamar services ou contracts diretamente. Implementar lógica de inicialização. Criar dependências externas diretas. Responsabilidades corretamente direcionadas: Organização e exposição: index.ts. Orquestração e decisão: tokenengine.service. Execução on-chain/off-chain: contracts/*. Exposição HTTP: tokenengine.controller.
**Contrato arquitetural fechado.**

## Module: tokenengine/README

### Responsabilidade principal
Fornecer documentação mínima e descritiva do módulo tokenengine, sem caráter normativo.

---

### O que este módulo FAZ hoje
- Declara o módulo tokenengine como stub
- Indica de forma mínima a existência esperada de controllers, services e routes
- Atua exclusivamente como documentação passiva para leitura humana

---

### O que este módulo explicitamente NÃO FAZ
- Não define contratos arquiteturais
- Não descreve invariantes
- Não documenta fluxos econômicos, tokenomics ou regras
- Não impõe regras ao código existente
- Não influencia comportamento em runtime

---

### Dados que este módulo pode LER
- Nenhum dado em runtime
- Apenas leitura humana

---

### Dados que este módulo pode ESCREVER
- Nenhum
- Não possui efeito colateral técnico

---

### Módulos dos quais este módulo DEPENDE
- Nenhum

---

### Módulos dos quais este módulo NUNCA deve depender
- Código executável
- Módulos internos
- Persistência
- Blockchain
- IA
- Engines ou pipelines

---

### Este módulo pode chamar IA?
**Não.** Trata-se apenas de documentação estática.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui qualquer efeito técnico.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Tratar o README como especificação normativa. Alterar código para “obedecer” o README. Derivar decisões técnicas a partir deste texto. Responsabilidades corretamente direcionadas: Fonte de verdade: código. Documentação futura: README revisado a posteriori. Contrato formal: MODULES.md.
**Contrato arquitetural fechado.**


## Module: userai/userai.controller

### Responsabilidade principal
Expor endpoints HTTP para interação de IA associada ao usuário, atuando como camada de borda entre HTTP e o userai.service.

---

### O que este módulo FAZ hoje
- Expõe endpoint(s) HTTP relacionados à interação de IA do usuário
- Encaminha comandos explícitos para o userai.service
- Retorna respostas HTTP baseadas no resultado do service
- Atua exclusivamente como gateway HTTP → service

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de IA diretamente
- Não toma decisões autônomas de domínio
- Não altera estado persistente
- Não executa pipelines determinísticos
- Não coordena múltiplos serviços
- Não executa validações complexas de domínio

---

### Dados que este módulo pode LER
- Payloads explícitos enviados via request
- Parâmetros de rota, se existentes
- Contexto mínimo do usuário, se incluído no request

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido diretamente
- Apenas respostas HTTP

---

### Módulos dos quais este módulo DEPENDE
- Framework HTTP (controller)
- userai/userai.service

---

### Módulos dos quais este módulo NUNCA deve depender
- Persistência direta
- engine/*
- IA direta (clientes LLM)
- validators dedicados
- filas, workers ou queues

---

### Este módulo pode chamar IA?
**Não.** Não executa chamadas diretas de IA; apenas delega ao service.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste controller: Executar chamadas diretas a LLMs. Implementar lógica de decisão de IA. Persistir dados. Tornar-se autoridade de domínio humano. Integrar pipelines ou agentes. Responsabilidades corretamente direcionadas: Exposição HTTP: userai.controller. Orquestração e decisão: userai.service. Execução de IA: fora do controller (service ou agentes, se existirem).
**Contrato arquitetural fechado.**

## Module: userai/userai.routes

### Responsabilidade principal
Definir e registrar as rotas HTTP do módulo userai, mapeando endpoints para o userai.controller.

---

### O que este módulo FAZ hoje
- Define o mapeamento de rotas HTTP do módulo userai
- Associa paths e métodos HTTP ao userai.controller
- Expõe a superfície HTTP para interações de IA vinculadas ao usuário
- Registra rotas no Router do framework HTTP

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de IA
- Não altera estado
- Não executa pipelines
- Não chama diretamente LLMs
- Não valida payloads
- Não persiste dados
- Não executa autenticação ou autorização explícita

---

### Dados que este módulo pode LER
- Definições estáticas de rotas (paths e métodos HTTP)
- userai.controller

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Apenas registro das rotas no Router do framework HTTP

---

### Módulos dos quais este módulo DEPENDE
- Framework HTTP (Router)
- userai/userai.controller

---

### Módulos dos quais este módulo NUNCA deve depender
- Services de domínio
- Engines determinísticas
- IA direta (clientes LLM)
- Persistência
- Validators
- Filas, workers ou queues

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como camada declarativa de roteamento.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui qualquer autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Inserir lógica de IA nas rotas. Executar decisões de domínio. Persistir dados ou disparar pipelines. Tratar rotas como camada de decisão. Responsabilidades corretamente direcionadas: Roteamento HTTP: userai.routes.ts. Exposição de endpoints: userai.controller. Orquestração e decisão: userai.service.
**Contrato arquitetural fechado.**

## Module: userai/userai.service

### Responsabilidade principal
Orquestrar interações de IA associadas ao usuário, coordenando chamadas controladas a capacidades de IA sem autoridade de estado humano.

---

### O que este módulo FAZ hoje
- Recebe comandos explícitos encaminhados pelo userai.controller
- Centraliza a orquestração das interações de IA do usuário
- Coordena chamadas a capacidades de IA em escopo controlado
- Retorna resultados estruturados ao controller chamador
- Atua como camada operacional auxiliar, não decisória

---

### O que este módulo explicitamente NÃO FAZ
- Não expõe endpoints HTTP
- Não executa pipelines determinísticos de engine
- Não persiste estados de domínio humano
- Não toma decisões normativas finais
- Não executa validação estrutural profunda de payloads
- Não atua como camada de roteamento

---

### Dados que este módulo pode LER
- Payloads explícitos encaminhados pelo controller
- Contexto mínimo do usuário necessário à chamada de IA
- Configurações internas de execução de IA

---

### Dados que este módulo pode ESCREVER
- Nenhum dado persistido
- Resultados retornados em memória ao controller
- Efeitos colaterais externos não persistidos (ex.: chamadas a LLMs)

---

### Módulos dos quais este módulo DEPENDE
- Tipos internos do módulo userai
- Clientes/SDKs de IA (se importados no código)
- Configurações de ambiente para execução de IA

---

### Módulos dos quais este módulo NUNCA deve depender
- Camada HTTP
- Persistência direta
- engine/*
- Pipelines de domínio (IND / EXD / U)
- validators dedicados
- filas ou workers explícitos

---

### Este módulo pode chamar IA?
**Sim.** Atua como orquestrador de chamadas controladas a capacidades de IA.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui autoridade sobre estados dimensionais humanos.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Persistir resultados de IA como estado humano. Tornar a IA autoridade decisória final. Executar pipelines determinísticos de domínio. Acoplar diretamente com outros módulos de estado (ex.: userdimensions). Introduzir lógica HTTP ou roteamento. Responsabilidades corretamente direcionadas: Orquestração de IA: userai.service. Exposição HTTP: userai.controller. Decisão de estado humano: outros módulos (ex.: userdimensions). Persistência: módulos específicos de estado.
**Contrato arquitetural fechado.**

## Module: userai/index

### Responsabilidade principal
Centralizar e expor publicamente os artefatos do módulo userai como ponto único de integração com a aplicação.

---

### O que este módulo FAZ hoje
- Centraliza a exposição pública do módulo userai
- Reexporta controllers, routes e services internos
- Atua como ponto único de entrada do módulo para o app principal
- Agrega exports sem executar lógica

---

### O que este módulo explicitamente NÃO FAZ
- Não executa lógica de domínio
- Não altera estado
- Não executa pipelines
- Não chama IA diretamente
- Não processa eventos
- Não persiste dados
- Não define regras de negócio

---

### Dados que este módulo pode LER
- Definições estáticas de exports dos subdomínios internos

---

### Dados que este módulo pode ESCREVER
- Nenhum
- Não possui efeitos colaterais em runtime

---

### Módulos dos quais este módulo DEPENDE
- userai/controllers/*
- userai/routes/*
- userai/services/*
- sistema de módulos TypeScript

---

### Módulos dos quais este módulo NUNCA deve depender
- infraestrutura
- persistência direta
- engines determinísticas
- pipelines de domínio
- validators
- filas ou workers

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como agregador passivo de exports.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui comportamento executável.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Inserir código executável. Chamar services diretamente. Implementar lógica de inicialização. Criar dependências externas diretas. Responsabilidades corretamente direcionadas: Organização e exposição: index.ts. Orquestração de IA: userai.service. Exposição HTTP: userai.controller.
**Contrato arquitetural fechado.**

## Module: userai/README

### Responsabilidade principal
Fornecer documentação mínima e descritiva do módulo userai, sem caráter normativo.

---

### O que este módulo FAZ hoje
- Declara o módulo userai como stub
- Indica de forma mínima a existência de controllers, services e routes
- Atua exclusivamente como documentação passiva para leitura humana

---

### O que este módulo explicitamente NÃO FAZ
- Não define contratos arquiteturais
- Não descreve invariantes
- Não documenta fluxos de IA, limites ou decisões
- Não impõe regras ao código existente
- Não influencia comportamento em runtime

---

### Dados que este módulo pode LER
- Nenhum dado em runtime
- Apenas leitura humana

---

### Dados que este módulo pode ESCREVER
- Nenhum
- Não possui efeito colateral técnico

---

### Módulos dos quais este módulo DEPENDE
- Nenhum

---

### Módulos dos quais este módulo NUNCA deve depender
- Código executável
- Módulos internos
- Persistência
- LLMs
- Engines ou pipelines

---

### Este módulo pode chamar IA?
**Não.** Trata-se apenas de documentação estática.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui qualquer efeito técnico.

---

### Estado do módulo
Alerta de quebra de contrato Caracteriza violação arquitetural deste subdomínio: Tratar o README como especificação normativa. Alterar código para “obedecer” o README. Derivar decisões técnicas a partir deste texto. Responsabilidades corretamente direcionadas: Fonte de verdade: código. Documentação futura: README revisado a posteriori. Contrato formal: MODULES.md.
**Contrato arquitetural fechado.**


## Module: web3/web3.controller

### Responsabilidade principal
Expor endpoints HTTP básicos relacionados a operações Web3, atuando como camada de borda entre HTTP e o Web3Service.

---

### O que este módulo FAZ hoje
- Define uma classe Web3Controller com handlers HTTP estáticos
- Expõe um endpoint de health check que retorna `{ module: "web3", ok: true }`
- Expõe um endpoint de leitura de saldo (`getBalance`)
  - Lê `address` via `req.params`
  - Valida apenas a presença do parâmetro
  - Delegada a obtenção do saldo para `Web3Service.getBalance(address)`
  - Retorna o resultado diretamente
- Expõe um endpoint de criação de carteira temporária (`createTempWallet`)
  - Delegada integralmente a criação para `Web3Service.createTemporaryWallet()`
  - Retorna o objeto resultante diretamente na resposta HTTP
- Captura exceções genéricas e retorna erro HTTP 500

---

### O que este módulo explicitamente NÃO FAZ
- Não persiste dados
- Não valida formato, checksum ou rede do `address`
- Não autentica nem autoriza usuários
- Não verifica ownership de carteira
- Não assina transações
- Não interage diretamente com blockchain, RPCs ou providers
- Não executa lógica de negócio Web3
- Não define contratos de segurança
- Não controla rate-limit, replay ou abuso
- Não transforma nem normaliza o payload de saída

---

### Dados que este módulo pode LER
- `req.params.address`
- Nenhum outro dado de request (body, headers, contexto de autenticação)

---

### Dados que este módulo pode ESCREVER
- Resposta HTTP (`res.json`, `res.status`)
- Logs de erro via `console.error`

---

### Módulos dos quais este módulo DEPENDE
- `web3/web3.service`
- Framework HTTP (req/res — Express ou compatível)

---

### Módulos dos quais este módulo NUNCA deve depender
- Banco de dados
- SDKs Web3 (ethers, web3.js, viem, etc.)
- Providers RPC
- Camadas de autenticação/autorização
- Configuração de rede ou `chainId`
- Cache
- Guards ou middlewares de segurança

---

### Este módulo pode chamar IA?
**Não.** Não há qualquer integração ou dependência de IA.

---

### Este módulo pode alterar IND ou U?
**Não.** Não interage com estados dimensionais ou domínio humano.

---

### Estado do módulo
Alerta de quebra de contrato O arquivo contém ... literal, caracterizando código incompleto. Métodos, fluxos ou tratamentos omitidos são AMBÍGUOS e não analisáveis. Caso o Web3Service execute escrita de estado, transações ou efeitos colaterais, o controller não impõe nenhuma barreira técnica para evitar uso indevido. Exposição de createTempWallet sem qualquer proteção pode violar invariantes de segurança em camadas superiores.
**Contrato arquitetural fechado.**

## Module: web3/web3.routes

### Responsabilidade principal
Definir e registrar as rotas HTTP públicas do módulo web3, mapeando endpoints para o Web3Controller.

---

### O que este módulo FAZ hoje
- Instancia um Router do Express
- Define rotas HTTP públicas do módulo web3
- Mapeia `GET /` para `Web3Controller.health`
- Mapeia `GET /balance/:address` para `Web3Controller.getBalance`
- Mapeia `GET /block` para `Web3Controller.getBlock`
- Mapeia `GET /wallet/temp` para `Web3Controller.createTempWallet`
- Exporta o router como default

---

### O que este módulo explicitamente NÃO FAZ
- Não implementa lógica de negócio
- Não valida parâmetros de rota
- Não aplica autenticação ou autorização
- Não define middlewares de segurança
- Não define versionamento de API
- Não controla escopo de ambiente (dev vs prod)
- Não aplica rate-limit, throttle ou proteção contra abuso
- Não trata erros
- Não registra logs

---

### Dados que este módulo pode LER
- Parâmetros de rota definidos na URL (`:address`)
- Nenhum dado de body, headers ou contexto de autenticação

---

### Dados que este módulo pode ESCREVER
- Nenhum dado diretamente
- Apenas delega execução para o controller

---

### Módulos dos quais este módulo DEPENDE
- `express` (Router)
- `web3/web3.controller`

---

### Módulos dos quais este módulo NUNCA deve depender
- Banco de dados
- SDKs Web3
- Providers RPC
- Middlewares globais ou locais
- Guards de autenticação/autorização
- Configuração de ambiente
- Camadas de cache

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como camada declarativa de roteamento.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui qualquer autoridade de mutação de estado.

---

### Estado do módulo
Alerta de quebra de contrato A rota /wallet/temp é explicitamente comentada como “dev tool”, porém: Não há qualquer restrição técnica que impeça uso em produção. A camada de rotas não diferencia ambientes. A rota /block está exposta, mas: O contrato de entrada/saída é AMBÍGUO. Não há parâmetros explícitos. Caso o controller execute chamadas RPC ou gere chaves, a exposição direta via GET viola princípios mínimos de segurança.
**Contrato arquitetural fechado.**

## Module: web3/web3.service

### Responsabilidade principal
Integrar o sistema a um provider JSON-RPC Web3, fornecendo operações utilitárias de leitura on-chain e geração de carteiras temporárias.

---

### O que este módulo FAZ hoje
- Define a classe `Web3Service`
- Inicializa um `ethers.JsonRpcProvider` com URL RPC definida inline
- Implementa métodos estáticos para:
  - Obter saldo de uma carteira (`getBalance`)
  - Obter informações do último bloco (`getBlock`)
  - Criar uma carteira temporária (`createTemporaryWallet`)
- Converte saldo de `BigInt` para string formatada via `ethers.formatEther`
- Gera par de chaves (`address` + `privateKey`) via `ethers.Wallet.createRandom()`
- Retorna objetos JavaScript simples como resultado direto dos métodos

---

### O que este módulo explicitamente NÃO FAZ
- Não persiste dados de carteira, chave ou histórico
- Não protege, criptografa ou mascara `privateKey`
- Não valida endereço de entrada (`address`)
- Não verifica rede, `chainId` ou ambiente
- Não executa transações
- Não assina mensagens ou contratos
- Não gerencia sessões
- Não aplica controle de acesso
- Não registra auditoria
- Não diferencia ambiente (dev / prod)
- Não aplica rate-limit ou throttling
- Não utiliza contratos inteligentes

---

### Dados que este módulo pode LER
- URL RPC definida inline no código
- Endereço de carteira recebido como argumento (`address`)
- Estado on-chain público via RPC (saldo, bloco)

---

### Dados que este módulo pode ESCREVER
- Objetos retornados em memória
- Logs de erro via `console.error`

---

### Módulos dos quais este módulo DEPENDE
- Biblioteca `ethers`
- Endpoint JSON-RPC público (hardcoded)
- Runtime Node.js

---

### Módulos dos quais este módulo NUNCA deve depender
- Banco de dados
- Vault / KMS
- Configuração por variáveis de ambiente
- SDKs de contratos inteligentes
- Cache
- Middlewares de segurança
- Políticas de ambiente
- Camadas de abstração de provider

---

### Este módulo pode chamar IA?
**Não.** Atua exclusivamente como integração Web3 via RPC.

---

### Este módulo pode alterar IND ou U?
**Não.** Não interage com estados dimensionais ou domínio humano.

---

### Estado do módulo
Alerta de quebra de contrato createTemporaryWallet retorna privateKey em texto plano: Quebra invariantes mínimos de segurança se exposto fora de ambiente controlado. URL RPC hardcoded: Viola isolamento de ambiente. Impossibilita controle operacional. Serviço pode ser abusado como wallet generator público sem qualquer barreira. Qualquer chamada a este service em produção cria risco crítico de vazamento de chaves.
**Contrato arquitetural fechado.**

## Module: web3/index

### Responsabilidade principal
Atuar como ponto de entrada público do módulo web3, reexportando o router HTTP do módulo.

---

### O que este módulo FAZ hoje
- Importa o Router do módulo web3 a partir de `routes/web3.routes`
- Reexporta o router como exportação padrão do módulo
- Expõe o módulo web3 para integração com o restante da aplicação

---

### O que este módulo explicitamente NÃO FAZ
- Não define rotas
- Não implementa lógica de negócio
- Não executa validações
- Não aplica middlewares
- Não gerencia dependências
- Não altera estado
- Não configura ambiente ou provider
- Não interage com blockchain ou RPC

---

### Dados que este módulo pode LER
- Nenhum dado em runtime
- Apenas imports estáticos

---

### Dados que este módulo pode ESCREVER
- Nenhum

---

### Módulos dos quais este módulo DEPENDE
- web3/routes/web3.routes

---

### Módulos dos quais este módulo NUNCA deve depender
- Controllers
- Services
- SDKs Web3
- Banco de dados
- Configuração de ambiente
- Middlewares globais

---

### Este módulo pode chamar IA?
**Não.** Não possui qualquer comportamento executável.

---

### Este módulo pode alterar IND ou U?
**Não.** Não interage com estados dimensionais.

---

### Estado do módulo
Alerta de quebra de contrato Nenhum no escopo deste arquivo isoladamente. Qualquer quebra decorre do comportamento do router/controller exposto.
**Contrato arquitetural fechado.**

## Module: web3/README

### Responsabilidade principal
Fornecer documentação mínima e descritiva do módulo web3, sem caráter normativo ou efeito técnico.

---

### O que este módulo FAZ hoje
- Declara a existência do módulo web3
- Indica genericamente a presença esperada de controllers, services e routes
- Atua exclusivamente como documentação passiva para leitura humana

---

### O que este módulo explicitamente NÃO FAZ
- Não define contratos arquiteturais
- Não define invariantes
- Não descreve fluxos Web3
- Não impõe restrições de segurança
- Não diferencia ambiente (dev / prod)
- Não define responsabilidades entre camadas
- Não influencia comportamento em runtime
- Não é fonte de verdade arquitetural

---

### Dados que este módulo pode LER
- Nenhum dado em runtime
- Apenas leitura humana

---

### Dados que este módulo pode ESCREVER
- Nenhum
- Não possui efeito colateral técnico

---

### Módulos dos quais este módulo DEPENDE
- Nenhum

---

### Módulos dos quais este módulo NUNCA deve depender
- Código executável
- Imports de módulos internos
- Blockchain, RPCs, providers ou contratos
- Segurança, chaves ou transações

---

### Este módulo pode chamar IA?
**Não.** Trata-se apenas de documentação estática.

---

### Este módulo pode alterar IND ou U?
**Não.** Não possui qualquer efeito técnico.

---

### Estado do módulo
Alerta de quebra de contrato O README não alerta sobre: Exposição de geração de wallets. Uso de RPC hardcoded. Retorno de privateKey em texto plano. Pode induzir falsa percepção de neutralidade ou segurança do módulo.
**Contrato arquitetural fechado.**

