# Teste de Mutação com Mull (C/C++)

Este tutorial apresenta o uso da ferramenta **Mull** para **Teste de Mutação** em aplicações C/C++, abordando desde a instalação até a análise dos relatórios de mutação. O conteúdo foi desenvolvido no contexto da disciplina de **Noções de Automação de Testes**, seguindo boas práticas de engenharia de software e testes.

---

## 📌 O que é Teste de Mutação?

O **Teste de Mutação** é uma técnica avançada de teste de software que avalia a **qualidade dos testes** existentes.  
A ideia central é introduzir **pequenas alterações artificiais (mutantes)** no código-fonte e verificar se os testes são capazes de detectar essas falhas.

- Se o teste falhar → **mutante morto**
- Se o teste passar → **mutante vivo**

Quanto maior a quantidade de mutantes mortos, **melhor a qualidade da suíte de testes**.

---

## 🛠️ Ferramenta Utilizada: Mull

O **Mull** é uma ferramenta de teste de mutação para **C e C++**, baseada na infraestrutura do **LLVM**.  
Ele funciona como um **plugin do compilador Clang**, aplicando mutações diretamente no **LLVM IR**, sem gerar arquivos-fonte mutados.

- Site oficial: https://github.com/mull-project/mull
- Documentação: https://mull.readthedocs.io

---

## ⚙️ Ambiente

- Sistema Operacional: **Ubuntu 22.04**
- Compilador: **Clang 19**
- LLVM: **19.x**
- Mull: **0.27.1**

---
## 📥 Instalação do Mull

Abra o terminal e execute:

```bash
curl -1sLf 'https://dl.cloudsmith.io/public/mull-project/mull-stable/setup.deb.sh' | sudo -E bash
sudo apt-get update
sudo apt-get install mull-19
```

## Verificação da instalação
Execute o comando abaixo para verificar se o Mull foi instalado corretamente:

```bash
mull-runner-19 -version
```
## Saída esperada:

```text
Mull: Practical mutation testing and fault injection for C and C++
Version: 0.27.1
LLVM: 19.1.1
```
---
## 🔧 Sobre o Funcionamento do Mull

O **Mull** é disponibilizado como um *plugin do compilador*, estando diretamente vinculado a versões específicas do **Clang/LLVM**.

Por esse motivo, todas as ferramentas possuem um **sufixo de versão**, como por exemplo:

- `mull-runner-19`

---
## 🚀 Primeiro Exemplo: Hello World com Mull

Antes de utilizarmos o **Mull** em um projeto real, vamos construir um **exemplo mínimo (Hello World)** para compreender como a ferramenta funciona e por que a simples execução não gera mutantes automaticamente.

### 📌 Conceito Importante

O **Mull** é disponibilizado como um *plugin do compilador* e, por esse motivo:

- É fortemente acoplado a versões específicas do **Clang** e do **LLVM**
- Seus binários e plugins possuem um **sufixo com a versão**, como por exemplo:
  - `clang-19`
  - `mull-runner-19`
  - `mull-ir-frontend-19`

👉 Neste tutorial, assumimos o uso do **Clang 19** e que o **Mull já foi instalado corretamente** no sistema.

---
## 🧩 Criando o código Hello World

Crie um arquivo chamado `main.cpp` com o seguinte conteúdo:

```cpp
int main() {
    return 0;
}
```
Esse é um programa válido em C++ que não contém nenhuma lógica passível de mutação.

### 🔧 Compilação sem o plugin do Mull

Abra o terminal no diretório onde está o arquivo `main.cpp` e execute:

```bash
clang-19 main.cpp -o main
```
### ❗ Possível erro: Clang não instalado
Caso você encontre o erro:

```text
Comando 'clang-19' não encontrado
```
Instale o compilador com:

```bash
sudo apt install clang-19
```
Após a instalação, repita o comando de compilação.

### ▶️ Execução do Mull (sem plugin)

Agora execute o Mull sobre o binário gerado:

```bash
mull-runner-19 ./main
```

### 📤 Saída esperada

```text
[warning] Could not find dynamic library: libc.so.6
[info] Warm up run (threads: 1)
       [################################] 1/1. Finished in 4ms
[info] Filter mutants (threads: 1)
       [################################] 1/1. Finished in 0ms
[info] Baseline run (threads: 1)
       [################################] 1/1. Finished in 3ms
[info] No mutants found. Mutation score: infinitely high
[info] Total execution time: 20ms
```
### 📌 Interpretação do resultado

A mensagem:


```text
No mutants found. Mutation score: infinitely high
```

indica que:

- `O Mull está funcionando corretamente`
- `Porém, nenhum mutante foi gerado`

⚠️ Isso ocorre porque o programa não foi compilado com o plugin do Mull, ou seja, nenhuma mutação foi inserida no binário.


### 🔌 Compilando com o plugin do Mull

Para que o Mull consiga gerar mutantes, é necessário instrumentar o código durante a compilação, utilizando o plugin mull-ir-frontend.

Execute o seguinte comando:

```bash
clang-19 \
  -fpass-plugin=/usr/lib/mull-ir-frontend-19 \
  -g -O0 \
  -grecord-command-line \
  main.cpp -o main
```
⚠️ Aviso comum

Durante a compilação, pode aparecer o aviso:

```text
[warning] Mull cannot find config (mull.yml). Using some defaults.
```
✔️ Esse aviso não é um erro.

✔️ Neste tutorial, os valores padrão do Mull são suficientes e o aviso pode ser ignorado.

### ▶️ Executando novamente o Mull

Agora execute novamente:


```bash
mull-runner-19 ./main
```

### 📤 Saída esperada

```text
[warning] Could not find dynamic library: libc.so.6
[info] Warm up run (threads: 1)
       [################################] 1/1. Finished in 6ms
[info] Filter mutants (threads: 1)
       [################################] 1/1. Finished in 0ms
[info] Baseline run (threads: 1)
       [################################] 1/1. Finished in 4ms
[info] No mutants found. Mutation score: infinitely high
[info] Total execution time: 11ms
```

---
### 🧠 Conclusão do Hello World

Mesmo após a compilação com o plugin do Mull, ainda não existem mutantes, e isso é esperado.

📌 Motivo:
- `O código não possui operações aritméticas, condições ou decisões lógicas`
- `Portanto, não há pontos de mutação aplicáveis`

✅ O mais importante é que, neste ponto:
- `O Mull está corretamente instalado`
- `O plugin está funcionando`
- `O fluxo de compilação e execução está correto`

### ➡️ Próximo passo

Com toda a infraestrutura configurada, o próximo passo será demonstrar o Mull em um cenário real, utilizando:

### 🧮 Exemplo Prático: Calculadora Simples

Este projeto demonstra o Mull em um cenário real, seguindo boas práticas de separação entre código de produção e código de teste.

Estrutura do Projeto

```text
calc/
├── calc.h
├── calc.cpp
└── test_calc.cpp
```

- `Código de produção`
- `Código de teste separado`
- `Operações mutáveis`
- `Avaliação do Mutation Score`

👉 No próximo exemplo, será apresentada uma calculadora simples, onde o Mull efetivamente gera e executa mutantes.

### calc.h (Interface)

```cpp
#ifndef CALC_H
#define CALC_H

int soma(int a, int b);
int subtracao(int a, int b);
int multiplicacao(int a, int b);
int divisao(int a, int b);

#endif
```

### calc.cpp (Implementação)

```cpp
#include "calc.h"

int soma(int a, int b) {
    return a + b;
}

int subtracao(int a, int b) {
    return a - b;
}

int multiplicacao(int a, int b) {
    if (a == 0 || b == 0) {
        return 0;
    }
    return a * b;
}

int divisao(int a, int b) {
    if (b == 0) {
        return 0;
    }
    return a / b;
}

```

test_calc.cpp (Testes)

```cpp

#include <cassert>
#include <iostream>
#include "calc.h"

void test_soma() {
    assert(soma(2, 3) == 5);
    assert(soma(-1, 1) == 0);
}

void test_subtracao() {
    assert(subtracao(5, 3) == 2);
    assert(subtracao(3, 5) == -2);
}

void test_multiplicacao() {
    assert(multiplicacao(3, 4) == 12);
    assert(multiplicacao(0, 5) == 0);
}

void test_divisao() {
    assert(divisao(10, 2) == 5);
    assert(divisao(10, 1) == 10);
}

int main() {
    test_soma();
    test_subtracao();
    test_multiplicacao();
    test_divisao();

    std::cout << "Todos os testes passaram com sucesso!" << std::endl;
    return 0;
}
```

### ⚠️ Observação Importante sobre a Flag -grecord-command-line

Para que o Mull funcione corretamente, ele precisa ter acesso às flags de compilação utilizadas em cada unidade de compilação. Para isso, utiliza-se a flag:

```bash
-grecord-command-line
```

No entanto, essa flag não funciona corretamente quando múltiplos arquivos fonte são compilados em um único comando.

### ❌ Exemplo de Compilação Incorreta

O comando abaixo não deve ser utilizado em projetos com mais de um arquivo .cpp:

```bash
clang-19 \
  -fpass-plugin=/usr/lib/mull-ir-frontend-19 \
  -g -O0 -grecord-command-line \
  calc.cpp test_calc.cpp \
  -o test_calc
```

### 🚫 Erro esperado

Ao executar esse comando, você poderá encontrar erros como:

```text
clang-19: error: linker command failed with exit code 1 (use -v to see invocation)
```

### 🔍 Por que esse erro acontece?

Quando múltiplos arquivos fonte são passados em um único comando:
- `O Clang executa múltiplos compiler jobs internos`
- `Cada arquivo é tratado como uma unidade de compilação separada`
- `O Mull não consegue identificar corretamente as flags associadas a cada unidade`
- `Como consequência, o processo de instrumentação falha`

📌 Por esse motivo, a própria documentação do Mull recomenda:

```text
Remover a flag -grecord-command-line ou realizar a compilação em etapas separadas.
```

### ✅ Solução Adotada Neste Tutorial

A solução adotada neste tutorial é:

```text
Compilar cada arquivo individualmente com o plugin do Mull e realizar a linkedição final sem o plugin.
```

Essa abordagem garante que:

- `Cada unidade de compilação seja corretamente instrumentada`
- `O Mull consiga gerar e executar mutantes`
- `O binário final funcione corretamente`
  
### 🔨 Compilação Correta do Projeto

Abra o terminal na pasta do projeto e execute os comandos abaixo.

### 📦 Compilação do arquivo calc.cpp

```bash
clang++-19 \
  -fpass-plugin=/usr/lib/mull-ir-frontend-19 \
  -g -O0 \
  -c calc.cpp
```

📦 Compilação do arquivo test_calc.cpp

```bash
clang++-19 \
  -fpass-plugin=/usr/lib/mull-ir-frontend-19 \
  -g -O0 \
  -c test_calc.cpp
```

### 🔗 Linkedição final (sem o plugin do Mull)

```bash
clang++-19 calc.o test_calc.o -o test_calc
```

### ▶️ Execução do Teste de Mutação

Após a compilação e linkedição corretas, execute o Mull:

```bash
mull-runner-19 ./test_calc
```

### 📊 Saída Esperada

```text
[info] Warm up run (threads: 1)
       [################################] 1/1. Finished in 14ms
[info] Filter mutants (threads: 1)
       [################################] 1/1. Finished in 0ms
[info] Baseline run (threads: 1)
       [################################] 1/1. Finished in 12ms
[info] Running mutants (threads: 4)
       [################################] 7/7. Finished in 489ms
[info] All mutations have been killed
[info] Mutation score: 100%
[info] Total execution time: 517ms
```

### 🧠 Interpretação do Resultado

- `7 mutantes foram gerados`
- `Todos os mutantes foram mortos pelos testes`
- `O Mutation Score é de 100%, indicando excelente qualidade dos testes`
  
### ⚠️ Nota sobre Avisos de Bibliotecas Dinâmicas

Durante a execução do mull-runner, podem ser exibidos avisos como:

```text
[warning] Could not find dynamic library: libstdc++.so.6
[warning] Could not find dynamic library: libm.so.6
[warning] Could not find dynamic library: libgcc_s.so.1
[warning] Could not find dynamic library: libc.so.6
```

### 📌 Importante

- `Esses avisos não representam erro`
- `Ocorrem porque o Mull analisa o binário em um ambiente isolado`
- `Algumas bibliotecas dinâmicas do sistema não são localizadas explicitamente`
- `Não compromete a execução dos testes nem o cálculo do Mutation Score`

## 🔁 Operadores de Mutação Suportados pelo Mull

O Mull implementa o teste de mutação por meio de **mutadores específicos**, que representam variações semânticas aplicadas ao código-fonte em nível de LLVM IR.

Do ponto de vista conceitual, esses mutadores podem ser classificados de acordo com categorias clássicas da literatura de Teste de Software, como AOR, ROR e LOR. No entanto, **o Mull utiliza nomes explícitos para cada operador**, conforme descrito em sua documentação oficial.

### 📚 Classificação Conceitual (Literatura)

Essas categorias são amplamente utilizadas para explicar o conceito de mutação:

- **AOR (Arithmetic Operator Replacement)**  
  Substituição de operadores aritméticos  
  Ex: `+ → -`, `* → /`

- **ROR (Relational Operator Replacement)**  
  Substituição de operadores relacionais  
  Ex: `== → !=`, `> → >=`

- **LOR (Logical Operator Replacement)**  
  Substituição de operadores lógicos  
  Ex: `&& → ||`, remoção de negação

- **Unary Operator Mutations**  
  Alteração de operadores unários  
  Ex: `++ → --`, `!x → x`

- **Constant Replacement**  
  Substituição de valores constantes  
  Ex: `a = b → a = 42`

---

### ⚙️ Operadores Reais Implementados pelo Mull

Na prática, o Mull utiliza mutadores com nomes explícitos. Alguns exemplos:

#### 🔢 Operadores Aritméticos
- `cxx_add_to_sub` — `+ → -`
- `cxx_sub_to_add` — `- → +`
- `cxx_mul_to_div` — `* → /`
- `cxx_div_to_mul` — `/ → *`
- `cxx_rem_to_div` — `% → /`

#### 🔍 Operadores Relacionais
- `cxx_eq_to_ne` — `== → !=`
- `cxx_ne_to_eq` — `!= → ==`
- `cxx_gt_to_ge` — `> → >=`
- `cxx_lt_to_le` — `< → <=`
- `cxx_ge_to_lt` — `>= → <`
- `cxx_le_to_gt` — `<= → >`

#### 🔗 Operadores Lógicos e Unários
- `cxx_remove_negation` — `!x → x`
- `negate_mutator` — negação de condicionais
- `cxx_pre_inc_to_pre_dec` — `++x → --x`
- `cxx_post_inc_to_post_dec` — `x++ → x--`

#### 🔁 Substituição de Constantes
- `cxx_assign_const` — `a = b → a = 42`
- `cxx_init_const` — `T a = b → T a = 42`

---

### 🧩 Grupos de Mutadores

O Mull organiza os mutadores em grupos, facilitando a configuração:

- `cxx_arithmetic`
- `cxx_comparison`
- `cxx_boundary`
- `cxx_increment`
- `cxx_decrement`
- `cxx_bitwise`
- `cxx_calls`
- `cxx_default` (grupo padrão)

---

### 🎯 Seleção de Operadores de Mutação

É possível selecionar quais mutadores serão aplicados criando um arquivo `mull.yml` na pasta raiz do projeto:

```yaml
mutators:
  - cxx_arithmetic
  - cxx_comparison
  - cxx_remove_negation
```

### 📄 Armazenamento dos Mutantes

O Mull não gera arquivos-fonte mutados.
Os mutantes são criados em memória, diretamente no LLVM IR, durante a execução.

### 📊 Relatórios e Saídas

O Mull fornece:

- Saída textual no terminal

- Informações sobre:

  - Total de mutantes

  - Mutantes mortos

  - Mutantes vivos

  - Tempo de execução

  - Mutation Score

Também é possível gerar relatórios em JSON e outros formatos, permitindo integração com CI/CD e análise posterior em planilhas.


### ▶️ Comando para gerar relatório em JSON

Execute o comando abaixo na raiz do projeto, para criar um arquivo json contedo informações do teste:

```bash
mull-runner-19 ./test_calc \
  --reporters=Elements \
  --report-dir=. \
  --report-name=mutation_report.json

```

### ☠️ Mutantes Mortos e Vivos

- Mutante morto: o teste detectou a falha

- Mutante vivo: o teste não detectou a falha

### 🧩 Categorias de Mutantes

O Mull pode classificar mutantes como:

- Mortos

- Vivos 

- Timeout

- Crash

### 📈 Mutation Score

O Mutation Score é calculado como:

```text
Mutantes Mortos / Total de Mutantes
```

### ⚠️ Avisos sobre Bibliotecas Dinâmicas

Avisos como:

```text
Could not find dynamic library: libstdc++.so.6
```
são esperados e não comprometem a execução dos testes.

### 🆘 Ajuda e Opções do Mull (`--help`)

O Mull disponibiliza um comando de ajuda que lista todas as opções disponíveis para execução dos testes de mutação.

Para visualizar todas as opções disponíveis do `mull-runner`, execute:

```bash
mull-runner-19 --help

```
