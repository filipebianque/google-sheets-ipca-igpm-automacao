# Automatize IPCA e IGP-M no Google Sheets

**E se eu te dissesse que dá para importar automaticamente IPCA e IGP-M para o Google Sheets?**  
Precisei disso para atualizar os reajustes da minha planilha de **contratos de TI**. Cada contrato vence em um mês e pode usar **índices diferentes** — automatizar economiza tempo, reduz erro e mantém tudo **atualizado**.

---

## 1) Importar a série oficial (SGS/BCB) — célula `A1` da nova aba

**IPCA (cód. 433):**
```
=IMPORTDATA("https://api.bcb.gov.br/dados/serie/bcdata.sgs.433/dados?formato=csv"
&"&dataInicial="&TEXT(DATE(1994;7;1);"dd/mm/yyyy")
&"&dataFinal="&TEXT(EOMONTH(TODAY();-1);"dd/mm/yyyy"))
```

**IGP-M (cód. 189):** troque **433** por **189**.

> Em planilhas com localidade Brasil, use ponto‑e‑vírgula `;` como separador. Se a sua localidade for EUA, troque por vírgula `,` e ajuste decimais.

---

## 2) Converter em colunas úteis (mês e percentual)

Na mesma aba, crie os cabeçalhos:
- `C1 = data`
- `D1 = valor (texto)`
- `E1 = data_ok`
- `F1 = valor_ok`

**C2 – data (texto, sem aspas)**
```
=ARRAYFORMULA(IF(A2:A="";;SUBSTITUTE(REGEXEXTRACT(A2:A;"^[^;]+");CHAR(34);"")))
```

**D2 – valor bruto (junta A+B e remove aspas)**
```
=ARRAYFORMULA(IF(A2:A="";;SUBSTITUTE(REGEXREPLACE(A2:A&" "&B2:B;"^.*;";"");CHAR(34);"")))
```

**E2 – data_ok (data real)**
```
=ARRAYFORMULA(IF(C2:C="";;IF(REGEXMATCH(TO_TEXT(C2:C);"/");DATE(VALUE(RIGHT(C2:C;4));VALUE(MID(C2:C;4;2));VALUE(LEFT(C2:C;2)));TO_DATE(C2:C))))
```

**F2 – valor_ok (número; troca espaço/NBSP por vírgula)**
```
=ARRAYFORMULA(IF(D2:D="";;VALUE(SUBSTITUTE(SUBSTITUTE(TRIM(D2:D);" ";",");CHAR(160);","))))
```

**Limpeza visual:** oculte **A, B, D, E** e trabalhe só com **C (mês)** e **F (% mensal)**.

---

## 3) Acumulado entre duas datas (para reajuste)

Na aba de contratos (ex.: `D2 = data inicial`, `E2 = data final`), usando a aba **IPCA**:
```
=IFERROR(
  PRODUCT(1 + FILTER(IPCA!$F$2:$F;
                     (IPCA!$E$2:$E>=EOMONTH($D2;-1)+1)*
                     (IPCA!$E$2:$E<=EOMONTH($E2;-1)+1))/100) - 1; 0)
```
> Para **IGP-M**, troque `IPCA` por `IGPM`. Resultado em decimal (ex.: `0,0723` = **7,23%**).

---

## Observações
- O BCB/SGS publica o IPCA (IBGE) e replica o IGP-M (FGV). O mês corrente aparece após a divulgação oficial.
- Valide sempre com as fontes oficiais antes de usar em produção.
- Melhorias e PRs são bem-vindos!

## Licença
MIT — veja o arquivo `LICENSE`.
