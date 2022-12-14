# Hash Table

## Índice

- [Criando a classe HashTable](#criando-a-classe-hashtable)
- [Lidando com colisões](#lidando-com-colisões)
  - [Encadeamento (Separate chaining)](#encadeamento-separate-chaining)
  - [Sondagem linear (Linear probing)](#sondagem-linear-linear-probing)
- [Criando funções de _hash_ melhores](#criando-funções-de-hash-melhores)

</br>

# Criando a classe HashTable

Usaremos um _objeto_ para representar a estrutura de dados.

```javascript
import { defaultToString } from '../utils.js';

class HashTable {
  _table;
  _toStrFn;

  constructor(toStrFn = defaultToString) {
    this._toStrFn = toStrFn;
    this._table = {};
  }

  size() {
    return Object.keys(this._table).length;
  }

  isEmpty() {
    return this.size() === 0;
  }

  clear() {
    this._table = {};
  }
}
```

Também criaremos uma classe para gerar um para chave/valor, que será o valor salvo na tabela. Esta classe também terá um método toString:

```javascript
export default class ValuePair {
  #key;
  #value;
  constructor(key, value) {
    this.#key = key;
    this.#value = value;
  }
  toString() {
    return `[#${this.#key}: ${this.#value}]`;
  }

  get key() {
    return this.#key;
  }

  get value() {
    return this.#value;
  }
}
```

A seguir, precisamos implementar três métodos básicos na classe HashTable:

- `put(key, value)`: adiciona um novo item à _hash table_ (ou atualiza item existente);
- `remove(key, value)`: remove o _valor_ da tabela cuja chave é _key_;
- `get(key)`: retorna o _valor_ associado à _key_;

## Criando a função _hash_

Antes de implementar os três métodos mencionados acima, precisamos criar a função _hash_:

```javascript
_loseloseHashCode(key) {
  if (typeof key === 'number') return key;

  const tableKey = this._toStrFn(key);
  let hash = 0;
  for (let i = 0; i < tableKey.length; i++) {
    hash += tableKey.charCodeAt(i);
  }

  return hash % 37
}

_hashCode(key) {
  return this._loseloseHashCode(key);
}
```

No método `loseloseHashCode` verificamos se `key` é um número; caso seja, a função retorna esse número. Em seguida, é gerado um número através da soma dos valores ASCII de todas os caracteres da _key_. Por fim, a função retorna o valor _hash_. Para trabalharmos com números baixos, usamos o resto da divisão do _hash_ por um número arbitrário - isso previne trabalharmos com números muito grandes.

OBS: mais sobre a tabela ASCII [aqui](https://www.asciitable.com/)

Com a função _hash_ concluída, podemos implementar os outros métodos.

## Inserindo uma chave e um valor na _hash table_

```javascript
put(key, value) {
  if (key !== null && value !== null) {
    const position = this._hashCode(key);
    this._table[position] = new ValuePair(key, value);
    return true;
  }
  return false;
}
```

Primeiramente é verificado se _key_ e _value_ são válidos; caso não sejam, a função retorna `false`, indicando que o dado não foi inserido (ou atualizado) na _hash table_.

Para um par chave/valor válido, a função define uma posição na tabela usando `hashCode(key)`. É criada, então, uma instância de `ValuePair` com a _chave_ e o _valor_ e na tabela uma nova instância de `ValuePair é armazenada.

## Extraindo valor da tabela _hash_

```javascript
get(key) {
  const valuePair = this._table[this._hashCode(key)];
  return valuePair === null ? undefined : valuePair.value
}
```

Primeiro achamos a posição da _key_ com a função _hashCode_, então acessamos a tabela nessa posição. Por fim a função retorna o valor associado à chave.

## Removendo valores da tabela _hash_

```javascript
remove(key) {
  const hash = this._hashCode(key);
  const valuePair = this._table[hash];
  if (valuePair !== null) {
    delete this._table[hash];
    return true;
  }
  return false;
}
```

Para remover um valor da _hash table_, primeiro identificamos sua posição com a função `hashCode`. Caso o _valor_ seja diferente de `null` (pois _hash tables_ não aceitam `null` como uma chave válida), deletamos esse valor com o operador `delete`. A função retornar `true` se a remoção aconteceu, ou `false` caso a remoção não tenha ocorrido.

<hr>

</br>

# Lidando com colisões

Às vezes chaves diferentes podem ter o mesmo _hash_. Chamamos essa situação **colisão** porque diferentes pares _chave/valor_ são atribuidos à mesma posição de uma _hash table_.

Suponha a seguinte situação:

```javascript
const hash = new HashTable();
hash.put('Jonathan', 'jon@email.com'); // loseloseHashCode('Jonathan') retorna 5
hash.put('Jamie', 'jamie@email.com'); // loseloseHashCode('Jamie') retorna 5
hash.put('Sue', 'sue@email.com'); // loseloseHashCode('Sue') retorna 5
```

Podemos implementar a seguinte função `toString` na classe `HashTable` para verificar como ficaria a tabela após as inserções acima.

```javascript
toString() {
  if (this.isEmpty()) return '';

  const keys = Object.keys(this._table);
  let objString = `{${keys[0]} => ${this._table[keys[0]].toString()}}`;

  for (let i = 1; i < keys.length; i++) {
    objString = `${objString}\n{${keys[i]} => ${this._table[keys[i]].toString()}}`
  }

  return objString;
}
```

Após chamar `console.log(hash.toSring())` teremos o seguinte resultado:

```javascript
console.log(hash.toString());
// {5 => [#Sue: sue@email.com]}
```

`Jonathan`, `Jamie` e `Sue` receberam uma mesma _hash_, e `Sue`, por ter sido a última adição à tabela, fica sendo a ocupante da posição `5`.

Há algumas técnicas para lidar com colisões: _separate chaining_ (encadeamento), _linear probing_ (sondagem linear) e _double hashing_. Comentaremos a seguir acerca das duas primeiras.

## Encadeamento (Separate chaining)

A técnica **separate chaining** consiste em criar uma lista encadeada (_linked list_) para cada posição da tabela, armazenando elementos nessa lista. É o modo mais simples de lidar com colisões, no entanto requer consumo adicional de memória fora da instância da _hash table_.

<figure>
    <img src="../separate-chaining-example.png"
         alt="Separate chaining">
    <figcaption>Os valores foram omitidos para simplificar o diagrama</figcaption>
</figure>

Para usar as técnicas _separate chaining_ e _linear probing_ precisamos alterar três métodos: `put`, `get`, e `remove`. Esses métodos serão diferentes para cada técnica.

Começemos com a implementação da classe `HashTableSeparateChaining`:

```javascript
import ValuePair from './ValuePair.js';
import LinkedList2 from '../linked-list/LinkedList2.js';
import { defaultToString } from '../utils.js';
import HashTable from './HashTable.js';

export default class HashTableSeparateChaining extends HashTable {
  _table;

  constructor(toStrFn = defaultToString) {
    super(toStrFn);
    this._table = {};
  }
}
```

### Método `put` (separate chaining)

```javascript
put(key, value) {
  // If any 'nullish' value is passed into 'put' return 'false':
  if (
    key === undefined ||
    key === null ||
    value === undefined ||
    value === null
  )
    return false;
  // Hash the key
  const position = this._hashCode(key);
 // If there the position is empty, create a new linked list into it
  if (this._table[position] === null || this._table[position] === undefined) {
    this._table[position] = new LinkedList2();
  }
 // Insert the key/value into the linked list
  this._table[position].push(new ValuePair(key, value));
 // Return 'true' if the addition is successfull
  return true;
}
```

Verficamos se a posição em que tentaremos inserir um valor já possui outros valores. Caso seja o primeiro valor da posição, iniciamos uma instância de `LinkedList`, então adicionamos a instância de `ValuePair` à lista encadeada usando o método `insert`.

### Método `get` (separate chaining)

```javascript
get(key) {
  const position = this._hashCode(key);
  const linkedList = this._table[position];
  // If there is no such key, return 'undefined'
  if (
    linkedList === null ||
    linkedList === undefined ||
    linkedList?.isEmpty()
  ) {
    return undefined;
  }
  // Get the 'head' reference of the list
  let current = linkedList.head;
  // Find the wanted element based on the 'key'
  while (current !== null) {
    if (current.element.key === key) {
      return current.element.value;
    }
    current = current.next;
  }
}
```

Descobrimos a _hash_ da key inserida na função, então acessamos a posição _position_ na tabela, salvando o valor na variável _linkedList_.
Se _linkedList_ é `nullish` ou é uma lista vazia, o método retorna `undefined`.

Caso a lista não esteja vazia, iteramos nela até encontrar o elemento cuja propriedade _key_ seja igual ao argumento _key_ passado no método `get`.

### Método `remove` (separate chaining)

```javascript
remove(key) {
  const position = this._hashCode(key);
  const linkedList = this._table[position];
  //If there is no such key, return 'false'
  if (
    linkedList === null ||
    linkedList === undefined ||
    linkedList?.isEmpty()
  ) {
    return false;
  }
  // Get the head reference of the list:
  let current = linkedList.head;
  // Find the element to be deleted:
  while (current !== null) {
    if (current.element.key === key) {
      // When found, remove the element
      linkedList.remove(current.element);
      // If the list becomes empty, it is removed from the table
      if (linkedList.isEmpty()) {
        delete this._table[position];
      }
      // Retur 'true' to confirm the deletion
      return true;
    }
    current = current.next;
  }
}
```

Assim como no método `get`, primeiramente é verificado se há algum valor na tabela com a _key_ passada como argumento.

Caso haja uma lista na posição informada, iteramos sobre ela até encontrar o elemento cuja propriedade _key_ seja igual ao argumento passado no método `remove`. Então, o elemento encontrado é removido, usando o método `remove` da classe `LinkedList2`.

Por fim, caso a lista tenha ficado vazia após a remoção do elemento, a entrada afetada da tabela é removida, e a função retorna `true`.

## Sondagem linear (linear probing)

Esta técnica de resolução de conflitos chama-se _sondagem linear_ porque lida com colisões armazenando os valores diretamente na tabela, e não em um estrutura de dados a parte.

Basicamente, ao tentarmos adicionar um elemento numa _posição_, caso esta esteja ocupada o valor é armazenado na próxima posição vaga.

O seguinte diagrama exemplifica o processo:

![Linear probing](../linear-probing.png)

Se usarmos _linear probing_, para remover um par _chave-valor_ da _has table_ não basta remover o elemento da sua posição na tabela, como fizemos anteriormente. Se apenas removermos o elemento da tabela, é possível que ao procurar por outros elementos com mesmo _hash_ encontremos aquela posição vazia.

Há duas opções para lidar com esse problema na remoção de elementos. A primeira opção chama-se _soft delete_. Essa abordagem consiste em usar um valor convencional (_flag_) para indicar a remoção do par _chave/valor_, em vez de realmente deletar o elemento. Com o passar do tempo, no entanto, a tabela poderá conter muitas dessas _flags_, o que reduzirá a eficiência da tabela.

O seguinte diagrama exemplifica a abordagem _soft (lazy) deletion_:

![Soft deletion](../soft-deletion.png)

A outra estratégia possível para lidar com remoção de elementos é verificar, a cada remoção, se é necessário mover algum elemento a uma posição anterior na tabela. Ao procupara por uma _chave_, essa abordagem previne encontrar espaços vazios.

A seguinte imagem ilustra essa abordagem:

![Shift strategy](../shift-strategy.png)

Ambas as estratégias têm prós e contras. A seguir vai a implementação da segunda estratégia:

[_Linear probing_ com realocação de elementos](./HashTableLinearProbing1.js)

<hr>

</br>

# Criando funções de _hash_ melhores

A função _hash_ **lose-lose** não é boa por gerar muitas colisões. Uma boa _hash function_ deve ter as seguintes propriedades:

- Baixo tempo de inserção e extração de dados (performance);
- Baixa probabilidade de colisões

Um função simples que é melhor que a **lose-lose** chama-se **djb2**:

```javascript
djb2HashCode(key) {
  const tableKey = this._toStrFn(key);
  let hash = 5381;
  for (let i = 0; i < tableKey.length; i++) {
    hash = (hash * 33) + tableKey.charCodeAt(i);
  }
  return hash % 1013;
}
```

Transformada a _key_ em `string`, a função `djb2HashCode` inicia a variável `hash` com um número primo (a maioria das implementações usa `5381`); então, iterando por cada caracter da `string` representante da _key_, multiplica _hash_ por `33` (usado como _magic number_) e soma este valor com o valor ASCII do caracter.

Por fim, a função retorna o resto da divisão de _hash_ por outro número primo aleatório, que deve ser maior que o tamanho da _hash table_.

Essa, certamente, não é a melhor funcção _hash_ existente, mas é uma das mais recomendadas pela comunidade.

Há, ainda, alguma técnicas para criar _hash functions_ para chaves numéricas. Uma relação destas e algumas implementações podem ser encontradas neste [link](http://web.archive.org/web/20071223173210/http://www.concentric.net/~Ttwang/tech/inthash.htm)
