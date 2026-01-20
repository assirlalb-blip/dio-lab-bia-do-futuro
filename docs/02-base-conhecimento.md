# Base de Conhecimento

## Dados Utilizados


| Arquivo | Formato | Utilização no Agente |
|---------|---------|---------------------|
| `categorias.json` | JSON | Define o taxonomia oficial (categorias/subcategorias), regras de orçamento (teto sugerido por categoria) e palavras-chave para classificar transações por descrição. |
| `stopwords_descricoes.json` | JSON | Lista pequena de palavras irrelevantes/ruído (ex.: “COMPRA”, “PAGAMENTO”, “PIX”, “DEB”) para ajudar na normalização das descrições e melhorar a categorização. |

---

## Adaptações nos Dados

- Criei uma **taxonomia enxuta** de categorias e subcategorias com nomes consistentes.
- Incluí um **dicionário mínimo de palavras-chave** (merchants comuns) para aumentar a precisão sem depender de internet.
- Defini **tetos percentuais sugeridos** por categoria para o agente gerar um orçamento inicial. Esses tetos são apenas orientação de organização (não são recomendação financeira).

---

## Estratégia de Integração

### Como os dados são carregados?
Os arquivos JSON da pasta `data/` são carregados no início da sessão do agente (startup do backend) e ficam em memória:

- `categorias.json` → usado para validar categorias permitidas e mapear keywords → categoria/subcategoria
- `stopwords_descricoes.json` → usado no pré-processamento de texto (normalização)

### Como os dados são usados no prompt?
- O conteúdo de `categorias.json` não precisa ir inteiro no prompt; o backend pode:
  - aplicar regras/keywords primeiro (classificação determinística inicial),
  - e mandar ao LLM apenas:
    1) a lista de categorias permitidas (taxonomia),
    2) e as transações já normalizadas (com a “categoria sugerida” + “confiança”).
- `stopwords_descricoes.json` é usado só na etapa de limpeza; não precisa entrar no prompt.

---

## Exemplo de Contexto Montado

```txt
Taxonomia permitida (categorias -> subcategorias):
- Moradia: [Aluguel/Financiamento, Condomínio, Contas (água/luz/gás), Internet/Telefone, Manutenção]
- Alimentação: [Supermercado, Restaurante, Delivery, Cafés/Lanches]
- Transporte: [Combustível, Transporte por app, Transporte público, Estacionamento, Manutenção]
- Saúde: [Farmácia, Consultas/Exames, Plano de saúde]
- Educação: [Cursos, Mensalidades, Livros]
- Lazer: [Viagens, Eventos, Assinaturas, Hobbies]
- Compras: [Vestuário, Eletrônicos, Casa]
- Financeiro: [Tarifas, Juros, IOF, Anuidade, Impostos]
- Transferências: [PIX/Doc/Ted entre contas]
- Renda: [Salário, Freelance, Reembolso, Outros]
- Outros: [Não classificado]

Transações normalizadas (amostra):
1) 05/01/2026 | "UBER *TRIP" | -R$ 23,40 | sugestão: Transporte > Transporte por app | confiança: Alta
2) 06/01/2026 | "IFOOD" | -R$ 51,90 | sugestão: Alimentação > Delivery | confiança: Alta
3) 07/01/2026 | "TARIFA MENSAL" | -R$ 29,90 | sugestão: Financeiro > Tarifas | confiança: Média

