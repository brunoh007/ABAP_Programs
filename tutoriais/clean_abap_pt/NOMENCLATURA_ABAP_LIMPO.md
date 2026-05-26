# 🎯 Nomenclatura - Clean ABAP PT

> Baseado em: [Clean ABAP - Names](https://github.com/SAP/styleguides/blob/main/clean-abap/CleanABAP.md#names)

## 📋 Índice

1. [Use Nomes Descritivos](#use-nomes-descritivos)
2. [Prefira Termos de Domínio](#prefira-termos-de-domínio)
3. [Use Plural](#use-plural)
4. [Use Nomes Pronunciáveis](#use-nomes-pronunciáveis)
5. [Use snake_case](#use-snake_case)
6. [Evite Abreviações](#evite-abreviações)
7. [Use as Mesmas Abreviações](#use-as-mesmas-abreviações)
8. [Substantivos para Classes, Verbos para Métodos](#substantivos-para-classes-verbos-para-métodos)
9. [Evite Palavras Vazias](#evite-palavras-vazias)
10. [Uma Palavra por Conceito](#uma-palavra-por-conceito)
11. [Use Nomes de Padrões com Cuidado](#use-nomes-de-padrões-com-cuidado)
12. [Evite Codificações e Notação Húngara](#evite-codificações-e-notação-húngara)
13. [Evite Obscurecer Funções Built-in](#evite-obscurecer-funções-built-in)

---

## Use Nomes Descritivos

**Objetivo:** Criar código autoexplicativo que comunica sua intenção.

Os nomes devem transmitir claramente o significado e o conteúdo das coisas, priorizando **o quê** a variável/classe representa, não o seu tipo de dado.

### ✅ Bom

```ABAP
CONSTANTS max_wait_time_in_seconds TYPE i ...
DATA customizing_entries TYPE STANDARD TABLE ...
METHODS read_user_preferences ...
CLASS /clean/user_preference_reader ...
```

**Por quê?** Lendo o nome, você entende imediatamente o propósito.

### ❌ Evitar

```ABAP
" anti-pattern
CONSTANTS sysubrc_04 TYPE sysubrc ...
DATA iso3166tab TYPE STANDARD TABLE ...
METHODS read_t005 ...
CLASS /dirty/t005_reader ...
```

**Por quê?** Os nomes não revelam a intenção. Você precisa olhar documentação externa ou código para entender.

---

## Prefira Termos de Domínio

**Objetivo:** Escolher os melhores termos para tornar o código compreensível para toda a equipe.

Busque bons nomes em dois lugares:

- **Domínio da Solução:** Termos técnicos da computação (`queue`, `tree`, `factory`)
- **Domínio do Problema:** Termos de negócio (`account`, `ledger`, `customer`)

### Regra Prática

- **Camadas de negócio** → Use termos do domínio do problema
- **Camadas técnicas** (factories, algoritmos) → Use termos do domínio da solução

### Importante

Não invente sua própria linguagem. Use termos que desenvolvedores, gerentes de produto e clientes possam entender sem um dicionário customizado.

### ✅ Exemplo

```ABAP
CLASS /clean/account_reader         " Domínio do problema
CLASS /clean/factory_account_reader " Domínio da solução (factory é um padrão)
```

---

## Use Plural

**Objetivo:** Deixar óbvio quando você está trabalhando com coleções.

A prática legada SAP era nomear tabelas em singular (`country` para "tabela de países"). A tendência moderna é usar plural.

### ✅ Recomendado

```ABAP
DATA countries TYPE STANDARD TABLE ...
DATA users TYPE STANDARD TABLE ...
```

### ❌ Evitar (Prática Legada)

```ABAP
DATA country TYPE STANDARD TABLE ...  " Confuso: singular para múltiplos itens
DATA user TYPE STANDARD TABLE ...
```

> **Nota:** Para tabelas de banco de dados, a convenção pode ser diferente (nomes em singular são tradicionais).

---

## Use Nomes Pronunciáveis

**Objetivo:** Facilitar a comunicação e discussão sobre o código.

Desenvolvemos pensando e falando sobre objetos. Use nomes que você consiga pronunciar facilmente.

### ✅ Bom

```ABAP
DATA detection_object_types TYPE ...
METHODS get_maximum_connections( )
```

**Por quê?** Você consegue falar: "detection object types" (tipo de objeto de detecção).

### ❌ Evitar

```ABAP
DATA dobjt TYPE ...
METHODS get_max_conns( )
```

**Por quê?** "dobjt" é impossível de pronunciar naturalmente.

---

## Use snake_case

**Objetivo:** Manter convenção consistente em uma linguagem case-insensitive.

ABAP é insensível a maiúsculas/minúsculas, então a recomendação é usar `snake_case` (palavras separadas por underscore) para toda a nomenclatura.

### ✅ Recomendado

```ABAP
" Variável com nome claro em snake_case
DATA max_response_time_in_millisec TYPE i.
```

### ❌ Evitar

```ABAP
" flatcase: sem separadores visuais
DATA maxresponsetimeinmilliseconds TYPE i.

" UPPERCASE: não segue a convenção moderna
DATA MAX_RESPONSE_TIME_IN_MILLISEC TYPE i.
```

### Dica para Nomes Longos

Se atingir o limite de caracteres (ex: 30 para métodos), conscientiosamente use abbreviações nos pontos certos, mas mantenha `snake_case`.

---

## Evite Abreviações

**Objetivo:** Maximizar a compreensão e reduzir ambiguidade.

Se tiver espaço, escreva nomes completos. Abrevie apenas quando ultrapassar limitações de tamanho.

### Quando Precisar Abreviar

1. Comece pelas palavras **menos importantes**
2. Mantenha consistência

### Por Que Evitar?

Abreviações criam ambiguidade muito rápido. Por exemplo, "cust" pode significar:
- **customizing** (personalização)
- **customer** (cliente)
- **custom** (customizado)

Todos os três são comuns em aplicações SAP!

### ✅ Bom

```ABAP
DATA customer_number TYPE ...
DATA customizing_entries TYPE ...
```

### ❌ Evitar

```ABAP
DATA cust_num TYPE ...      " cust = customizing ou customer?
DATA cust_entries TYPE ...  " Ambíguo!
```

---

## Use as Mesmas Abreviações

**Objetivo:** Permitir que colegas encontrem código relacionado.

Pessoas buscam por palavras-chave para localizar código. Use a **mesma abreviação** para o mesmo conceito em todo o projeto.

### ✅ Bom

```ABAP
" Sempre "dobjt" para "detection object type"
DATA dobjt_list TYPE ...
METHODS get_dobjt( )
METHODS validate_dobjt( )
```

### ❌ Evitar

```ABAP
" Diferentes abreviações para o mesmo conceito
DATA dot_list TYPE ...        " dot
METHODS get_dotype( )         " dotype
METHODS validate_detobjtype( ) " detobjtype
```

**Por quê?** A busca por "dobjt" não encontrará os outros usos!

---

## Substantivos para Classes, Verbos para Métodos

**Objetivo:** Deixar claro o que é um tipo vs. uma ação.

### Classes = Substantivos

Use substantivos ou frases nominais:

```ABAP
CLASS /clean/account
CLASS /clean/user_preferences
INTERFACE /clean/customizing_reader
```

**Por quê?** Classes representam "coisas" (objetos).

### Métodos = Verbos

Use verbos ou frases verbais:

```ABAP
METHODS withdraw
METHODS add_message
METHODS read_entries
```

**Por quê?** Métodos representam "ações".

### Métodos Booleanos com Prefixo

Para métodos que retornam verdadeiro/falso, use `is_` ou `has_`:

```ABAP
IF is_empty( table ).
IF has_permission( user ).
```

**Por quê?** A leitura fica natural: "if is empty" (se está vazio).

---

## Evite Palavras Vazias

**Objetivo:** Remover ruído e deixar nomes mais precisos.

Palavras como `data`, `info`, `object` geralmente não adicionam informação real.

### ✅ Bom

```ABAP
account              " ao invés de: account_data
alert                " ao invés de: alert_object
user_preferences     " ao invés de: user_info
response_time_in_sec " ao invés de: response_time_variable
```

**Por quê?** Mais conciso e ainda claro.

### Quando Adicionar Contexto

Se remover a palavra deixar obscuro, adicione contexto mais específico:

```ABAP
" Ao invés de apenas "response_time"
response_time_in_seconds

" Deixa muito claro qual unidade
```

---

## Uma Palavra por Conceito

**Objetivo:** Evitar que o leitor desperdice tempo procurando diferenças inexistentes.

Escolha um termo para cada conceito e use-o consistentemente. Não misture sinônimos.

### ✅ Bom

```ABAP
METHODS read_this.
METHODS read_that.
METHODS read_those.
```

**Por quê?** O padrão é óbvio: sempre "read".

### ❌ Evitar

```ABAP
METHODS read_this.
METHODS retrieve_that.    " retrieve = sinônimo de read?
METHODS query_those.      " query = sinônimo de read?
```

**Por quê?** O leitor gasta tempo pensando: "há uma diferença entre read, retrieve e query?"

### Mapeie Seus Termos

Dentro de seu projeto:
- Sempre use `read` (não `retrieve`, `fetch`, `query`)
- Sempre use `create` (não `build`, `construct`, `make`)
- Sempre use `validate` (não `check`, `verify`, `test`)

---

## Use Nomes de Padrões com Cuidado

**Objetivo:** Usar termos de design pattern apenas quando realmente implementar o padrão.

Não chame sua classe `file_factory` a menos que ela **realmente implemente** o Factory Pattern.

### Padrões Comuns

- [singleton](https://en.wikipedia.org/wiki/Singleton_pattern)
- [factory](https://en.wikipedia.org/wiki/Factory_method_pattern)
- [facade](https://en.wikipedia.org/wiki/Facade_pattern)
- [composite](https://en.wikipedia.org/wiki/Composite_pattern)
- [decorator](https://en.wikipedia.org/wiki/Decorator_pattern)
- [iterator](https://en.wikipedia.org/wiki/Iterator_pattern)
- [observer](https://en.wikipedia.org/wiki/Observer_pattern)
- [strategy](https://en.wikipedia.org/wiki/Strategy_pattern)

### ✅ Bom

```ABAP
" Realmente implementa Factory Pattern
CLASS /clean/account_factory
  METHODS:
    create_from_file( file ) RETURNING account,
    create_from_database( id ) RETURNING account.
ENDCLASS.
```

### ❌ Evitar

```ABAP
" Apenas retorna um objeto - NÃO é um factory!
CLASS /dirty/account_factory
  METHODS get_account( ) RETURNING account.
ENDCLASS.
```

---

## Evite Codificações e Notação Húngara

**Objetivo:** Deixar código mais limpo sem perder informação.

Remova **todos** os prefixos de codificação (como `i_`, `e_`, `rv_` etc).

### ✅ Recomendado

```ABAP
METHOD add_two_numbers.
  result = a + b.
ENDMETHOD.
```

### ❌ Evitar (Notação Húngara)

```ABAP
METHOD add_two_numbers.
  rv_result = iv_a + iv_b.
ENDMETHOD.
```

**Prefixos comuns (evite):**
- `i_` = input/importing
- `e_` = export/exporting
- `rv_` = return value
- `l_` = local
- `c_` = constant
- `m_` = member
- `g_` = global

**Por quê remover?**
1. Seus IDEs modernos mostram o tipo
2. O tipo não muda, o significado pode mudar
3. Deixa nome mais curto e legível
4. Novos ABAPers não aprendem essa convenção antiga

> Para mais detalhes, veja: [Avoid Encodings](https://github.com/SAP/styleguides/blob/main/clean-abap/sub-sections/AvoidEncodings.md)

---

## Evite Obscurecer Funções Built-in

**Objetivo:** Manter funções nativas do ABAP acessíveis.

Dentro de uma classe, métodos com mesmo nome obscurecem funções built-in do ABAP, **independentemente** do número e tipo de argumentos.

### ❌ Evitar

```ABAP
" Obscurece a função built-in lines()
METHODS lines RETURNING VALUE(result) TYPE i.

" Obscurece a função built-in line_exists()
METHODS line_exists RETURNING VALUE(result) TYPE i.
```

### ❌ Evitar (Methods de classe)

```ABAP
" Obscurece built-in condense()
CLASS-METHODS condense RETURNING VALUE(result) TYPE i.

" Obscurece built-in strlen()
CLASS-METHODS strlen RETURNING VALUE(result) TYPE i.
```

### ✅ Bom

```ABAP
" Renomear para evitar conflito
METHODS count_lines RETURNING VALUE(result) TYPE i.
METHODS line_present RETURNING VALUE(result) TYPE i.
```

**Por quê?** Permite que o código continue usando as funções built-in do ABAP sem confusão.

---

## 📚 Resumo das Melhores Práticas

| Prática | ✅ Fazer | ❌ Não Fazer |
|---------|---------|------------|
| **Nomes Descritivos** | `max_wait_time_in_seconds` | `sysubrc_04` |
| **Termos de Domínio** | `account`, `customer`, `ledger` | Linguagem customizada |
| **Plural** | `countries`, `users` | `country`, `user` (para tabelas) |
| **Pronunciável** | `detection_object_types` | `dobjt` |
| **Formato** | `snake_case` | `flatcase`, `UPPERCASE` |
| **Abreviações** | Evitar, ou uso consistente | Uso inconsistente |
| **Classes** | Substantivos: `account_reader` | Verbos: `read_account` |
| **Métodos** | Verbos: `read_account` | Substantivos: `account_read` |
| **Palavras Vazias** | `account` | `account_data` |
| **Consistência** | Sempre `read` | `read`, `retrieve`, `fetch` |
| **Padrões** | Apenas se realmente usar | Nomes genéricos como factory |
| **Notação Húngara** | Remover `i_`, `e_`, `rv_` | Usar prefixos de tipo |
| **Built-ins** | Não obscurecer: `count_lines` | Obscurecer: `lines` |

---

## 🎯 Próximas Leituras

Recomendamos estudar também:

- [Use Pronounceable Names](#use-nomes-pronunciáveis) - Facilita discussões em equipe
- [Pick One Word per Concept](#uma-palavra-por-conceito) - Mantém código consistente
- [Avoid Noise Words](#evite-palavras-vazias) - Deixa tudo mais claro

---

**Baseado em:** Robert C. Martin's _Clean Code_, Chapter 2: Meaningful Names  
**Versão:** Clean ABAP - Português Brasil  
**Referência:** https://github.com/SAP/styleguides/blob/main/clean-abap/CleanABAP.md#names
