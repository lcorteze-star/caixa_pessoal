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
