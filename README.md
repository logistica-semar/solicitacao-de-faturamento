# Acompanhamento de Faturamento | Grupo Semar Supermercados

Portal web para **solicitar faturamento** e **acompanhar o andamento** das solicitações do Centro de Distribuição (CD) do Grupo Semar, feito para ser usado principalmente pelo celular.

**Acesse:** https://logistica-semar.github.io/acompanhamento-faturamento/

---

## O que o portal faz

O portal tem duas abas:

| Aba | Para quê |
|---|---|
| **Solicitar** | Formulário de Solicitação de Faturamento (Google Forms), direto na página. |
| **Acompanhar** | Quem fez o pedido vê na hora se a solicitação já foi atendida, sem precisar abrir a planilha. |

Na aba **Acompanhar**:

- **Busca** por loja, placa, nome do solicitante, faturista ou ID do pedido. Cada aparelho lembra a última busca.
- **Período:** Hoje, Ontem ou 7 dias.
- **Contadores por status**, separados em *Em aberto* (Pendente, Trabalhando, Backlog) e *Concluídos* (Finalizado, Cancelado). Tocar num contador filtra a lista.
- **Cartões de pedido**, todos do mesmo tamanho, com:
  - loja, placa, setor, solicitante e horário;
  - ID completo do pedido (tocar copia o código);
  - trilha de andamento: Pendente, Trabalhando ou Backlog, Finalizado;
  - faturista, quantidade de notas e tempo de espera (fica vermelho depois de 2 horas);
  - observações do solicitante e do CPD;
  - nos cancelados, o motivo informado pelo CPD.
- **Aviso de mudança:** quando um pedido muda de status, o cartão acende e aparece "1 pedido mudou de status".
- **Botão Ajuda** com WhatsApp, ligação e e-mail dos responsáveis.
- Modo escuro automático (segue o aparelho) e layout próprio para PC.

## Status

| Status | Significado |
|---|---|
| Pendente | Pedido recebido, aguardando o faturamento. |
| Trabalhando | O faturista está faturando o pedido. |
| Backlog | Pedido em espera. |
| Finalizado | Pedido faturado. |
| Cancelado | Pedido cancelado. O motivo vem da coluna **Observações**, preenchida pelo CPD. |

## Como funciona

```
Google Forms ──> Planilha (aba Pedidos + aba Colaboradores)
                        │
                        │  lida no máximo 1 vez a cada 15 min
                        ▼
              Google Apps Script (Code.gs)
              entrega os pedidos em JSON
                        │
                        ▼
              GitHub Pages (index.html)
              página que as pessoas abrem
```

- **Planilha:** a aba `Pedidos` é a tabela de fatos. A aba `Colaboradores` é a dimensão que troca o e-mail do faturista pelo nome.
- **Apps Script (`Code.gs`):** publicado como App da Web. Lê só os últimos 7 dias, mais os pedidos antigos que ainda estejam em aberto, e devolve um JSON compacto.
- **GitHub Pages (`index.html`):** página estática que busca o JSON e monta a tela. Não tem nenhum dado da planilha guardado no repositório.

### Atualização de 15 em 15 minutos

A atualização acontece em horários exatos: **:00, :15, :30 e :45**.

- **Uma leitura por intervalo:** a planilha é lida no máximo 1 vez a cada 15 minutos, para todas as pessoas juntas. A primeira busca do intervalo grava os dados no cache do Apps Script, e as demais recebem essa cópia.
- **Sem leituras simultâneas:** uma trava (`LockService`) impede que duas pessoas disparem a leitura ao mesmo tempo na virada do horário.
- **F5 não gasta nada:** cada aparelho guarda a cópia dos pedidos. Abrir de novo ou atualizar a página dentro do mesmo intervalo não chama o Google.
- **Sem ninguém olhando, nada roda:** o projeto não usa gatilhos (triggers).
- **Barra de progresso:** a linha vermelha no topo mostra quanto falta para a próxima atualização. Não existe botão para forçar leitura.
- **Relógio do Google:** a página usa a hora do servidor, então um aparelho com relógio adiantado não busca antes da hora.

Tudo roda nos planos gratuitos do Google e do GitHub.

## Arquivos

| Arquivo | Onde fica | O que é |
|---|---|---|
| `index.html` | Este repositório (GitHub Pages) | A página: visual, filtros, cartões e ajuda. |
| `Code.gs` | Projeto do Google Apps Script | A API de dados que lê a planilha. Fica neste repositório só como referência. |
| `README.md` | Este repositório | Esta documentação. |
| `LICENSE` | Este repositório | Termos de uso do código. |

## Configuração

**`index.html`** (objeto `CONFIG`, no início do script):

| Campo | Para quê |
|---|---|
| `API_URL` | Endereço do App da Web do Apps Script (termina em `/exec`). |
| `FORM_URL` | Endereço do Google Forms de solicitação. |
| `LOGO_URL` e `LOGO_RECORTE` | Imagem do logo e a área usada dela. |

**`Code.gs`** (objeto `CONFIG`, no início do arquivo):

| Campo | Para quê |
|---|---|
| `PLANILHA_ID` | ID da planilha de Solicitação de Faturamento. |
| `ABA_PEDIDOS` / `ABA_COLABORADORES` | Nomes das abas lidas. |
| `DIAS_HISTORICO` | Quantos dias de pedidos são enviados (padrão: 7). |
| `CICLO_MIN` | Intervalo de atualização em minutos (padrão: 15). |

As colunas da aba `Pedidos` são encontradas pelo **nome do cabeçalho**, não pela letra. Mudar uma coluna de lugar não quebra nada, mas renomear um cabeçalho quebra. Cabeçalhos usados:

`ID Pedido`, `Carimbo de data/hora`, `Nome do Solicitante`, `Setor Solicitante`, `Loja`, `Placa`, `Unitizador`, `Observações do Solicitante`, `Status`, `Última Movimentação`, `Faturista`, `Qtd. Notas`, `Observações`.

## Como publicar uma alteração

**Página (`index.html`)**

1. Neste repositório, envie o novo `index.html` (**Add file > Upload files**) e confirme o commit.
2. Em 1 ou 2 minutos o GitHub Pages atualiza o site.

**API (`Code.gs`)**

1. No Google Apps Script, substitua o código pelo novo `Code.gs`.
2. Vá em **Implantar > Gerenciar implantações > Editar** e escolha **Nova versão**.
3. Mantenha **Executar como: Eu** e **Quem pode acessar: Qualquer pessoa**.

Editando a implantação existente, o endereço `/exec` continua o mesmo e a página não precisa mudar.

Para medir o tempo de leitura da planilha, rode a função `testarTempoDeLeitura` no editor do Apps Script e veja o registro de execução.

## Privacidade

- O repositório contém apenas o código da página. Nenhum dado da planilha fica salvo aqui.
- A planilha pode ser privada: quem lê os dados é o Apps Script, com a permissão da conta dona do script.
- A API entrega apenas as colunas usadas pelo portal e troca o e-mail do faturista pelo nome.

## Tecnologias

HTML, CSS e JavaScript puros (sem frameworks), Google Apps Script, Google Sheets, Google Forms e GitHub Pages.

## Autoria

Desenvolvido por **Luccas Souza** para a Logística do Grupo Semar Supermercados, CD Suzano.

Dúvidas sobre o uso: botão **Ajuda** no próprio portal.
