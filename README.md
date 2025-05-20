# 📝 Relatório sobre o Funcionamento do Circuito e Implementação das Instruções – Projeto MIPS Pipeline

## 1. Introdução

Este relatório descreve a arquitetura, o funcionamento e as extensões feitas no circuito MIPS pipeline fornecido, conforme especificações do documento “Processador Didático Pipeline”. O objetivo do trabalho foi expandir o processador base com suporte a novas instruções e implementar mecanismos de *forwarding* e *stalling* para tratamento de hazards.

---

## 2. Estrutura Geral do Circuito

O circuito foi estruturado segundo o modelo de pipeline clássico de 5 estágios, com os seguintes componentes principais:

* **Banco de Registradores (BR)**: Armazena temporariamente valores intermediários e operacionais.
* **Memória de Dados (MD)**: Usada para leitura e escrita via instruções `LW` e `SW`.
* **ULA (Unidade Lógica e Aritmética)**: Executa as operações matemáticas e lógicas.
* **Program Counter (PC)**: Controla o fluxo sequencial ou desviado do programa.
* **Unidade de Controle (UC\_W/UC)**: Decodifica a instrução e gera os sinais de controle.
* **Forwarding Unit**: Permite que resultados intermediários sejam reutilizados antes de serem gravados no banco de registradores.
* **Hazard Detection Unit**: Detecta conflitos de dados e implementa *stalling*.

O circuito foi expandido para lidar corretamente com os tipos R, I e J de instruções.

---

## 3. Instruções Implementadas

Foram adicionadas nove instruções ao processador, com suporte completo à detecção de hazards e alteração do fluxo de controle:

---

### 3.1. **AND** (Tipo R)

* **Operação**: `BR[rd] = BR[rs] AND BR[rt]`
* **Implementação**: Adicionada à ULA como operação lógica bit a bit, com decodificação via campo `funct`.

---

### 3.2. **OR** (Tipo R)

* **Operação**: `BR[rd] = BR[rs] OR BR[rt]`
* **Implementação**: Operação incluída na ULA com seleção via `funct`.

---

### 3.3. **BNE** (Tipo I)

* **Operação**: `if (BR[rs] ≠ BR[rt]) PC = PC + Imm`
* **Implementação**:

  * Campo de controle `BNESelect` criado para diferenciar `BEQ` e `BNE`.
  * Lógica combinacional com `Zero XOR BNESelect` define se o desvio é tomado.
  * Adaptação do MUX do PC para incluir essa lógica de salto.

---

### 3.4. **SLTI** (Tipo I)

* **Operação**: `BR[rt] = (BR[rs] < Imm) ? 1 : 0`
* **Implementação**: Inserida na ULA com suporte à comparação imediata, utilizando o campo de opcode.

---

### 3.5. **SLT** (Tipo R)

* **Operação**: `BR[rd] = (BR[rs] < BR[rt]) ? 1 : 0`
* **Implementação**: Adicionada na ULA como operação comparativa.

---

### 3.6. **JR** (Tipo R)

* **Operação**: `PC = BR[rs]`
* **Implementação**:

  * Detectada por opcode `000000` e funct `001000`.
  * Valor de `BR[rs]` (saída do banco) é conectado diretamente ao MUX do PC (entrada 3).
  * Controle por sinal `JR`, que tem prioridade máxima no seletor `PCSrc`.

---

### 3.7. **JAL** (Tipo J)

* **Operação**: `BR[31] = PC + 4; PC = label`
* **Implementação**:

  * `PC + 4` é salvo em `$ra` no estágio ID.
  * `JumpAddr` montado com `{PC[31:28], instruction[25:0], 00}`.
  * Utiliza o mesmo caminho de `J`, com sinal adicional de escrita no registrador 31.

---

### 3.8. **SLL** (Tipo R)

* **Operação**: `BR[rd] = BR[rt] << shamt`
* **Implementação**:

  * Lógica de deslocamento à esquerda adicionada à ULA.
  * Campo `shamt` extraído da instrução e usado diretamente.

---

### 3.9. **SLR** (Tipo R)

* **Operação**: `BR[rd] = BR[rt] >> shamt`
* **Implementação**:

  * Similar ao `SLL`, mas com deslocamento à direita na ULA.

---

## 4. Forwarding e Stalling

### 4.1. **Forwarding Unit**

* **Objetivo**: Evitar stalls desnecessários repassando resultados de EX/MEM ou MEM/WB para as entradas da ULA.
* **Implementação**:

  * Comparações entre `RegisterRd` de EX/MEM e MEM/WB com os registradores `rs`/`rt` da ID/EX.
  * Multiplexadores nas entradas da ULA selecionam dinamicamente a fonte correta dos operandos.

---

### 4.2. **Hazard Detection Unit (Stalling)**

* **Objetivo**: Inserir bolhas no pipeline quando o resultado de `LW` ainda não está disponível.
* **Lógica**:

  * Se `ID/EX.MemRead = 1` e `ID/EX.RegisterRt == IF/ID.RegisterRs ou Rt` → **STALL**
* **Ações em STALL**:

  * `PCWrite = 0`, `IF/IDWrite = 0`
  * Inserção de NOP nos sinais de controle do ID/EX

---

## 5. Conclusão

A expansão do processador MIPS pipeline contemplou nove novas instruções e técnicas de forwarding e stalling. O circuito tornou-se significativamente mais robusto, suportando desvios complexos, operações lógicas e comparações. Com isso, o processador é capaz de executar programas mais realistas e diversos, respeitando o comportamento esperado de uma arquitetura MIPS funcional e eficiente.

---

