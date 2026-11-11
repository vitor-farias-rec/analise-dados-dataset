# Atividade 1: tratamento e análise de solicitações de serviços públicos (KNIME)

Relatório da Atividade 1 (prática de Ciência de Dados) do curso **Ciência de Dados, Indicadores e Dashboards**, do Compet Superior, no contexto do projeto Agenda Saúde Digital Tacaratu.

- **Autor:** Vitor Pontes de Farias
- **Ferramenta:** KNIME Analytics Platform
- **Base de dados:** solicitações de serviços públicos (573 linhas e 9 colunas na base bruta)
- **Ano:** 2026

## Resultados em resumo

| Indicador | Resultado |
| --- | --- |
| Registros antes e depois da limpeza | 573 e 560 (13 duplicatas removidas) |
| Solicitações concluídas | 76,61% (429 de 560) |
| Tempo médio de atendimento | 52,53 h |
| Mediana do tempo de atendimento | 39,56 h |
| Desvio-padrão (populacional) | 47,14 h |
| Potenciais outliers (regra de 1,5 × IQR) | 29, todos acima de 126,17 h |
| Serviço mais solicitado (moda) | Buracos nas vias (162 solicitações, 28,9%) |

## Sumário

1. [Etapa 1: Conhecendo os dados](#etapa-1-conhecendo-os-dados)
2. [Etapa 2: Limpeza e preparação dos dados](#etapa-2-limpeza-e-preparação-dos-dados)
3. [Etapa 3: Transformação dos dados](#etapa-3-transformação-dos-dados)
4. [Etapa 4: Análise estatística dos dados](#etapa-4-análise-estatística-dos-dados)

## Arquivos do projeto

- `dados/base_bruta_solicitacoes.csv`: base bruta (573 linhas e 9 colunas).
- `dados/base_tratada_solicitacoes.csv`: base tratada ao final da Etapa 3 (560 linhas e 13 colunas).
- `workflow/atividade_1_compet_superior.knwf`: workflow do KNIME.

---

## Etapa 1: Conhecendo os dados

**Quantas linhas e colunas existem no dataset original?** 573 linhas e 9 colunas.

**Quais são os nomes das variáveis?** `id_solicitacao`, `data_abertura`, `data_fechamento`, `bairro`, `servico`, `status`, `prioridade`, `canal` e `prazo_horas`.

### Tabela 1: diagnóstico inicial da base bruta

| Coluna | Tipo da variável | Registros ausentes | Registros duplicados | Inconsistências |
| --- | --- | --- | --- | --- |
| `id_solicitacao` | Campo de identificação | 0 | 13 | 13 ids duplicados |
| `data_abertura` | Data e hora | 0 | n/a | Registros com `data_abertura` posterior à `data_fechamento` |
| `data_fechamento` | Data e hora | 135 | n/a | Registros com `data_fechamento` anterior à `data_abertura` da mesma linha. Exemplo: em `SOL-0146` (primeira linha do CSV), o fechamento ocorre mais de 1 dia antes da abertura |
| `bairro` | Qualitativa nominal | 26 | n/a | 26 linhas com campos vazios |
| `servico` | Qualitativa nominal | 0 | n/a | Falta de padronização. Exemplo: `LIMPEZA URBANA` em caixa alta, fora do padrão `Limpeza urbana` |
| `status` | Qualitativa nominal | 0 | n/a | Nenhuma |
| `canal` | Qualitativa nominal | 0 | n/a | Nenhuma |
| `prioridade` | Qualitativa ordinal | 14 | n/a | 14 linhas com campos vazios |
| `prazo_horas` | Quantitativa discreta | 0 | n/a | Nenhuma |

*Fonte: elaborado pelo autor a partir da base bruta, 2026.*

A figura a seguir mostra a verificação dessas informações no KNIME.

![Figura 1: verificação das estatísticas da base bruta](imagens/figura-01-estatisticas-base-bruta.png)

*Figura 1: verificação das estatísticas da base bruta. Fonte: elaborado pelo autor no KNIME, 2026.*

O dataset tratado ao final da Etapa 3 está em [`dados/base_tratada_solicitacoes.csv`](dados/base_tratada_solicitacoes.csv).

---

## Etapa 2: Limpeza e preparação dos dados

### Tabela 2: quantidade de problemas corrigidos

| Problema | Qtd. | Tratamento |
| --- | --- | --- |
| Duplicatas | 13 | Removidas, mantendo a linha original da 1ª ocorrência |
| Registros antes e depois da limpeza | 573 e 560 | Base reduzida de 573 para 560 linhas |
| Bairro ausente | 24 | Preenchido com "Não informado" |
| Prioridade ausente | 14 | Preenchida com "Não informada" |
| Fechamento vazio em Aberta ou Em andamento | 131 | Mantido vazio, o que é esperado |
| Sem data de fechamento e com status Aberta ou Em andamento | 0 | Nenhum caso incompatível |
| Nomes de serviço fora do padrão | 26 | Padronizados em 5 categorias |
| Datas | 989 | Já estavam no formato `AAAA-MM-DD HH:MM` e foram convertidas para data e hora sem erros |

*Fonte: elaborado pelo autor a partir da limpeza da base bruta no KNIME, 2026.*

> Os valores da Tabela 2 são contados depois da remoção das duplicatas. Por isso diferem da Tabela 1 em bairro ausente (26 antes, 24 depois) e em fechamento vazio (135 antes, 131 depois).

### Duplicatas

Foram identificadas 13 linhas duplicadas. Elas são excluídas do dataset na limpeza, e a base final fica com 560 linhas (originalmente eram 573). A tabela abaixo mostra cada duplicata ao lado da linha original.

### Tabela 3: registros duplicados e seus originais (base bruta, antes da limpeza)

| RowID | id_solicitacao | data_abertura | data_fechamento | bairro | servico | status | prioridade | canal | prazo_horas | observacao |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Row32 | SOL-0019 | 2026-09-16 02:45 | 2026-10-03 20:05 | Zona Norte | Coleta de lixo | Concluída | Média | Presencial | 36 | Original |
| Row369 | SOL-0019 | 2026-09-16 02:45 | 2026-10-03 20:05 | Zona Norte | Coleta de lixo | Concluída | Média | Presencial | 36 | Duplicata |
| Row384 | SOL-0060 | 2026-08-23 20:58 | 2026-08-28 21:09 | Zona Norte | Drenagem | Concluída | Média | Telefone | 120 | Original |
| Row448 | SOL-0060 | 2026-08-23 20:58 | 2026-08-28 21:09 | Zona Norte | Drenagem | Concluída | Média | Telefone | 120 | Duplicata |
| Row27 | SOL-0071 | 2026-08-14 04:29 | 2026-08-15 03:59 | Zona Norte | Drenagem | Concluída | Média | Presencial | 120 | Original |
| Row516 | SOL-0071 | 2026-08-14 04:29 | 2026-08-15 03:59 | Zona Norte | Drenagem | Concluída | Média | Presencial | 120 | Duplicata |
| Row364 | SOL-0087 | 2026-09-06 07:37 | 2026-09-05 05:37 | Zona Sul | Drenagem | Concluída | Média | Telefone | 120 | Original |
| Row401 | SOL-0087 | 2026-09-06 07:37 | 2026-09-05 05:37 | Zona Sul | Drenagem | Concluída | Média | Telefone | 120 | Duplicata |
| Row172 | SOL-0134 | 2026-09-05 02:11 | 2026-09-13 12:38 | Zona Sul | Limpeza urbana | Concluída | Alta | Presencial | 48 | Original |
| Row210 | SOL-0134 | 2026-09-05 02:11 | 2026-09-13 12:38 | Zona Sul | Limpeza urbana | Concluída | Alta | Presencial | 48 | Duplicata |
| Row372 | SOL-0321 | 2026-10-02 00:09 | 2026-10-03 23:14 | Centro | Limpeza urbana | Concluída | Baixa | Telefone | 48 | Original |
| Row480 | SOL-0321 | 2026-10-02 00:09 | 2026-10-03 23:14 | Centro | Limpeza urbana | Concluída | Baixa | Telefone | 48 | Duplicata |
| Row11 | SOL-0341 | 2026-09-08 21:25 | | Zona Norte | Buracos nas vias | Em andamento | Média | Site | 96 | Original |
| Row84 | SOL-0341 | 2026-09-08 21:25 | | Zona Norte | Buracos nas vias | Em andamento | Média | Site | 96 | Duplicata |
| Row281 | SOL-0349 | 2026-08-20 16:50 | | Centro | Iluminação pública | Em andamento | Média | Presencial | 72 | Original |
| Row416 | SOL-0349 | 2026-08-20 16:50 | | Centro | Iluminação pública | Em andamento | Média | Presencial | 72 | Duplicata |
| Row236 | SOL-0424 | 2026-09-21 07:12 | 2026-09-22 20:22 | Zona Norte | Limpeza urbana | Concluída | Média | Aplicativo | 48 | Original |
| Row510 | SOL-0424 | 2026-09-21 07:12 | 2026-09-22 20:22 | Zona Norte | Limpeza urbana | Concluída | Média | Aplicativo | 48 | Duplicata |
| Row219 | SOL-0466 | 2026-08-18 08:27 | | Zona Sul | Buracos nas vias | Em andamento | Média | Aplicativo | 96 | Original |
| Row549 | SOL-0466 | 2026-08-18 08:27 | | Zona Sul | Buracos nas vias | Em andamento | Média | Aplicativo | 96 | Duplicata |
| Row213 | SOL-0478 | 2026-09-30 18:42 | 2026-10-02 14:18 | Zona Leste | Iluminação pública | Concluída | Alta | Aplicativo | 72 | Original |
| Row456 | SOL-0478 | 2026-09-30 18:42 | 2026-10-02 14:18 | Zona Leste | Iluminação pública | Concluída | Alta | Aplicativo | 72 | Duplicata |
| Row245 | SOL-0485 | 2026-08-12 06:12 | | | Drenagem | Em andamento | Alta | Telefone | 120 | Original |
| Row561 | SOL-0485 | 2026-08-12 06:12 | | | Drenagem | Em andamento | Alta | Telefone | 120 | Duplicata |
| Row432 | SOL-0500 | 2026-08-05 11:38 | 2026-08-08 16:38 | | Limpeza urbana | Concluída | Média | Telefone | 48 | Original |
| Row563 | SOL-0500 | 2026-08-05 11:38 | 2026-08-08 16:38 | | Limpeza urbana | Concluída | Média | Telefone | 48 | Duplicata |

*Fonte: elaborado pelo autor a partir da limpeza da base bruta no KNIME, 2026.*

![Figura 2: filtro de duplicatas com o Rule Engine sobreposto](imagens/figura-02-duplicados-originais.png)

*Figura 2: execução do filtro, com sobreposição do node Rule Engine, para exibir a tabela com registros duplicados e seus originais (base bruta, antes da limpeza). Fonte: elaborado pelo autor no KNIME e editado no Fotor para adição da janela do node Rule Engine, 2026.*

Os campos `bairro` e `prioridade` vazios são preenchidos com "Não informado" e "Não informada" na etapa de transformação (Etapa 3).

### Compatibilidade entre data de fechamento e status

Um Rule Engine verifica se a data de fechamento é compatível com o status. As regras são:

1. `Concluída` sem data de fechamento é incompatível.
2. `Aberta` ou `Em andamento` com data de fechamento é incompatível.
3. Qualquer outro caso é compatível.

![Figura 3A: Rule Engine que verifica a compatibilidade entre fechamento e status](imagens/figura-03a-regra-fechamento-vs-status.png)

*Figura 3A: execução do node Rule Engine para verificar se a data de fechamento é compatível com o status. Fonte: elaborado pelo autor no KNIME, 2026.*

O resultado mostra que todos os registros são compatíveis: 41 `Aberta`, 429 `Concluída` e 90 `Em andamento`.

![Figura 3B: resultado da checagem de compatibilidade](imagens/figura-03b-compatibilidade-fechamento-status.png)

*Figura 3B: execução do node Rule Engine constatando a compatibilidade da data de fechamento com o status. Fonte: elaborado pelo autor no KNIME, 2026.*

### Padronização do campo `servico`

Antes da limpeza, o campo `servico` tinha 10 grafias diferentes para 5 serviços (por exemplo, `BURACOS NAS VIAS` e `Buracos nas vias`).

![Figura 4: padrões de escrita encontrados em servico](imagens/figura-04-padroes-escrita-servico.png)

*Figura 4: contagem e exibição dos diferentes padrões de escrita do campo `servico` (base bruta, antes da limpeza). Fonte: elaborado pelo autor no KNIME, 2026.*

### Tabela 4: registros com data de fechamento anterior à data de abertura (base bruta, antes da limpeza)

| RowID | id_solicitacao | data_abertura | data_fechamento | data_invalida |
| --- | --- | --- | --- | --- |
| Row0 | SOL-0146 | 2026-08-04 20:12 | 2026-08-03 19:12 | Sim |
| Row18 | SOL-0248 | 2026-09-04 09:43 | 2026-09-04 05:43 | Sim |
| Row115 | SOL-0308 | 2026-08-26 17:13 | 2026-08-25 13:13 | Sim |
| Row189 | SOL-0154 | 2026-08-08 20:55 | 2026-08-08 18:55 | Sim |
| Row215 | SOL-0312 | 2026-09-05 07:00 | 2026-09-04 20:00 | Sim |
| Row233 | SOL-0051 | 2026-08-10 23:03 | 2026-08-09 20:03 | Sim |
| Row280 | SOL-0470 | 2026-08-10 08:23 | 2026-08-10 01:23 | Sim |
| Row298 | SOL-0212 | 2026-09-24 14:26 | 2026-09-23 08:26 | Sim |
| Row364 | SOL-0087 | 2026-09-06 07:37 | 2026-09-05 05:37 | Sim |
| Row465 | SOL-0009 | 2026-08-06 00:47 | 2026-08-04 04:47 | Sim |
| Row491 | SOL-0128 | 2026-09-19 17:33 | 2026-09-18 19:33 | Sim |

*Fonte: elaborado pelo autor a partir da base bruta e adaptado para exibir apenas `RowID`, `id_solicitacao`, `data_abertura`, `data_fechamento` e `data_invalida`, 2026.*

A variável `data_invalida` foi criada antecipadamente, nesta etapa, para o leitor constatar as inconsistências de datas sem comparar uma por uma.

A base após a limpeza e a preparação da Etapa 2, com a variável `data_invalida` adicionada, tem 560 linhas e 10 colunas. Ela está em [`dados/base_tratada_solicitacoes.csv`](dados/base_tratada_solicitacoes.csv), junto com as colunas da Etapa 3.

---

## Etapa 3: Transformação dos dados

O fluxo principal da Etapa 3 está na figura abaixo, com os nodes comentados.

![Figura 5: workflow principal da Etapa 3 com nodes comentados](imagens/figura-05-workflow-etapa-3.png)

*Figura 5: workflow principal da Etapa 3 com nodes comentados. Fonte: elaborado pelo autor no KNIME, 2026.*

Os demais nodes, fora do fluxo principal, serviram para checagem. Por isso há vários CSV Writer, usados para exportar as tabelas e conferir cada passo também em planilha. Os passos do fluxo principal estão descritos a seguir.

![Figura 6: Duplicate Row Filter](imagens/figura-06-duplicate-row-filter.png)

*Figura 6: Duplicate Row Filter, para a remoção de linhas duplicadas. Fonte: elaborado pelo autor no KNIME, 2026.*

![Figura 7: String Manipulation (Multi Column)](imagens/figura-07-string-manipulation-multi-column.png)

*Figura 7: String Manipulation (Multi Column), que retira espaços sobrando no início e no fim do texto (`strip`). Fonte: elaborado pelo autor no KNIME, 2026.*

![Figura 8: String Manipulation](imagens/figura-08-string-manipulation.png)

*Figura 8: String Manipulation, que padroniza os nomes de todos os campos da variável `servico` em letra minúscula. Fonte: elaborado pelo autor no KNIME, 2026.*

![Figura 9: Rule Engine com regra de formatação de escrita](imagens/figura-09-rule-engine-padronizacao.png)

*Figura 9: Rule Engine com a regra de formatação de escrita, que troca as variações de escrita por um padrão (por exemplo, `buracos*` vira `Buracos nas vias`). Fonte: elaborado pelo autor no KNIME, 2026.*

![Figura 10: Missing Value](imagens/figura-10-missing-value.png)

*Figura 10: Missing Value, que preenche os campos vazios de `bairro` ("Não informado") e `prioridade` ("Não informada"). Fonte: elaborado pelo autor no KNIME, 2026.*

![Figura 11: String to Date&Time](imagens/figura-11-string-to-datetime.png)

*Figura 11: String to Date&Time, que transforma as datas de texto em data e hora para permitir cálculos. Fonte: elaborado pelo autor no KNIME, 2026.*

![Figura 12: Date&Time Difference](imagens/figura-12-datetime-difference.png)

*Figura 12: Date&Time Difference, que calcula quantos minutos se passaram entre a abertura e o fechamento e cria a coluna `dif_minutos`. Fonte: elaborado pelo autor no KNIME, 2026.*

![Figura 13: Rule Engine que cria a coluna data_invalida](imagens/figura-13-rule-engine-data-invalida.png)

*Figura 13: Rule Engine que cria a coluna `data_invalida`. Ela sinaliza `Sim` quando o fechamento é anterior à abertura e `Não aplicável` nos registros sem `data_fechamento`. Fonte: elaborado pelo autor no KNIME, 2026.*

![Figura 14: Date&Time to String](imagens/figura-14-datetime-to-string.png)

*Figura 14: Date&Time to String, que extrai ano e mês de `data_abertura` em texto e grava na nova coluna `data_abertura_mes`. Fonte: elaborado pelo autor no KNIME, 2026.*

![Figura 15a: Column Renamer](imagens/figura-15a-column-renamer.png)

![Figura 15b: Math Formula](imagens/figura-15b-math-formula.png)

*Figura 15: Column Renamer, que renomeia `data_abertura_mes` para `mes_abertura`, e Math Formula, que converte os minutos de `dif_minutos` em horas (divisão por 60) na coluna `horas_calc`. Fonte: elaborado pelo autor no KNIME, 2026.*

![Figura 16a: Rule Engine que cria tempo_atendimento_horas](imagens/figura-16a-rule-engine-tempo-atendimento.png)

![Figura 16b: Rule Engine que cria no_prazo](imagens/figura-16b-rule-engine-no-prazo.png)

*Figura 16: últimas duas Rule Engines da Etapa 3. A primeira copia as horas apenas dos casos válidos e cria `tempo_atendimento_horas`. A segunda compara esse tempo com `prazo_horas` e cria `no_prazo` (`Sim`, `Não` ou `Não aplicável`). Fonte: elaborado pelo autor no KNIME, 2026.*

Por fim, um node Date&Time to String transforma as colunas de data de abertura e de fechamento de volta para texto, no formato `yyyy-MM-dd HH:mm`.

### Colunas criadas na Etapa 3

| Coluna | Descrição |
| --- | --- |
| `data_invalida` | `Sim` quando o fechamento é anterior à abertura, `Não` caso contrário e `Não aplicável` sem data de fechamento |
| `mes_abertura` | Ano e mês da abertura (`AAAA-MM`) |
| `tempo_atendimento_horas` | Tempo entre abertura e fechamento, em horas, apenas para registros válidos |
| `no_prazo` | `Sim` se o tempo de atendimento é menor ou igual a `prazo_horas`, `Não` se maior e `Não aplicável` sem tempo válido |

O dataset tratado, com 560 linhas e 13 colunas, está em [`dados/base_tratada_solicitacoes.csv`](dados/base_tratada_solicitacoes.csv).

---

## Etapa 4: Análise estatística dos dados

### Parte A: Estatística descritiva

- **Variância populacional:** `Σ(x − média)² ÷ n`
- **Desvio-padrão populacional:** raiz quadrada da variância populacional.

### Tabela 5: medidas descritivas do tempo de atendimento

| Questão | Medida | Resultado | Origem no KNIME |
| --- | --- | --- | --- |
| 1 | Média | 52,53 h | Statistics (Mean) |
| 2 | Mediana | 39,56 h | Box Plot (mediana da visualização) |
| 3 | Menor tempo e maior tempo | 3,10 h e 425,33 h | Statistics (Min e Max) |
| 4 | Amplitude total | 422,23 h | Cálculo: Max − Min, com valores do Statistics |
| 5 | Variância (populacional) | 2222,30 h² | `Σ(x − média)² ÷ n`, com n = 418 pelo Row Filter |
| 6 | Desvio-padrão (populacional) | 47,14 h | Raiz quadrada da variância populacional |
| 7 | Q1 | 23,50 h | Box Plot |
| 8 | Q3 | 64,57 h | Box Plot |
| 9 | IQR | 41,07 h | Cálculo: Q3 − Q1, com valores do Box Plot |
| 10 | Potenciais outliers | 29 valores acima de 126,17 h (limites: −38,10 e 126,17) | Row Filter (`tempo > Q3 + 1,5 × IQR`) e Sorter; limites calculados a partir de Q1, Q3 e IQR |

*Fonte: elaborado pelo autor no KNIME, 2026. Quartis obtidos pelo Box Plot do KNIME.*

![Figura 17: node Statistics com os valores da Parte A](imagens/figura-17-statistics.png)

*Figura 17: execução do node Statistics para apresentar os valores da Parte A. Fonte: elaborado pelo autor no KNIME, 2026.*

Foi pedido usar somente os tempos de atendimento válidos. Por isso, um Row Filter encontra os registros com `tempo_atendimento_horas` preenchido, já que essa coluna só tem valor quando a solicitação foi concluída e a data de fechamento é posterior à de abertura.

![Figura 18: Row Filter para tempo_atendimento_horas](imagens/figura-18-row-filter-tempo-valido.png)

*Figura 18: Row Filter para `tempo_atendimento_horas`. Fonte: elaborado pelo autor no KNIME, 2026.*

São 418 registros válidos. A variância é `Σ(x − média)² ÷ 418`, ou seja, a soma dos quadrados das diferenças entre cada tempo e a média (52,534), dividida pelo número de registros válidos. Como foram solicitadas as fórmulas populacionais, o divisor é n = 418. O resultado, aproximadamente 2222,30 h², está em horas ao quadrado, por isso se calcula em seguida o desvio-padrão, a raiz quadrada da variância: √2222,30 ≈ 47,14 h.

> O node Statistics do KNIME (Figura 17) mostra a variância amostral (divisão por n − 1: 2227,63 h²) e o desvio-padrão correspondente (47,20 h). Os valores da Tabela 5 usam a fórmula populacional, como pedido.

![Figura 19: Box Plot de tempo_atendimento_horas](imagens/figura-19-box-plot.png)

*Figura 19: Box Plot de `tempo_atendimento_horas`. Fonte: elaborado pelo autor no KNIME, 2026.*

Para identificar os potenciais outliers, foi aplicada a regra de 1,5 × IQR. Com Q1 = 23,50 h, Q3 ≈ 64,57 h e IQR = Q3 − Q1 ≈ 41,07 h, calculam-se os limites. Valores abaixo do limite inferior ou acima do limite superior são potenciais outliers.

- Limite inferior = Q1 − 1,5 × IQR = 23,50 − 61,60 ≈ −38,10
- Limite superior = Q3 + 1,5 × IQR = 64,57 + 61,60 ≈ 126,17

Como o menor tempo de atendimento é 3,10 h e tempos de atendimento não podem ser negativos, nenhum valor ficou abaixo do limite inferior. Para o limite superior, foi aplicado no KNIME um Row Filter na coluna `tempo_atendimento_horas`, selecionando os valores maiores que 126,17 h. Foram identificados 29 potenciais outliers, todos acima do limite superior.

![Figura 20: Row Filter aplicado para encontrar outliers](imagens/figura-20-row-filter-outliers.png)

*Figura 20: Row Filter aplicado para encontrar outliers. Fonte: elaborado pelo autor no KNIME, 2026.*

### Parte B: Frequências

![Figura 21: Value Counter da variável servico](imagens/figura-21-value-counter-servico.png)

*Figura 21: Value Counter da variável `servico`, mostrando a quantidade de solicitações e a moda. Fonte: elaborado pelo autor no KNIME, 2026.*

**Moda da variável `servico`:** Buracos nas vias, por ser o valor que mais aparece na coluna, com 162 contagens.

### Tabela 6: frequência de solicitações por serviço

| Serviço | Quantidade | % do total |
| --- | --- | --- |
| Buracos nas vias (moda) | 162 | 28,9% |
| Iluminação pública | 119 | 21,3% |
| Coleta de lixo | 118 | 21,1% |
| Limpeza urbana | 81 | 14,5% |
| Drenagem | 80 | 14,3% |
| **Total** | **560** | **100%** |

*Fonte: elaborado pelo autor após consulta do Value Counter no KNIME, 2026.*

### Tabela 7: frequência por status

| Status | Quantidade | % do total |
| --- | --- | --- |
| Concluída | 429 | 76,61% |
| Em andamento | 90 | 16,07% |
| Aberta | 41 | 7,32% |
| **Total** | **560** | **100%** |

*Fonte: elaborado pelo autor após consulta do Value Counter no KNIME, 2026.*

### Tabela 8: frequência por nível de prioridade

| Prioridade | Quantidade | % do total |
| --- | --- | --- |
| Média | 293 | 52,3% |
| Alta | 136 | 24,3% |
| Baixa | 117 | 20,9% |
| Não informada | 14 | 2,5% |
| **Total** | **560** | **100%** |

*Fonte: elaborado pelo autor após consulta do Value Counter no KNIME, 2026.*

![Figura 22: Math Formula para a porcentagem de solicitações concluídas](imagens/figura-22-math-formula-concluidas.png)

*Figura 22: Math Formula para o cálculo da porcentagem de solicitações concluídas. Fonte: elaborado pelo autor no KNIME, 2026.*

A porcentagem de solicitações concluídas em relação ao total da base limpa é de **76,607%**.

### Parte C: Interpretação estatística

#### 16. O que a média e a mediana revelam sobre os tempos de atendimento?

Pelo histograma da Figura 23, dá para afirmar que, a partir da faixa de 20 a 40 horas, quanto maior o tempo de atendimento, menos atendimentos são realizados. Por isso a distribuição tem uma cauda à direita, o que faz a mediana (valor que divide os dados ao meio) ficar menor que a média. Isso mostra que há alguns casos em que os atendimentos foram mais lentos, ou seja, duraram mais tempo, e isso puxa a média para cima. Assim, é possível inferir que a maior parte dos atendimentos foi rápida e que a média não é um bom parâmetro para representar o tempo normal de atendimento, pois é influenciada pelos outliers.

![Figura 23: histograma de tempo_atendimento_horas](imagens/figura-23-histograma.png)

*Figura 23: histograma de `tempo_atendimento_horas`. Fonte: elaborado pelo autor no KNIME, 2026.*

#### 17. Os tempos apresentam grande ou pequena dispersão? Justifique utilizando o desvio-padrão.

Apresentam grande dispersão. No Scatter Plot da Figura 24, é possível observar que os pontos de cada serviço ficam espalhados, do início do eixo até valores bem altos, e nenhum deles parece ter um tempo fixo. Isso se confirma com o desvio-padrão: como apresentado na Parte A, ele é de 47,14 horas, valor muito próximo da média (52,53 h). Isso significa que os tempos de atendimento estão espalhados em uma grande amplitude, de tempos pequenos até valores muito altos, como 425,3 h. A conclusão é que o tempo que uma pessoa pode esperar para ser atendida é muito pouco previsível, mesmo dentro de um mesmo serviço.

![Figura 24: scatter plot de tempo_atendimento_horas por serviço](imagens/figura-24-scatter-plot.png)

*Figura 24: scatter plot de `tempo_atendimento_horas`. Fonte: elaborado pelo autor no KNIME, 2026.*

#### 18. Foram identificados outliers? Como eles podem afetar a média?

Sim. No Scatter Plot acima está destacado um outlier bem expressivo, com `tempo_atendimento_horas` de 425,3. A maior parte dos pontos fica concentrada perto do início do eixo, com pontos isolados à medida que o tempo de atendimento aumenta, em todos os serviços. Pela regra de 1,5 × IQR (limite superior de 126,17 h), foram identificados 29 outliers, todos acima do limite superior.

Como citado na questão 16, esses atendimentos muito demorados puxam a média para cima. A média é o cálculo de todos os tempos dividido pelo número total de tempos analisados e, por isso, ela (52,5 h) sofre a influência dos outliers: atendimentos como o de 425,3 h pesam bastante no resultado. Assim, poucos casos muito demorados bastam para elevar a média, fazendo com que ela represente mal o tempo da maioria das solicitações.

#### 19. O que o IQR revela sobre a dispersão dos tempos de atendimento?

O IQR revela que metade dos atendimentos fica na faixa entre os quartis Q1 e Q3, sem ser influenciada pelos extremos. Os quartis dividem os tempos, do menor para o maior, em quatro partes iguais: o Q1 indica que 25% dos atendimentos foram concluídos até aquele tempo, e o Q3 indica 75%. O IQR é a diferença entre esses dois quartis (IQR = Q3 − Q1 = 64,57 − 23,5 = 41,07 horas).

No box plot da Figura 19, a altura da caixa representa visualmente esse IQR, e a reta mais marcada dentro dela (a mediana) a divide em duas faixas de largura diferente: a de cima, entre a mediana e o Q3, é cerca de uma vez e meia a de baixo, entre o Q1 e a mediana. Isso confirma que mesmo os dados normais variam bastante. Os pontos acima (outliers) ficam fora dessa faixa, o que reforça que o IQR não é afetado por eles.

#### 20. Por que foi necessário realizar a limpeza antes dos cálculos estatísticos?

A limpeza é necessária porque ela é o filtro que impede que dados duplicados entrem no cálculo da média ou das frequências, o que tornaria essas medidas incorretas. Além disso, a falta de padronização dos nomes na coluna `servico`, as datas de fechamento anteriores às de abertura e as solicitações ainda abertas poderiam fazer, respectivamente, com que o mesmo serviço fosse contado como categorias diferentes, com que tempos de atendimento negativos entrassem nos cálculos e com que solicitações sem conclusão fossem tratadas como se tivessem um tempo de atendimento.

É importante ter um dataset tratado e com todos os dados corretos, porque isso também influencia a tomada de decisão que essa base trará. No caso do projeto de Tacaratu, por exemplo, afetaria diretamente a qualidade e a entrega do serviço à população.
