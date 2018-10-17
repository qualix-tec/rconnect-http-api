## R-Connect HTTP API

Uma API feita para clientes da Qualix para permitir o uso com outros sistemas via interface HTTP, o que permite uma integração indiferente do banco.

## Objetivo

Esta API tem o propósito de permitir a manipulação das tabelas persistidas e utilizadas pelo R-Connect.
Com ela é possível realizar as seguintes operações:

* Criação
* Leitura
* Atualização
* Exclusão

## Como utilizar

A instalação deve ser feita de antemão pela equipe de Qualix.
Caso a API não esteja disponível contate o nosso suporte.

Após feita a instalação, a utilização da API depende de alguns requisitos que devem ser cumpridos.

### Autenticação

Todas as requisições feitas pela API requerem que o usuário esteja autenticado. Para isso, no cabeçalho `Authorization` da requisição, insira o usuário e a senha fornecido pela Qualix, no formato [Basic](https://pt.stackoverflow.com/a/256521).

### Formato da URL

Abaixo está um template da URL utilizado pela API:

`<VERBO> /qualix/rconnect/api/crud/:acao/[:id]?ds={persistencia}`

Aonde:

* `<VERBO>` representa o verbo HTTP. Podendo ser: `GET`, `POST`, `PUT`, `DELETE`.
* `:acao` é o nome da operação a ser realizada, é ela quem decide o `<VERBO>` a ser empregado. Podendo ser: `create`, `read`, `update`, `delete`, `upsert`, `bulk/upsert`.
* `[:id]` também dependente da operação `:acao` é um parâmetro que representa o id a ser _modificado_ ou _removido_ do banco.
* `{persistencia}` representa o nome da tabela/classe aonde a `:acao` irá tomar efeito.

### Conteúdo

Para requisições do tipo `create`, `update`, `upsert` ou `bulk/upsert`, um [payload](http://www.ramosdainformatica.com.br/json-o-que-e-payload/) no formato [JSON](https://www.devmedia.com.br/json-tutorial/25275) deve ser enviado. Este payload deve contar propriedades com os nomes iguais aos campos na tabela `{persistencia}`, caso contrário a propriedade é ignorada.

> __NOTA:__ Não é possível adicionar novos campos, a API limita-se apenas a utilizá-los. Para adicionar novos campos é necessário entrar em contato com o suporte da Qualix para personalizar a API, tenha em mente que isso pode agregar custos.

### O parâmetro \_\_id\_\_

Para a API de integração, este parâmetro é especial, pois ele representa o `ID` do registro no banco. É ele quem auxilia o banco a alterar um registro ou criar um novo. Este parâmetro pode ser utilizado de duas formas:

A. Para operações de inserção/inserção condicionada/alteração/criação em massa, ele é utilizado dentro do payload. Exemplo: `{ "__id__": 1, "nome": "John Doe" }` vai informar o banco para utilizar o id 1 e atualizar o campo `nome` ao invés de criar um novo registro.

B. Para operações de leitura/exclusão, ele é representado pelo parâmetro `[:id]` na URL.

## Operações

As operações são endereços que fazem parte da API para realizar determinadas ações.

#### Leitura

Retorna o objeto a partir do id fornecido.

> __Formato:__ `GET /qualix/rconnect/api/crud/:id?ds={persistencia}&d={profundidade}`

No momento as consultas estão limitadas por ID.

O parâmetro `{profundidade}` existe somente para leitura, este parâmetro representa até que nível de profundidade um objeto deve ser serializado. Por padrão esse nível é 7, o que na maior parte das vezes é o suficiente para visualizar um objeto inteiro.

> __NOTA:__ Se existir repetição de objetos com grande frequência em __profundidades diferentes,__ pode ser necessário rever esse valor para um número menor.

#### Inserção (create)

Cria um novo registro no banco a partir do payload fornecido.

> __Formato:__ `POST /qualix/rconnect/api/crud/create?ds={persistencia}`

Retorna um objeto com a propriedade `__id__` representando o id do novo registro criado.

#### Atualização (update)

Atualiza um registro a partir do `:id` fornecido. Se o id não existir, um erro é retornado.

> __Formato:__ `PUT /qualix/rconnect/api/crud/update/:id`

Retorna um objeto com a propriedade `__id__` representando o id do registro atualizado.

### Exclusão (delete)

Exclui um registro do banco a partir do `:id` fornecido. Se o id não existir, um erro é retornado.

> __Formato:__ `DELETE /qualix/rconnect/api/crud/delete/:id`

Retorna um objeto com a propriedade `deleted` com valor `true` caso a operação tenha sucedido.

#### Inserção condicionada (upsert)

Cria ou atualiza um novo registro. A atualização depende do parâmetro `:id` em que caso o seu valor não seja encontrado, um novo registro é criado e este valor é ignorado.

> __Formato:__ `POST qualix/rconnect/api/qualix/crud/upsert/:id`

Assim como a operação `update`, esta operação exige um payload com os campos a serem atualizados.

Retorna um objeto com a propriedade `__id__` representando o id do registro criado ou atualizado.

> __NOTA:__ Apesar de ser possível utilizar a operação `create` em conjunto com o parâmetro `__id__` para simular o mesmo efeito dessa operação, ainda é recomendado o uso de `upsert` devido a validações adicionais e questões de legibilidade.

### Inserção em massa (bulk upsert)

Cria ou atualiza mais de um registro, diferente das operações `create`, `upsert` e `update`, esta operação deve receber um array de objetos, cada objeto deve conter um `__id__`, que caso não esteja presente resulta em um novo registro sendo criado com as propriedades do objeto decorrente.

>  __Formato:__ `POST qualix/rconnect/api/crud/bulk/upsert`

Retorna um array de objetos representando sucesso ou erro.


### Esquema

Em adição às operações, também existe uma forma de visualizar quais propriedades compõem uma class/tabela.

> __Formato:__ `POST qualux/rconnect/api/schemas/:persistencia`

Aonde `:persistencia` pode ser o nome de uma classe no sistema. Por exemplo: `QualixInt.PedPendentes`.

### Gerenciando erros

Quando uma operação falhar, os erros podem variar em dois formatos:

A. Um objeto com a propriedade `error` conterá a propriedade `message`, com o texto do erro. Cada objeto pode conter erros de origem representado por `origin` que geraram o erro do nível mais alto do objeto. Uma operação também pode gerar uma coleção de erros, este é representado pelo objeto `errors`

B. Um objeto com a propriedade `errors` conterá um array de objetos `error`, seguindo o formato A.

> __NOTA:__ A geração de erros pode intercalar entre os dois formatos, conforme o exemplo abaixo.

```json
{
  "error": {
    "message": "Ocorreu um erro de aplicação",
    "internalCode": 5001,
    "origin": {
      "message": "Erro de permissão.",
      "internalCode": 5001,
      "errors": [
        {
          "message": "Erro ao salvar arquivo A.txt",
          "internalCode": 5001
        },
        {
          "message": "Erro ao salvar arquivo B.txt",
          "internalCode": 5001
        }
      ]
    }
  }
}
```
