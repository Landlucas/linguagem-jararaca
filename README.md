# Linguagem Jararaca #

## 1 Introdução ##

Este manual de referência descreve a linguagem de programação Jararaca.

### 1.1 Motivação e Contexto ###

A gramática da linguagem Jararaca foi elaborada por Lucas Land ao receber o desafio de criação de uma gramática na disciplina de **Linguagens Formais e Autômatos (202001), no curso de Ciência da Computação da Universidade Feevale**.

Fortemente baseada na popular linguagem Python, a Jararaca busca os mesmos princípios: Seja você um iniciante em programação ou um desenvolvedor experiente, é fácil aprender e usar a Jararaca. Mas isso com uma grande diferença: uma linguagem **100% brasileira**.

O Python é usado com sucesso em milhares de aplicativos de negócios do mundo real em todo o planeta, incluindo muitos sistemas complexos e de importância crítica. A Jararaca busca seguir esse sucesso.

### 1.2 Notação ###

As definições léxica e de sintaxe da Jararaca usam uma notação gramatical modificada do Formalismo de Backus-Naur (BNF ou FNB). Com isso, temos o seguinte estilo de definição:

```
<nome>  ::= <letra> (<letra> | "_")*
<letra> ::=  "a"..."z"
```

Onde `<nome>` é um não terminal, e a expressão consiste em sequências de `<letra>`s com caracteres únicos de `'a'` a `'z'` e/ou sequências de `_` separadas pela barra vertical, `|`, indicando uma escolha. Já o símbolo `*` significa zero ou mais repetições do item anterior.

Esta notação indica as possibilidades de substituição para símbolo da esquerda. Símbolos que nunca aparecem no lado esquerdo são terminais.

Além dessas, existem mais regras aplicadas na expressão acima:
- Parênteses são usados ​​para agrupar;
- As cadeias de caracteres literais são colocadas entre aspas;
- As regras normalmente estão contidas em uma única linha (explicado em detalhe em seguida);

Apesar da notação usada ser quase a mesma, há uma grande diferença entre o significado das definições lexicais e sintáticas: a léxica opera nos caracteres individuais da fonte de entrada, enquanto a análise de sintaxe opera na sequência de caracteres gerados pela léxica.

## 2 Formatação e Análise Léxica ##

O código desenvolvido em Jararaca é lido primeiramente por um analisador léxico. Ele faz a varredura do programa fonte caractere por caractere e traduz em uma sequência de símbolos léxicos ou tokens. Nos subcapítulos é descrito como o analisador léxico divide um arquivo nesses tokens.

A Jararaca lê o texto do programa no formato Unicode; a codificação de um arquivo de origem pode ser fornecida por uma declaração de codificação e o padrão é UTF-8. Caso o arquivo de origem não puder ser decodificado, um `ErroDeSintaxe` será gerado.

### 2.1 Linhas Lógicas ###

Um programa em Jararaca é dividido em várias "linhas lógicas". O final de uma linha lógica é representado pelo token NOVALINHA.

Uma linha lógica é construída a partir de uma linha física. Uma linha física é uma sequência de caracteres terminados por uma sequência de fim de linha. Qualquer uma das sequências de fim de linha de uma plataforma popular pode ser usada - em Unix usando ASCII LF (avanço de linha), no Windows usando a sequência ASCII CR LF (retorno seguido de avanço de linha) ou o antigo input do Macintosh usando o caractere ASCII CR (retorno).

Uma linha lógica que contém apenas espaços e tabs é ignorada (ou seja, nenhum token NOVALINHA é gerado).

### 2.2 Identação ###

Os espaços em branco iniciais (`space`) no início de uma linha lógica são usados ​​para calcular o nível de indentação da linha lógica, que por sua vez é usado para determinar o agrupamento de instruções. O número total de espaços que precedem o primeiro caractere que não está em branco determina a identação da linha. 

Tabs são substituídos (da esquerda para a direita) por espaços. É rejeitado e tratado como inconsistente se um código de origem juntar tabs e espaços de uma maneira que torne o significado dependente do valor de um tab nos espaços, então um `TabError` é gerado nesse caso.

Os níveis de identação de linhas consecutivas são usados ​​para gerar tokens SOBE e DESCE, usando uma pilha de números, da seguinte maneira:

Os números colocados na pilha sempre incrementarão estritamente de baixo para cima. No início de cada linha lógica, o nível de indentação da linha é comparado ao topo da pilha. Se é igual, nada acontece. Se for maior, é colocado na pilha e um token SOBE é gerado. Se for menor, deve ser um dos números que ocorrem na pilha; todos os números na pilha maiores são retirados da pilha e, para cada número retirado, um token DESCE é gerado. No final do arquivo, um token DESCE é gerado para cada número restante na pilha maior que zero.

### 2.3 Identificadores ###

Além de NOVALINHA, SOBE e DESCE, existem as outros tokens que são os identificadores: palavras-chave, literais, operadores e delimitadores. Os caracteres de espaço em branco (exceto os terminadores de linha, discutidos anteriormente) não são tokens, mas servem para delimitar tokens.

#### 2.3.1 Palavras-Chave ####

Os seguintes identificadores são usados ​​como palavras reservadas ou palavras-chave da Jararaca:

```
retorna    nao      enquanto
se         funcao   quebra
```

#### 2.3.2 Literais ####

Literais são notações para cadeias de caracteres que representam valores constantes de alguns tipos internos:

- Palavra: Literais de palavra são descritos pelas seguintes definições lexicais:

```
palavraliteral        ::=  palavracurta | palavralonga
palavracurta          ::=  "'" itempalavracurta* "'" | '"' itempalavracurta* '"'
palavralonga          ::=  "'''" itempalavralonga* "'''" | '"""' itempalavralonga* '"""'
itempalavracurta      ::=  caracterpalavracurta
itempalavralonga      ::=  caracterpalavralonga 
caracterpalavracurta  ::=  <qualquer caracter exceto nova linha ou aspas>
caracterpalavralonga  ::=  <qualquer caracter>
```

- Número: Literais de números inteiros descritos pelas seguintes definições lexicais:

```
numero        ::=  digitosemzero (["_"] digito)* | "0"+ (["_"] "0")*
digitosemzero ::=  "1"..."9"
digito        ::=  "0"..."9"
```
#### 2.3.3 Operadores ####

Os seguintes tokens são operadores:

```
+       -       *       /         
!=      ==      &       |      
<       >       <=      >=           
```

#### 2.3.4 Delimitadores ####

Os seguintes tokens são delimitadores:

```
(       )       =       +=
```

Os seguintes caracteres ASCII têm um significado especial como parte de outros tokens ou são significativos para o analisador lexical:

```
'       "   
```

## 3 Estrutura e modelo de execução ##

O programa em Jararaca é construído a partir de blocos de código. Um bloco é uma parte de código Jararaca que é executado como uma unidade. Os tipos de blocos são: um módulo ou um corpo de uma função.

Dentro dos blocos existem *declarações*. Essas são as instruções que um intérprete Jararaca pode executar.

### 3.1 Declarações Simples ###

```
simple_stmt     ::=  expression_stmt
                      | return_stmt
                      | break_stmt
expression_stmt ::=  starred_expression                 
return_stmt     ::=  "retorna" [expression_list]          
break_stmt      ::=  "quebra"       
```

### 3.2 Declarações Compostas ###

```
compound_stmt ::=  if_stmt
                   | while_stmt
                   | funcdef 
```