# Caixa Pessoal — Plano do Produto

App de finanças do casal (PWA), dentro do Ecossistema Única.
Primeiros usuários: Luiz e Caroline. Depois família, amigos e público.

## Princípio central

O app responde, na hora da compra: **"quanto ainda posso gastar neste mês e quanto já está comprometido nos próximos?"**

## Decisões já tomadas

- **Plataforma:** PWA (instalável em iPhone e Android). Empacotar para as lojas depois.
- **Cliente Única:** acesso completo grátis via **código de convite**. Não cliente: plano básico, com upgrade pago.
- **Visão:** consolidada do casal e individual de cada um.
- **Identidade visual:** cores e logos do Ecossistema Única.

## Fase 1 — Histórico, classificação e plano futuro

Objetivo: o app conhece a família **antes** de pedir lançamentos diários.

### Fluxo

1. **Grupo "Casal"**: login, convite da Caroline, contas e cartões (BTG, Inter, Riachuelo, conta corrente de cada um).
2. **Importar histórico**: PDF, CSV ou OFX de faturas e extratos, e comprovantes de pagamento (PDF ou foto).
3. **Leitura e classificação por IA**: cada linha vira uma transação com data, valor, estabelecimento, parcela (n/N) e categoria sugerida.
4. **Categorias nascem do histórico**: o app propõe a árvore de categorias (moradia, mercado, restaurantes, saúde, assinaturas, etc.) e tipos de receita. O casal aprova, renomeia ou funde.
5. **Revisão**: fila das linhas de baixa confiança. A correção do usuário vira regra (ex.: "IFOOD" → Restaurantes) e vale para os próximos meses.
6. **Parcelas futuras**: parcelamentos em curso geram os compromissos dos meses seguintes.
7. **Diagnóstico e planejamento**:
   - receita média, despesa média por categoria, fixo versus variável;
   - quanto da receita já está comprometido por mês nos próximos 12 meses;
   - orçamento mensal sugerido por categoria, com base no histórico;
   - sobra (ou falta) projetada.
8. **Tela principal**: disponível no mês, comprometido nos meses seguintes, visão casal e individual.
9. **Lançamento manual simples** (valor, categoria, cartão, parcelas) para o dia a dia. Voz e foto ficam para a Fase 2.

### Cuidados

- **Deduplicação**: o mesmo gasto pode aparecer na fatura, no comprovante e no extrato. A chave é data, valor e estabelecimento. Pagamento de fatura no extrato não conta como despesa (evita contar em dobro).
- **Parcelas**: reconhecer "03/10" e projetar as 7 restantes.
- **Transferências entre contas do casal** não são receita nem despesa.
- **Confirmação humana**: nada entra no orçamento sem passar pelo limiar de confiança ou pela revisão.
- **Privacidade (LGPD)**: arquivos criptografados, acesso restrito ao grupo, exclusão sob pedido. Enviar documentos à IA exige base legal e aviso claro.

### Entrega da Fase 1

- Estrutura do projeto e banco de dados com isolamento por grupo.
- Importação de arquivos e pipeline de leitura.
- Tela de revisão e de categorias.
- Painel com disponível do mês e comprometido futuro.

## Fase 2 — Lançar sem digitar
Voz ("almoço, 85 reais, BTG") e foto da nota fiscal, com leitura automática. A nota fica pendente até casar com o extrato.

## Fase 3 — Conciliação mensal
Importação mensal de fatura e extrato, casamento com lançamentos e notas, tela de diferenças.

## Fase 4 — Poupança e metas
Aposentadoria e viagens anuais: valor, prazo e aporte mensal sugerido, com acompanhamento.

## Fase 5 — IA consultora (plano pago)
Plano de reorganização: cortes, fim de parcelamentos, quanto poupar, simulações de "e se".

## Fase 6 — Abertura ao público
Planos, pagamentos, lojas Apple e Google, LGPD completa. Ordem: familiares, amigos, público.

## Planos

| | Cliente Única (convite) | Básico (grátis) | Avançado (pago) |
|---|---|---|---|
| Lançamentos, cartões, parcelas, visão do casal | Sim | Sim | Sim |
| Importação de PDF, voz e foto | Sim | Com limite mensal | Maior limite |
| Metas de poupança | Sim | Sim | Sim |
| IA consultora | Sim | Não | Sim |

## Tecnologia (proposta)

- **App:** PWA em TypeScript (React), com layout mobile.
- **Backend:** Supabase (Postgres, login, armazenamento) com Row Level Security por grupo.
- **IA:** API da Anthropic (Claude) para ler PDFs e imagens, classificar transações e, depois, gerar sugestões.
- **Custo de IA:** limites por plano e cache de regras aprendidas, para não reclassificar o que já se conhece.

## Perguntas em aberto

1. Quantos meses de histórico você tem à mão (3, 6, 12)?
2. Os PDFs são protegidos por senha? (faturas costumam ser)
3. Prefere que eu construa o app com código aqui no repositório, ou usar o Lovable (construtor com Supabase integrado) para acelerar a Fase 1?
