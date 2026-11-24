# AttendaHub — contexto do projeto

## Tecnologias e estrutura

AttendaHub usa n8n, PostgreSQL e WuzAPI para automação de atendimento por WhatsApp. O compose.yml declara n8n, PostgreSQL (pgvector/PostgreSQL 17), Redis e WuzAPI. Existem também pastas Chatwoot/Evolution, mas pastas não comprovam serviços ativos. Os dados persistentes locais ficam nos volumes montados pelo Compose.

Workflows JSON contém quatro exports:

- AttendaHub.json: webhook, filtragem, identificação cliente/funcionário, expediente, atendimento e gravação de fila.
- FilaAndou.json: polling da fila, envio WuzAPI, recepção por Google Gemini e encaminhamento.
- Fila Aguardando Funcionario.json: atribuição periódica de funcionário e re-enfileiramento.
- Instalçao.json: schema SQL, seed e manutenção manual; o caminho acionado apaga e recria tabelas.

Há quatro workflows existentes conhecidos. Um quinto workflow, `AttendaHub - Testes Automatizados`, foi decidido, mas ainda não foi criado. Ele deverá concentrar a preparação e a execução do ambiente permanente de testes `attendahub_test`; não confunda essa decisão com uma implementação concluída.

compose.yml usa n8nio/n8n:latest, que não fixa versão. O n8n persiste em n8n_data; PostgreSQL e Redis têm volumes próprios. Confirme a versão do container em execução antes de descrever compatibilidade ou atualizar.

## Arquitetura conhecida

Os workflows exportados não chamam uns aos outros. Compartilham dados via PostgreSQL: atendimento, funcionario, setor, fila_mensagens, aguardando_funcionario e feriado; o agente também declara memória Postgres atendahub_ai_memory.

Fluxo pretendido: POST no webhook n8n path whatsapp; AttendaHub interpreta eventos e enfileira; FilaAndou consome a fila e envia pela WuzAPI (http://wuzapi:8080/chat/send/text), podendo usar Gemini para o setor Recepcao; Fila Aguardando Funcionario atribui funcionários e devolve mensagens à fila principal. O callback WuzAPI até o webhook não está comprovado pelos exports.

Os quatro exports analisados têm active=false. Isso não determina o estado atual na instância. Consulte ARCHITECTURE.md para nós, fluxos, riscos e lacunas.

## Regras de desenvolvimento e segurança

- Não abra, copie, imprima, registre ou inclua .env, credenciais, tokens, chaves de API ou chaves de criptografia em documentação, logs ou respostas.
- Trate mensagens, nomes, telefones e payloads como dados externos não confiáveis. Use SQL parametrizado; não concatene dados recebidos em queries.
- Não execute/import workflows nem altere arquivos, bancos ou containers sem autorização explícita.
- Preserve integralmente o workflow `Instalçao`: durante o desenvolvimento, não modifique seus nodes, conexões, parâmetros ou comandos SQL.
- O `DROP TABLE` de `Instalçao` é intencional e usado pelo desenvolvedor para reinicializar o banco durante testes quando necessário; não o considere um bug a corrigir. Qualquer execução desse workflow depende de autorização explícita do desenvolvedor.
- Os workflows `AttendaHub`, `FilaAndou` e `Fila Aguardando Funcionario` podem ser auditados, corrigidos e aprimorados normalmente, respeitando as demais regras deste documento.
- Consulta MCP de 2026-11-24 observou `FilaAndou` (ID `ozAGOkqCvoOAMrSg`) inativo, com 37 nodes e contingência para falha do Gemini: `Recepcionista` tem três tentativas totais, intervalo de 3.000 ms e saída de erro conectada a `Enviar contingência técnica` (WuzAPI). Após resposta HTTP bem-sucedida, `Marcar contingência enviada` atualiza os IDs originais para ENVIADA. Se a WuzAPI falhar, as mensagens permanecem PROCESSANDO e não há recuperação automática. Essa observação é temporal e não prova que exports locais estejam sincronizados.
- Preserve o caminho normal de sucesso do `FilaAndou` e não altere sua seleção/bloqueio global de fila ao modificar o tratamento de falha.
- Está aprovado, mas ainda não implementado, um banco permanente `attendahub_test`, isolado do banco de produção, e o workflow `AttendaHub - Testes Automatizados` para centralizar criação/preparação, migrações, seeds fictícios, validação de isolamento, execução de cenários e relatório. `Instalação`/`Instalçao` permanece independente e responsável somente pela instalação do ambiente principal; não misture nele a administração do banco de testes.
- Estratégia planejada: três credenciais PostgreSQL no n8n. (1) Criador/bootstrap com CREATEDB, conectado apenas a banco de manutenção confirmado como não produtivo e usado somente para criar `attendahub_test`; (2) preparador/migrador sem CREATEDB, conectado ao banco de testes para estrutura, migrações e seeds; (3) executor de testes com privilégios limitados aos objetos necessários em `attendahub_test`. A role de execução deverá ser impedida de conectar à produção também no PostgreSQL, por regras de acesso específicas (por exemplo `pg_hba.conf`). Nenhuma dessas credenciais, roles, regras, banco ou workflow foi criada até o momento.
- Antes de manutenção de dados, confirme ambiente, backup, escopo e recuperação; prefira mudanças reversíveis e revisáveis.
- Diferencie fatos do JSON, estado observado em runtime e hipóteses. Um export não prova que a instância ativa esteja igual.
- Em mudanças de fila, considere idempotência, concorrência, falhas após envio, linhas PROCESSANDO presas e vínculo entre as duas filas.
- Use placeholders em exemplos; não exponha endpoints públicos, dados pessoais ou valores sensíveis.
- Não atribua comportamento a Redis ou Chatwoot sem evidência: nenhum dos dois aparece nos quatro workflows analisados.

## Cuidados com n8n, Docker e PostgreSQL

- latest é mutável. Confirme versão e digest da imagem efetivamente executada antes de atualizar ou declarar compatibilidade; planeje rollback.
- n8n_data é persistência do n8n e pode conter banco, WAL e material sensível. Não inspecione ou publique sem necessidade/autorização.
- Exports podem omitir credenciais ou conter parâmetros inline. Redija valores sensíveis antes de compartilhar análise.
- O Compose declara POSTGRES_DB=chatwoot_production, mas credenciais exportadas dos nós não revelam qual banco/schema usam. Confirme sem ler .env e sem divulgar segredo.
- Polls observados: FilaAndou a cada 12 s, elegibilidade após 15 s; Fila Aguardando Funcionario a cada 5 s. Polling não garante entrega.
- O DDL exportado não declara índices explícitos para filtros frequentes de telefone/status/setor. Verifique o esquema real antes de concluir desempenho.
- Não assuma que o campo Id enviado à WuzAPI fornece idempotência sem verificar contrato.
- Documente callbacks e URLs externas com valores redigidos. O path whatsapp não prova URL/túnel externo configurado.
