# Solicitação de Faturamento | Grupo Semar Supermercados

Portal web para **solicitar faturamento**, **acompanhar o andamento** das solicitações e **medir o desempenho do CPD** no Centro de Distribuição (CD) do Grupo Semar. Feito para ser usado principalmente pelo celular.

**Acesse:** https://logistica-semar.github.io/solicitacao-de-faturamento/

---

## O que o portal faz

O portal tem três abas:

| Aba | Para quê |
|---|---|
| **Solicitar** | Formulário de Solicitação de Faturamento (Google Forms), direto na página. |
| **Acompanhar** | Para a operação: quem fez o pedido vê na hora se a solicitação já foi atendida, sem precisar abrir a planilha. |
| **Métricas** | Para os gestores: desempenho do CPD (notas por faturista, tempo de faturamento, dias mais faturados). |

Na aba **Acompanhar**:

- **Busca** por loja, placa, nome do solicitante, faturista ou ID do pedido. Cada aparelho lembra a última busca.
- **Período:** Hoje, Ontem ou 7 dias.
- **Contadores por status**, separados em *Em aberto* (Pendente, Trabalhando, Backlog) e *Concluídos* (Finalizado, Cancelado). Tocar num contador filtra a lista.
- **Cartões de pedido**, todos do mesmo tamanho, com:
  - loja, placa, setor, solicitante, data e hora da solicitação;
  - ID completo do pedido (tocar copia o código);
  - trilha de andamento: Pendente, Trabalhando ou Backlog, Finalizado;
  - faturista e quantidade de notas;
  - tempo: nos pedidos em aberto ele corre a cada minuto (fica vermelho depois de 2 horas); nos Finalizados e Cancelados fica fixo, da solicitação até a última movimentação;
  - observações do solicitante e do CPD;
  - nos cancelados, o motivo informado pelo CPD.
- **Aviso de mudança:** quando um pedido muda de status, o cartão acende e aparece "1 pedido mudou de status".
- **Botão Ajuda** com WhatsApp, ligação e e-mail dos responsáveis.
- Modo escuro automático (segue o aparelho) e layout próprio para PC.

Na aba **Métricas** (período de 7, 30 ou 90 dias):

- **Destaque:** total de notas faturadas, variação contra o período anterior e curva de notas por dia (com média e melhor dia).
- **Indicadores:** pedidos, finalizados, tempo de faturamento (mediana) e cancelados, cada um com a variação contra o período anterior.
- **Quem mais faturou:** pódio dos 3 primeiros faturistas e lista dos demais (notas, pedidos, participação e tempo).
- **Ritmo do CPD:** mediana do tempo até finalizar e distribuição por faixas de tempo.
- **Pedidos por setor**, **Quando chegam os pedidos** (mapa de calor por dia da semana e hora) e **Dias mais faturados** (top 5).

O tempo de faturamento vai do **Carimbo de data/hora** até a hora em que o pedido virou **Finalizado** (ou Cancelado).

### Registro de tempos

O AppSheet não atualiza a coluna Última Movimentação quando o status muda. Para não mexer no AppSheet nem na planilha, o próprio Apps Script registra essa hora:

- **Comparação:** a cada leitura (de 15 em 15 min), o status de cada pedido é comparado com o da leitura anterior.
- **Onde fica:** quando um pedido vira Finalizado ou Cancelado, a hora fica guardada nas **Propriedades do script** (Configurações do projeto > Propriedades do script). Nenhum arquivo é criado no Drive e a planilha não é alterada; ela só é lida.
- **Espaço:** só ficam os dias usados pelas Métricas (cerca de 90); os mais antigos são apagados sozinhos. Com uns 60 pedidos por dia, usa perto de 190 KB dos 500 KB disponíveis.
- **Precisão:** a hora anotada é o meio do intervalo entre as duas leituras, com erro de até 7,5 minutos para mais ou para menos.
- **Intervalo grande:** se a leitura anterior foi há mais de 40 minutos, a hora fica em branco, para não registrar um valor impreciso.
- **Gatilho:** para não perder mudanças quando ninguém está com a página aberta, rode uma vez a função `instalarGatilho` no editor. Ela cria uma leitura automática de 15 em 15 minutos. Para desligar, rode `removerGatilho`.
- **Conferência:** a função `conferirRegistro` mostra no registro de execução quantos pedidos já têm tempo e o espaço usado.
- **Prioridade:** se o registro não tiver a hora, vale a Última Movimentação da planilha, desde que ela seja depois do Carimbo.

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
- **Apps Script (`Code.gs`):** publicado como App da Web, com duas rotas:
  - `.../exec`: últimos 7 dias, mais os pedidos antigos ainda em aberto (aba Acompanhar);
  - `.../exec?v=metricas`: base compacta de 90 dias (aba Métricas).
- **GitHub Pages (`index.html`):** página estática que busca o JSON e monta a tela. Não tem nenhum dado da planilha guardado no repositório.

### Atualização

**Acompanhar:** horários exatos de 15 em 15 minutos (**:00, :15, :30 e :45**).

**Métricas:** de hora em hora (horário cheio), e só quando alguém abre a aba.

- **Uma leitura por intervalo:** a planilha é lida no máximo 1 vez por intervalo (15 min no Acompanhar, 1 h nas Métricas), para todas as pessoas juntas. A primeira busca do intervalo grava os dados no cache do Apps Script, e as demais recebem essa cópia.
- **Sem leituras simultâneas:** uma trava (`LockService`) impede que duas pessoas disparem a leitura ao mesmo tempo na virada do horário.
- **F5 não gasta nada:** cada aparelho guarda a cópia dos pedidos. Abrir de novo ou atualizar a página dentro do mesmo intervalo não chama o Google.
- **Gatilho opcional:** com `instalarGatilho`, uma leitura automática a cada 15 minutos mantém o registro de tempos completo e deixa o cache pronto. Fora isso, se ninguém estiver olhando, nada roda.
- **Barra de progresso:** a linha vermelha no topo mostra quanto falta para a próxima atualização. Não existe botão para forçar leitura.
- **Relógio do Google:** a página usa a hora do servidor, então um aparelho com relógio adiantado não busca antes da hora.

Tudo roda nos planos gratuitos do Google e do GitHub.

## Arquivos

| Arquivo | Onde fica | O que é |
|---|---|---|
| `index.html` | Este repositório (GitHub Pages) | A página: visual, filtros, cartões, métricas e ajuda. |
| `Code.gs` | Projeto do Google Apps Script | A API de dados que lê a planilha. Fica neste repositório só como referência. |
| `Backup.gs` | Outro projeto do Google Apps Script | Backup diário que tira das tabelas originais o que passa de 30 dias. Fica neste repositório só como referência. |
| `logo-semar.png` | Este repositório | Logo oficial do Semar (sem fundo) usado no topo da página. |
| `favicon.svg`, `favicon-32.png`, `apple-touch-icon.png`, `icon-192.png`, `icon-512.png` | Este repositório | Ícone do site com o símbolo do Semar (aba do navegador e atalho na tela inicial do celular). |
| `site.webmanifest` | Este repositório | Nome e ícones usados quando alguém adiciona o portal à tela inicial. |
| `og-image.jpg` | Este repositório | Imagem de prévia ao compartilhar o link (WhatsApp, Teams, e-mail). |
| `README.md` | Este repositório | Esta documentação. |
| `LICENSE` | Este repositório | Termos de uso do código. |

## Backup de histórico (mais de 30 dias)

As tabelas originais (Pedidos, Notas Fiscais e Comentários) guardam só os últimos 30 dias. Todo o resto vai, de forma automática, para as planilhas de backup, que têm as mesmas colunas na mesma ordem. Isso fica num projeto do Apps Script separado (`Backup.gs`).

- **Quando roda:** uma vez por dia, por volta das 3h, por um gatilho diário. Leva poucos segundos.
- **Pedidos:** sai o pedido com mais de 30 dias que já terminou. Pedido Pendente, Trabalhando ou Backlog fica, mesmo antigo, até terminar.
- **Notas Fiscais e Comentários:** saem junto com o pedido: só quando têm mais de 30 dias e o pedido delas já saiu da tabela Pedidos. Assim o AppSheet nunca mostra um pedido sem as notas ou os comentários dele.
- **Ordem segura:** primeiro copia para o backup, depois apaga da original. Antes de apagar, confere de novo se as linhas são as mesmas; se alguém mexeu nelas durante o backup, ficam para o dia seguinte.
- **Sem duplicar:** o que já está no backup não é copiado de novo.
- **Formatos:** as planilhas de backup são Tabelas com colunas de tipo fixo, então o script não muda formato de célula. Textos que o Planilhas transformaria em número, como a loja "01", vão com apóstrofo e ficam iguais aos da original.
- **Proteção:** cada tabela original é encontrada pelo cabeçalho igual ao do backup. Se não achar, não mexe naquela tabela e registra o motivo. Se Pedidos der erro, Notas Fiscais e Comentários também não são mexidas.
- **Métricas de 90 dias:** como a planilha principal fica com 30 dias, a aba Métricas do site lê também o (Backup) Pedidos, no máximo 1 vez por hora e só quando alguém abre a aba. O Acompanhar continua lendo só a principal.

Funções do `Backup.gs`:

| Função | O que faz |
|---|---|
| `simularBackup` | Mostra o que seria movido, sem alterar nenhuma planilha. |
| `executarBackup` | Faz o backup agora (é a função que o gatilho diário chama). |
| `instalarGatilhoBackup` / `removerGatilhoBackup` | Liga ou desliga o backup diário automático. |
| `verUltimoBackup` | Mostra o resultado do último backup. |

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
| `DIAS_HISTORICO` | Dias enviados para o Acompanhar (padrão: 7). |
| `CICLO_MIN` | Intervalo de atualização do Acompanhar, em minutos (padrão: 15). |
| `DIAS_METRICAS` | Dias enviados para as Métricas (padrão: 90). |
| `CICLO_METRICAS_MIN` | Intervalo de atualização das Métricas, em minutos (padrão: 60). |
| `BACKUP_PEDIDOS_ID` / `BACKUP_PEDIDOS_GID` | Planilha e aba do (Backup) Pedidos, lidas pelas Métricas para completar os 90 dias. |

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
