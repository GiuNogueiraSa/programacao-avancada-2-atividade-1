# Tradutor de #hashtags e @menções

Atividade 1 – Programação Avançada 2 (Compiladores)
Tutorial prático de RegEx em JavaScript, com os **4 desafios extras** implementados.

## Como executar

Não é preciso instalar nada. Abra o `index.html` no navegador (Chrome, Edge ou Firefox) com duplo clique.

## O que a página faz

| Marcação | Exemplo | Resultado |
|---|---|---|
| `@usuario` | `@joao` | João da Silva |
| `#categoria:valor` | `#frutas:banana` | 🍌 |
| `$texto$` (Desafio 4) | `$muito$` | **muito** |

Hashtags sem categoria (`#festa`), hashtags fora da lista (`#frutas:kiwi`) e menções fora da lista (`@zeca`) aparecem em vermelho e são listadas em "Erros encontrados".

## Desafios extras

### 1. Contador
Um objeto `contagem = { mencoes, hashtags, negritos }` é criado a cada clique e somamos 1 apenas nos `return` que deram certo. O total aparece abaixo do resultado.

### 2. E-mails
Regex das menções:

```js
/(?<![\wÀ-ÿ.])@([\wÀ-ÿ]+(?:\.[\wÀ-ÿ]+)*)/g
```

O trecho `(?<![\wÀ-ÿ.])` é um **lookbehind negativo**: ele exige que o `@` **não** venha colado em letra, dígito, `_` ou ponto. Por isso `ana@site.com` deixa de gerar o erro "Menção desconhecida: @site".

### 3. Nomes com ponto
O trecho `(?:\.[\wÀ-ÿ]+)*` só aceita o ponto quando há pelo menos uma letra depois dele.
- `@maria.silva` → casa o nome inteiro → **Maria da Silva**
- `Vou com @maria.` → o ponto final fica fora do nome → **Maria Oliveira.**

### 4. Nova marcação: negrito com `$texto$`

```js
/\$([^$\n]+)\$/g
```

- `\$` é o caractere `$` (escapado, porque `$` sozinho em regex quer dizer "fim do texto").
- `([^$\n]+)` captura tudo até o próximo `$`, sem atravessar linhas.
- Um `$` sem par (ex.: `custa $10`) não é alterado.

## Testes realizados

| # | Entrada | Resultado | Erros |
|---|---|---|---|
| 1 | `Hoje @joao trouxe #frutas:banana e @maria levou #comidas:pizza!` | Hoje João da Silva trouxe 🍌 e Maria Oliveira levou 🍕! | Nenhum |
| 2 | `Vai ter #festa amanhã` | #festa em vermelho | Hashtag em formato inválido |
| 3 | `Oi @zeca, #frutas:kiwi é bom` | os dois em vermelho | Menção desconhecida; Hashtag desconhecida |
| 4 | `@Maria adora #Animais:Gato` | Maria Oliveira adora 🐱 | Nenhum |
| 5 | `Comi #frutas:maçã!` | Comi 🍎! | Nenhum |
| 6 | `<b>@ana</b>` | `<b>Ana Souza</b>` como texto | Nenhum |
| D2 | `Meu email é ana@site.com` | texto sem alteração | Nenhum |
| D3 | `Oi @maria.silva! Vou com @maria.` | Oi Maria da Silva! Vou com Maria Oliveira. | Nenhum |
| D4 | `Isso é $muito$ bom e custa $10` | Isso é **muito** bom e custa $10 | Nenhum |

## Relação com compiladores

| No tradutor | Na compilação |
|---|---|
| A regex reconhece `#categoria:valor` e `@usuario` | Análise léxica: reconhecer tokens com expressões regulares |
| `#festa` rejeitada por não seguir o formato | Erro léxico/sintático |
| `#frutas:kiwi` tem formato certo, mas não está na lista | Erro semântico (como usar uma variável não declarada) |

### Discussão

**A regex `#([\wÀ-ÿ]+)(?::([\wÀ-ÿ]+))?` é uma expressão regular no sentido formal?**
Sim. Ela só usa as três operações das ER formais:
- **concatenação**: `#` seguido da categoria, seguido de `:` e do valor;
- **união**: a classe `[\wÀ-ÿ]` é só uma abreviação de `a | b | ... | z | 0 | ... | 9 | _ | à | ...`;
- **fecho de Kleene**: `x+` equivale a `x x*`, e `x?` equivale a `(x | ε)`.

Os grupos de captura `( )` e `(?: )` não mudam a linguagem reconhecida; eles só servem para o JavaScript guardar pedaços do texto.

**Dá para escrever um AFD equivalente?**
Sim. Toda linguagem descrita por uma ER formal é regular, e toda linguagem regular é reconhecida por um AFD. Chamando de L o conjunto de caracteres permitidos:

| Estado | Lê | Vai para |
|---|---|---|
| q0 (inicial) | `#` | q1 |
| q1 | L | **q2** (final: hashtag sem valor, ex. `#festa`) |
| **q2** | L | q2 |
| **q2** | `:` | q3 |
| q3 | L | **q4** (final: `#categoria:valor`) |
| **q4** | L | q4 |

Inclusive, o AFD mostra de onde vem o erro "formato inválido": a cadeia termina em q2 em vez de q4.

**E exigir a categoria repetida no final (`#frutas:banana#frutas`)?**
Não é possível só com ER formal. Essa linguagem é do tipo `{ #w:v#w }`, em que a mesma cadeia `w` precisa aparecer duas vezes. Para conferir isso, o autômato teria que "lembrar" uma categoria de tamanho qualquer, e um AFD tem um número finito de estados. Pelo **lema do bombeamento**, a linguagem não é regular. No JavaScript daria para fazer com uma *retroreferência* (`#([\wÀ-ÿ]+):([\wÀ-ÿ]+)#\1`), mas o `\1` é um recurso que vai além das expressões regulares formais.
