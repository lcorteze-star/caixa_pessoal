# Formatos de importação — o que aprendemos

Regras extraídas de arquivos reais (set/out 2026). **Não há dados pessoais neste arquivo.**
Serve de especificação para os leitores (parsers) do app e para o briefing do Lovable.

## 1. Fatura BTG (PDF)

- Texto extraível (não é imagem). Lançamentos em **duas colunas por linha**: `DD Mon  Nome (n/N)   R$ valor`.
  - Um regex com `finditer` por linha lê as duas colunas.
- Seções por cartão: `Lançamentos do cartão <físico|virtual|adicional> | <titular> | Final NNNN  Total do cartão: R$ X`.
  - Titular e final do cartão identificam **quem** gastou (cartão adicional = o outro cônjuge).
- Data sem ano: o mês maior que o mês da fatura pertence ao **ano anterior** (compras parceladas antigas).
- Parcela vem no nome: `(5/12)`. Compra à vista não tem marcador.
- Seção "Pagamentos feitos pelo cliente" **não é despesa** (pagamento da fatura anterior).
- Texto de rodapé (juros, rotativo, parcelamento da fatura) é ruído, e não deve virar lançamento.
- **Validação obrigatória**: soma de cada seção = `Total do cartão` impresso. Bateu centavo a centavo nos 6 cartões testados.
- A data do lançamento parcelado é a da **compra original**, e não a do mês da cobrança.

## 2. Fatura Inter (CSV)

- Colunas: `Data, Lançamento, Categoria, Tipo, Valor`. Aspas em todos os campos, UTF-8 **com BOM** (usar `utf-8-sig`).
- Valor `R$ 1.234,56` com **espaço não separável (U+00A0)** entre `R$` e o número. Normalizar antes de converter.
- `Tipo`: `Compra à vista` ou `Parcela n/N`.
- A linha `PAGAMENTO ON LINE` (valor negativo) é **pagamento da fatura anterior**, e não despesa.
  - Dica de conciliação: ela é igual ao total da fatura do mês anterior.
- `IOF INTERNACIONAL` vem como linha própria, ligada a compras no exterior.
- O banco já manda uma categoria (`Categoria`). Usar como sugestão, e não como verdade.
- **Linhas idênticas na mesma fatura são legítimas** (dois cartões, duas compras iguais). Não deduplicar dentro do mesmo arquivo.

## 3. Extrato de conta corrente BTG (PDF)

- Linhas: `DD/MM/AAAA HHhMM  [Categoria]  Transação  Descrição  Valor`.
- Linhas `Saldo Diário` (23h59) servem de **checkpoint de conciliação**, e não são lançamentos.
- Quebras de layout:
  - A descrição pode vir em **linha separada, antes** da linha da data (ocorre quando a categoria é "Outra Categoria").
  - Em `Pagamento de boleto/conta`, o favorecido vem colado ao tipo de transação.
  - A categoria às vezes quebra em duas linhas (ex.: `Lazer e` / `Entretenimento`).
- Cabeçalho repete a cada página. Descartar.
- **Rendimento automático**: o saldo cresce em centavos por dia sem lançamento. A soma dos lançamentos fica alguns reais abaixo do saldo final. É rendimento, e não perda de linha.
- Tipos vistos: Pix enviado/recebido, Transferência enviada/recebida (investimentos), Pagamento de boleto, Pagamento de conta, Pagamento de fatura do cartão, Devolução de Pix.

## 4. Comprovantes (imagem/PDF)

- Comprovante de transferência (Pix) e de pagamento de boleto: valor, data/hora, favorecido, pagador, ID/autenticação.
- Servem para **confirmar** uma linha do extrato (mesmo valor, mesma data, mesmo favorecido), e não para criar lançamento novo.
  - Se a linha do extrato já existe, o comprovante só a enriquece (favorecido completo, boleto).
- Capturas de tela do app do banco (lista de compras recentes): servem para lançamentos do mês corrente. **Podem se sobrepor** entre capturas, e a mesma compra aparece em duas imagens. Deduplicar por data, valor e estabelecimento.

## 5. Regras de negócio que apareceram

| Situação | Tratamento |
|---|---|
| Pagamento de fatura (extrato) | Não é despesa; liquida a fatura do cartão |
| `PAGAMENTO ON LINE` no cartão | Não é despesa; é o mesmo evento visto do outro lado |
| Transferência entre contas do casal | Não é receita nem despesa |
| Transferência para "Investimentos" | Movimento para poupança, e não despesa |
| Compra parcelada `n/N` | Gera `N-n` parcelas futuras no mesmo valor |
| Boleto de terceiros (financiamento, consórcio, cartão de outro banco) | **Perguntar** ao usuário o que é, e lembrar da resposta |
| Pix para pessoa física | Fila de revisão (pode ser diarista, família, reembolso) |
| Cartão adicional | Despesa do titular do adicional, mesmo na fatura do principal |

## 6. Classificação: o que funcionou

- Regras por palavra-chave no estabelecimento cobrem a maior parte das linhas.
- Sobra uma fração significativa (≈ 1 em 5 linhas) para revisão humana: nomes de pessoas, lojas de nicho, razões sociais opacas.
- Cada correção do usuário vira regra para os meses seguintes.
- Normalizar acentos e caixa antes de comparar; atenção a espaços (`CASAS BAHIA` vs `CASASBAHIA`).

## 7. Novos formatos (lote de 92 arquivos)

### Fatura Riachuelo/Midway (PDF)
- Texto extraível. Tabela `Data | Loja | Descrição | Valor original | Parcela (nn/nn) | Valor do mês`.
- Compras parceladas trazem **data da compra original**. `PAGAMENTO` aparece como linha negativa (não é despesa).
- A página 2 em diante é boleto/propaganda, e deve ser ignorada. Seção "Total a vencer" dá o saldo futuro das parcelas.
- Pagamento feito por Pix aparece em outra conta (a do pagador), então a fatura isolada não diz de onde saiu o dinheiro.

### Comprovantes (imagem)
- A extração de texto de imagem funciona bem em comprovantes de banco. Modelos vistos: BTG (transferência, pagamento de boleto, pagamento de fatura), Inter (Pix enviado, pagamento de convênio/boleto, pagamento de fatura) e recibo de maquininha.
- Campos comuns: valor, data/hora, favorecido (nome e CNPJ/CPF), pagador, instituição, ID/autenticação.
- Boleto de órgão público (IPVA, IPTU, Detran, multa) identifica o órgão no "código de convênio" ou no nome do favorecido.
- Pix para `Receita Federal` costuma ser imposto (DAS, INSS de empregado doméstico, renegociação). Perguntar ao usuário e lembrar.
- **O nome do arquivo não é confiável** para mês/competência. Um arquivo chamado `…1026` tinha data de pagamento em 08/09. Usar sempre a data do documento.
- O pagador do comprovante pode ser **outra conta ou pessoa jurídica** do mesmo casal. O app precisa permitir várias contas por pessoa, inclusive PJ.

### Capturas de tela do app do banco (fatura em aberto)
- Lista por dia, com tipo, estabelecimento, categoria do banco e valor. Ordem decrescente, em rolagem.
- Capturas consecutivas **se sobrepõem** (o fim de uma repete o começo da outra). Deduplicar por data + estabelecimento + valor **somente entre capturas**, nunca dentro da mesma imagem.
- Cabeçalho mostra o total da fatura aberta, a previsão das faturas futuras (parcelas já assumidas), vencimento e melhor dia de compra. São dados valiosos para conferência.
- O OCR perde linhas e mistura colunas. Mostrar o total do app ao lado da soma lida, e exibir a diferença ao usuário.

## 8. Conferência da previsão de parcelas

Projetar `N-n` parcelas futuras de cada lançamento da última fatura reproduziu a previsão que BTG e Inter exibem no app, com diferença de centavos (R$ 0,15 a R$ 0,18) em 3 meses seguidos. Usar essa comparação como teste automático do motor de previsão.

## 9. Modelo de contas: o que o app precisa suportar

- Várias contas por pessoa, em bancos diferentes (BTG, Inter), incluindo conta de pessoa jurídica.
- A mesma pessoa pode aparecer com **nomes diferentes** em contas distintas (nome civil vs. nome completo). Casar por CPF/CNPJ ou por regra do usuário.
- A fatura de um cartão pode ser paga por contas diferentes a cada mês.
- Pagamentos de despesa da casa feitos por uma PJ do casal precisam ser marcados (origem PJ) para não distorcer o orçamento da família.
- Despesas anuais atrasadas ou concentradas (IPVA, IPTU, licenciamento, multas) precisam de categoria própria e opção "não recorrente" para não distorcer médias.

## 10. Formatos do segundo lote

### Extrato Inter (CSV)
- Separador `;`, cabeçalho de 3 linhas (`Conta`, `Período`, `Saldo`), depois `Data Lançamento;Histórico;Descrição;Valor;Saldo`.
- Valores no padrão BR (`-2.161,55`), datas `DD/MM/AAAA`, **ordem decrescente** (mais novo primeiro). Espaço sobrando no fim de `Histórico` (`Pix enviado `): aplicar trim.
- A coluna `Saldo` permite conferir a transcrição: saldo final menos saldo inicial = soma dos lançamentos. Conferido sem diferença.
- Tipos: Pix enviado/recebido, Compra no débito, Pagamento de Convênio (órgãos públicos), Pagamento de Título, Pagamento efetuado (fatura do cartão).
- O mesmo extrato existe em PDF. Preferir o CSV e usar o PDF só como reserva.

### Extrato BTG (XLS)
- É um `.xls` antigo (gerado por JasperReports). Pandas precisa do `xlrd`. Cabeçalho nas primeiras 10 linhas, colunas esparsas e células mescladas: descartar colunas vazias.
- Contém as linhas `Saldo Diário` misturadas aos lançamentos. Filtrar.
- No app final, aceitar também XLSX, e orientar o usuário a baixar CSV quando o banco oferecer.

### Guia de parcelamento de IPTU (PDF da Prefeitura)
- A primeira parte traz o acordo (número de parcelas, valor total, dívidas incluídas). Depois vêm as guias, uma por parcela, cada uma com vencimento e valor.
- **A parcela cresce todo mês**: `valor_base × (1 + 1% × (n−1))`. Não projetar parcela fixa. O motor de previsão precisa aceitar uma regra de reajuste.
- O PDF não informa quais parcelas já foram pagas. Cruzar com os extratos.

## 11. Regras de negócio novas
- Uma pessoa jurídica do casal pode **pagar fatura de cartão e despesas pessoais** e também transferir valores para as contas da família. Marcar a origem (PF/PJ) e dar ao usuário a escolha de contar esses repasses como receita.
- Transferências de ida e volta entre PJ e PF no mesmo mês (valor igual) se anulam e não são receita.
- O nome do arquivo ou do comprovante não identifica o tipo do imposto. Foi preciso perguntar ao usuário (INSS da babá vs. Simples da empresa), e a resposta vira regra por favorecido + faixa de valor.
- Impostos de empresa pagos pelo casal entram numa categoria separada para não poluir o orçamento da casa.
- Despesas anuais já conhecidas (IPVA em cotas, IPTU parcelado) devem aparecer na previsão dos meses em que vencem.

## 12. Contrato de financiamento imobiliário (PDF do banco, "DDC")
- Documento de **Demonstrativo Descritivo de Crédito**: cabeçalho com prazo total, prazo remanescente, taxa, sistema de amortização (SAC), data da última parcela.
- Corpo: tabela corrida de **todas as parcelas, passadas e futuras**, com colunas: nº, vencimento, amortização, juros, índice, seguros (MIP, DFI, residencial), taxa de administração, situação (`Paga`, `Emitida`, `Projetada`) e valor total da parcela e saldo devedor.
- O texto vem **sem quebra de linha entre as parcelas**: precisa de regex por linha lógica (`nº dd/mm/aaaa ... Situação valor saldo`).
- A situação `Paga` / `Emitida` / `Projetada` diz o que já foi pago, o que está emitido e o que é estimativa. Usar isso diretamente na previsão: parcelas `Projetada` são previsão, e não compromisso certo.
- Linhas `Operação: Pula parcela` indicam parcelas **puladas** (valor incorporado ao saldo devedor). Merecem alerta ao usuário, porque aumentam o saldo.
- Parcela em SAC **cai todo mês**. O motor de previsão deve ler a tabela do contrato, e não repetir o último valor.

## 13. Extrato Inter PJ (CSV)
- Mesmo layout do extrato Inter PF. A conta é de pessoa jurídica (CNPJ do titular).
- Entradas concentradas em poucos pagadores. O app pode destacar **concentração de renda** (percentual da renda que vem do maior pagador).
- Pagamentos da empresa para a família (retirada, pró-labore) e da família para a empresa precisam ser tratados como transferência entre PF e PJ.
