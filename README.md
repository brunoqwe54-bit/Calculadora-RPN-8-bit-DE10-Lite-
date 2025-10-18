# Calculadora RPN 8-bit (DE10-Lite)

Projeto de **calculadora/ULA** em **Verilog estrutural puro** (nível de portas), pensado para a placa **DE10-Lite (MAX10)**.  
Sem `always`, sem `for/generate` em lógica de dados (apenas flip-flops básicos). A interface usa os **SW**, **KEY**, **LEDs** e **HEX** da placa.

> **Top-level:** `alc8_top_de10lite_v3`

## O que faz

- Pilha **4×8 bits** (estilo RPN) com *push* e *commit*.
- Operações binárias: **ADD, SUB, AND, OR, XOR**  
  – ADD tem **saturação** (255 quando há carry).  
- Operação unária: **NOT**.  
- Operações sequenciais: **MUL** (8×8) e **DIV** (8/8 sem sinal), **POW** (base^expoente via loop de multiplicação).
- Exibição em **HEX / OCT / DEC** nos 4 displays.
- *Flags* e status:
  - `LEDR[3]` carry (latch de ADD)
  - `LEDR[4]` overflow (latch + pulso em ADD/SUB, e saturação em MUL/POW)
  - `LEDR[5]` zero da última resposta
  - `LEDR[6]` erro geral (underflow de operandos, tentativa de div/0, etc.)
  - `LEDR[7]` mostra **EEE** nos displays (latch para **divisão por zero** ou **SUB negativa**)
  - `LEDR[8]` indica se o topo da pilha (S0) é válido
  - `LEDR[9]` **busy** (MUL/DIV/POW em execução)

## Controles (DE10-Lite)

- **KEY0** (ativo em 0): *ENTER* → faz **PUSH** do valor dos `SW[7:0]` para S0 (quando `SW9=0`) **ou** confirma operação (quando `SW6` for ativado; ver abaixo).
- **KEY1** (ativo em 0): cicla a **operação** (mostrada em `LEDR[2:0]`):
  - `000` ADD, `001` SUB, `010` MUL, `011` DIV, `100` AND, `101` OR, `110` XOR, `111` NOT
- **SW8**: **RESET** (em nível alto) → limpa pilha, flags e displays.
- **SW9**: **VIEW** (1 = ver pilha, 0 = editor).  
  - `VIEW=0`: `SW[7:0]` é o **editor** (valor para PUSH).  
  - `VIEW=1`: olha a pilha (S0/S1/S2/S3) conforme `SW[1:0]`.
- **SW6**: **EXEC** (borda 0→1) → tenta executar a operação selecionada.
- **SW5:4**: base de exibição: `00=HEX`, `01=OCT`, `10=DEC`, `11=HEX`.
- **SW3**: quando **MUL** está selecionado (`010`), `SW3=1` muda para **POW** (base^expoente).
- **SW1:0**: índice de **PEEK** quando `VIEW=1`:
  - `00=S0`, `01=S1`, `10=S2`, `11=S3`.

### Fluxo típico
1. Coloque `SW9=0` (editor) e ajuste `SW[7:0]` para o primeiro operando, pressione **KEY0** (*PUSH*).  
2. Ajuste `SW[7:0]` para o segundo operando, **KEY0** novamente.  
3. Com **KEY1** escolha a operação (olhe `LEDR[2:0]`).  
4. Dê um pulso em **SW6** (0→1) para **EXEC**. Para operações sequenciais, `LEDR[9]=1` até terminar.  
5. Use `SW9=1` + `SW1:0` para **PEEK** na pilha.

> **Erros exibidos**: “EEE” acende quando ocorre **DIV/0** ou quando uma **SUB dá negativa** (s1 < s0 no momento do commit).  
> Limpa ao fazer **PUSH** sem erro.

## Arquivos principais

- `rtl/alc8_top_de10lite_v3.v` — **Todo o projeto** (módulos estruturais e topo).
- `rtl/...` (opcional) — se preferir quebrar em arquivos, manter os nomes dos módulos.
- `README.md`, `.gitignore`, `LICENSE`.

## Como sintetizar (Quartus Prime Lite)

1. **New Project Wizard** → aponte para a pasta do repo.  
2. **Add Files** → adicione `rtl/alc8_top_de10lite_v3.v`.  
3. **Top-level entity**: `alc8_top_de10lite_v3`.  
4. **Device**: família **MAX10** (DE10-Lite).  
5. **Pin Planner**: associe `CLK_50`, `SW[9:0]`, `KEY[1:0]`, `LEDR[9:0]`, `HEX0..HEX3` conforme o manual da DE10-Lite (ou importe um `.qsf` pronto).  
6. **Compile**.

> Importante: os nomes internos de portas lógicas (`g_and8`, `g_or8`, etc.) foram **renomeados** para evitar colisão com megafunções do Quartus.

## Organização sugerida

