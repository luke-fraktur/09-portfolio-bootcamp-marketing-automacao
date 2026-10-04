# Desafio Criativo: Planejando Automações com n8n Usando Bons Prompts

**Aluno:** Lucas Oliveira Soares de Freitas  
**Projeto/Empresa:** Agência Escala Pro  
**Caminho escolhido:** Google Drive  
**Data de conclusão:** 04/09/2026  

---

## 1. Definição da automação desejada

### Processo a ser automatizado

Quero criar uma automação no **n8n** para organizar automaticamente os documentos recebidos pela Agência Escala Pro em uma pasta de entrada do Google Drive. O fluxo deverá identificar novos arquivos, validar seu tipo e seus metadados, classificá-los conforme o projeto ou cliente indicado no nome do arquivo e movê-los para a subpasta correspondente. Ao final, a automação deverá registrar o processamento em uma planilha de controle e enviar uma notificação de conclusão ou de erro.

### Público ou responsáveis

A automação será utilizada pela equipe operacional e comercial da Agência Escala Pro, especialmente por Lucas Oliveira Soares de Freitas e pelos responsáveis pelo atendimento dos clientes. O objetivo é reduzir tarefas manuais, evitar arquivos fora do padrão e criar rastreabilidade sobre cada documento processado.

### Resultado esperado

O resultado esperado é que cada novo arquivo seja processado uma única vez, encaminhado para a pasta correta do Google Drive, registrado com data, nome, tipo, status e mensagem de processamento em uma planilha de controle e comunicado à equipe responsável. Arquivos inválidos ou sem informações mínimas deverão ser enviados para uma pasta de pendências, sem interromper o processamento dos demais itens.

---

## 2. Contexto, ferramentas e regras de negócio

### Ferramentas envolvidas

| Ferramenta | Função no processo |
|---|---|
| **n8n** | Orquestrar o workflow, aplicar regras, tratar exceções e registrar o resultado. |
| **Google Drive** | Monitorar a pasta de entrada, consultar arquivos, criar ou localizar pastas, mover documentos e armazenar pendências. |
| **Google Sheets** | Manter o log operacional de arquivos processados, status, timestamps e mensagens. |
| **Gmail** | Enviar notificações de sucesso, pendência ou falha técnica à equipe responsável. |

O nó Google Drive do n8n oferece operações para listar, pesquisar, criar, mover, atualizar, fazer upload, baixar e compartilhar arquivos e pastas [1]. Para iniciar a automação a partir de novos eventos, pode-se utilizar o Google Drive Trigger, configurado para verificar eventos compatíveis na pasta monitorada [2].

### Estrutura sugerida no Google Drive

| Pasta | Finalidade |
|---|---|
| `01_ENTRADA_DOCUMENTOS` | Pasta observada pelo gatilho do workflow. |
| `02_CLIENTES` | Diretório das pastas de clientes ou projetos. |
| `03_PENDENCIAS` | Arquivos sem padrão, sem projeto identificável ou com falha de validação. |
| `04_ARQUIVADOS` | Arquivos já processados e encerrados. |

A planilha `LOG_AUTOMACOES_N8N` deverá possuir as colunas `data_processamento`, `arquivo_id`, `nome_arquivo`, `mime_type`, `cliente_projeto`, `status`, `pasta_destino`, `mensagem` e `execution_id`.

### Regras importantes

1. O fluxo deve processar somente arquivos criados ou disponibilizados na pasta `01_ENTRADA_DOCUMENTOS`.
2. O identificador do arquivo deve ser usado como chave de idempotência. Se o mesmo `arquivo_id` já estiver registrado como processado, o workflow deverá encerrar o item sem duplicar movimentações ou notificações.
3. O nome esperado deve seguir o padrão `CLIENTE_PROJETO_TIPO_DATA.ext`, por exemplo `ACME_PLANO_MENSAL_2026-08-13.pdf`. Quando o nome não seguir o padrão mínimo, o arquivo deverá ser movido para `03_PENDENCIAS`.
4. Serão aceitos, inicialmente, arquivos PDF, DOCX, XLSX, CSV, PNG e JPG. Extensões não permitidas deverão ser direcionadas para pendência.
5. Se a pasta do cliente ou projeto não existir, o fluxo deverá criá-la dentro de `02_CLIENTES` ou sinalizar a pendência conforme a política de segurança da operação.
6. Arquivos vazios, sem nome, sem identificação do projeto ou com duplicidade deverão ser marcados como `PENDENTE_VALIDACAO`.
7. Cada item deverá gerar um registro no Google Sheets, independentemente de o resultado ser `PROCESSADO`, `PENDENTE` ou `ERRO`.
8. O fluxo não deverá apagar arquivos. Em caso de erro, o item deve permanecer recuperável no Google Drive.
9. As credenciais deverão ser configuradas no n8n por meio do mecanismo próprio de credenciais; nenhum token ou senha deverá ser escrito no prompt, na planilha ou em campos públicos.
10. Falhas temporárias de comunicação deverão ser registradas e notificadas, sem mascarar a causa original.

---

## 3. Prompt final para orientar a criação do workflow

```text
Atue como um especialista em n8n, automação de processos e Google Drive.

Crie uma automação para organizar documentos recebidos pela Agência Escala Pro.

Público:
A equipe operacional e comercial da Agência Escala Pro, incluindo Lucas Oliveira Soares de Freitas e os responsáveis pelo atendimento dos clientes.

Objetivo:
Monitorar uma pasta de entrada do Google Drive, validar cada novo arquivo, identificar o cliente ou projeto pelo nome do arquivo, mover o documento para a pasta de destino correta, registrar o processamento em uma planilha Google Sheets e notificar a equipe por Gmail.

Ferramentas envolvidas:
- n8n;
- Google Drive;
- Google Sheets;
- Gmail.

Estrutura do Google Drive:
- 01_ENTRADA_DOCUMENTOS: pasta monitorada;
- 02_CLIENTES: pasta-raiz dos clientes e projetos;
- 03_PENDENCIAS: arquivos que falharem na validação;
- 04_ARQUIVADOS: arquivos encerrados, quando aplicável.

Fluxo desejado:
1. Inicie com o Google Drive Trigger monitorando a pasta 01_ENTRADA_DOCUMENTOS para novos arquivos.
2. Consulte os metadados do arquivo no Google Drive, incluindo id, nome, mimeType, tamanho, data de criação e parent folder.
3. Verifique se o arquivo pertence à pasta monitorada e se seu arquivo_id já existe na planilha LOG_AUTOMACOES_N8N.
4. Se o arquivo já estiver registrado como PROCESSADO, encerre o item sem repetir movimentação, log ou notificação.
5. Valide se o arquivo possui nome, tamanho maior que zero e uma extensão permitida: PDF, DOCX, XLSX, CSV, PNG ou JPG.
6. Valide o padrão mínimo do nome CLIENTE_PROJETO_TIPO_DATA.ext. Extraia o cliente ou projeto a partir do nome, sem presumir dados ausentes.
7. Se a validação falhar, mova o arquivo para 03_PENDENCIAS, registre o status PENDENTE_VALIDACAO no Google Sheets e envie um e-mail com a causa objetiva da pendência.
8. Se a validação for aprovada, pesquise no Google Drive a pasta do cliente ou projeto dentro de 02_CLIENTES.
9. Se a pasta não existir, crie-a somente quando essa ação estiver autorizada; caso contrário, mova o arquivo para 03_PENDENCIAS e registre PASTA_NAO_ENCONTRADA.
10. Mova o arquivo aprovado para a pasta de destino usando o identificador do arquivo e o identificador da pasta, evitando decisões baseadas apenas no nome.
11. Registre no Google Sheets data_processamento, arquivo_id, nome_arquivo, mime_type, cliente_projeto, status, pasta_destino, mensagem e execution_id.
12. Envie uma notificação de sucesso por Gmail com o nome do arquivo, a pasta de destino e o link do Google Drive.
13. Configure uma rota de erro para registrar status ERRO, preservar o arquivo, salvar a mensagem técnica e notificar o responsável.

Nós n8n sugeridos e lógica:
- Google Drive Trigger: inicia o fluxo quando houver novo arquivo na pasta de entrada.
- Google Drive — Get/Search: obtém metadados e pesquisa registros ou pastas.
- IF: separa idempotência, validação aprovada e pendência.
- Code ou Edit Fields: normaliza nome, extensão, cliente/projeto e campos do log; não invente valores ausentes.
- Google Sheets — Lookup/Append Row: consulta a chave arquivo_id e grava o resultado do processamento.
- Google Drive — Create Folder: cria a pasta de destino quando permitido.
- Google Drive — Move File: encaminha o arquivo aprovado ou pendente.
- Gmail — Send: notifica sucesso, pendência ou erro.
- Error Trigger ou workflow de erro: centraliza falhas e permite o registro operacional.

Regras e exceções:
- Não excluir arquivos.
- Não expor credenciais, tokens ou dados sensíveis.
- Não processar duas vezes o mesmo arquivo_id.
- Não considerar uma curtida, nome ou extensão como prova suficiente quando os metadados forem inconsistentes.
- Não interromper o processamento de todos os itens por causa de um único arquivo inválido.
- Registrar sempre o resultado final no Google Sheets.
- Usar mensagens de erro claras, curtas e acionáveis.

Entregue:
1. A descrição da arquitetura do workflow.
2. A lista de nós em ordem, com a responsabilidade de cada um.
3. As condições de cada decisão IF.
4. O esquema de dados da planilha de log.
5. O tratamento de erros e a estratégia de idempotência.
6. Um roteiro de teste com casos de sucesso, pendência, duplicidade, extensão inválida e falha de credencial.
7. Uma lista de credenciais que precisam ser configuradas no n8n, sem incluir valores secretos.
```

---

## 4. Arquitetura operacional do workflow

### Sequência dos nós

| Ordem | Nó | Responsabilidade | Saída principal |
|---:|---|---|---|
| 1 | Google Drive Trigger | Detectar novo arquivo na pasta de entrada. | `file_id`, nome e evento. |
| 2 | Google Drive — Get | Buscar metadados completos. | `mimeType`, tamanho, pais e links. |
| 3 | Google Sheets — Lookup | Consultar se o `file_id` já foi processado. | Registro existente ou vazio. |
| 4 | IF — Idempotência | Encerrar duplicados ou liberar novos itens. | Rota duplicada ou nova. |
| 5 | Code/Edit Fields | Normalizar nome, extensão e cliente/projeto. | Campos derivados e flags. |
| 6 | IF — Validação | Separar aprovado de pendente. | Rota aprovada ou pendente. |
| 7 | Google Drive — Search | Encontrar a pasta de destino. | `folder_id` ou ausência. |
| 8 | Google Drive — Create Folder | Criar pasta quando permitido. | Nova pasta de destino. |
| 9 | Google Drive — Move File | Mover o arquivo para a pasta correta. | Confirmação de movimentação. |
| 10 | Google Sheets — Append Row | Registrar sucesso, pendência ou erro. | Linha do log operacional. |
| 11 | Gmail — Send | Notificar o responsável. | Confirmação do envio. |
| 12 | Workflow de erro / Error Trigger | Registrar e comunicar falhas inesperadas. | Log de erro e alerta. |

### Condições principais

A primeira condição verifica se o `arquivo_id` já possui registro `PROCESSADO`. A segunda valida presença de nome, tamanho, extensão permitida e padrão mínimo de identificação. A terceira verifica se a pasta do cliente/projeto foi encontrada ou criada. Cada rota deve terminar com um registro no log, garantindo rastreabilidade mesmo nos casos de pendência ou erro.

### Estratégia de idempotência

A chave operacional será o `arquivo_id` nativo do Google Drive. Antes de qualquer movimentação, o workflow consulta esse valor no Google Sheets. Um item previamente marcado como `PROCESSADO` não deve ser movido novamente nem gerar novo alerta. Para maior robustez em produção, recomenda-se substituir ou complementar a planilha por uma base com índice único quando o volume de processamento justificar essa evolução.

---

## 5. Roteiro de testes e critérios de aceite

| Caso | Entrada | Resultado esperado |
|---|---|---|
| Sucesso | PDF com nome no padrão e cliente existente. | Arquivo movido, log `PROCESSADO` e e-mail de sucesso. |
| Pasta ausente | Arquivo válido, mas cliente não localizado. | Criar pasta se autorizado; caso contrário, `PENDENTE` e movimentação para pendências. |
| Extensão inválida | Arquivo `.exe` ou outra extensão não permitida. | Não processar como válido; mover para pendências e registrar motivo. |
| Nome incompleto | Arquivo sem cliente/projeto ou data mínima. | `PENDENTE_VALIDACAO`, sem inferência de dados. |
| Duplicidade | Mesmo `arquivo_id` já processado. | Encerrar sem duplicar ações e registrar, se necessário, status `IGNORADO_DUPLICADO`. |
| Arquivo vazio | Tamanho igual a zero. | Mover para pendências e registrar `ARQUIVO_VAZIO`. |
| Falha de credencial | Acesso ao Drive, Sheets ou Gmail indisponível. | Acionar tratamento de erro, preservar o arquivo e enviar alerta técnico quando possível. |

O aceite do workflow ocorrerá quando todos os casos acima produzirem o resultado esperado, quando não houver credenciais expostas e quando cada execução puder ser rastreada pelo `execution_id` e pelo `arquivo_id`.

---

## 6. Credenciais e segurança

Antes de ativar o workflow, devem ser configuradas no n8n as credenciais OAuth ou equivalentes para Google Drive, Google Sheets e Gmail. Os nomes das credenciais podem ser, respectivamente, `Google Drive - Agência Escala Pro`, `Google Sheets - Log Automacoes` e `Gmail - Notificacoes Operacionais`. Os valores secretos não devem ser inseridos neste documento nem compartilhados no Google Drive.

O workflow deverá ser testado primeiro em uma pasta de homologação, com dados não sensíveis. A pasta de produção só deverá ser monitorada após a validação de idempotência, permissões, regras de nomenclatura e tratamento de erros.

---

## 7. Entrega e publicação pelo Google Drive

O arquivo desta entrega deverá ser salvo como `.md` ou `.txt`. Para submeter pela opção escolhida, faça upload no Google Drive, abra o menu **Compartilhar**, altere o acesso geral para **Qualquer pessoa com o link** com permissão de leitor, copie o link e cole-o no campo de submissão da DIO. O link deve permitir a leitura do arquivo sem expor as credenciais usadas no n8n.

---

## Referências

[1]: https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.googledrive "n8n Docs — Google Drive node"
[2]: https://docs.n8n.io/integrations/builtin/trigger-nodes/n8n-nodes-base.googledrivetrigger "n8n Docs — Google Drive Trigger"
[3]: https://docs.n8n.io/build/flow-logic/handle-errors-gracefully "n8n Docs — Handle errors gracefully"
[4]: https://docs.n8n.io/build/work-with-data/expressions-versus-data-nodes "n8n Docs — Expressions versus data nodes"

---

**Autor da elaboração:** Manus AI  
**Aluno:** Lucas Oliveira Soares de Freitas  
**Data:** 04/09/2026
