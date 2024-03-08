# Referências e cópias de objetos

Uma das diferenças fundamentais entre objetos e primitivos é que objetos são armazenados e copiados por "referência", enquanto valores primitivos: strings, números, booleanos, etc -- são sempre copiados "como um valor integral".

Isso é fácil de entender se olharmos um pouco nos bastidores do que acontece quando copiamos um valor.

Vamos começar por um primitivo, como uma string.

Aqui fazemos uma cópia de `message` em `phrase`

```js
let message = "Hello!";
let phrase = message;
```
 
Como resultado nós temos duas variáveis independentes, cada uma armazenando a string `"Hello!".`

![](variable-copy-value.svg)

Um resultado bastante óbvio, certo?

Objetos não são assim.

**Uma variável à qual foi atribuida um objeto não armazena o próprio objeto, mas o seu "endereço em memória" - em outras palavras "uma referência" a ele**

Vejamos um exemplo de tal variável:

```js
let user = {
  name: "John",
};
```

E aqui está como ele é realmente armazenado na memória:

![](variable-contains-reference.svg)

O objeto é armazenado em algum lugar na memória (à direita na figura), enquanto a variável `user` (à esquerda) possui uma "referência" para ele.

Nós podemos pensar em uma variável objeto, como no exemplo `user`, como uma folha de papel com o endereço do objeto nela.

Quando realizamos ações com o objeto, por exemplo pegar a propriedade `user.name`, o interpretador do JavaScript olha para o que há naquele endereço e realiza a operação no próprio objeto.

Agora aqui está o motivo da importância.

**Quando uma variável objeto é copiada, a referência é copiada, mas o próprio objeto não é duplicado.**

Por exemplo:

```js no-beautify
let user = { name: "John" };

let admin = user; // copia a referência
```

Agora temos duas variáveis, cada uma armazenando uma referência para o mesmo objeto:

![](variable-copy-reference.svg)

Como você pode ver, ainda há um objeto, porém com duas variáveis referenciando ele.

Podemos usar qualquer uma das variáveis para acessar o objeto e modificar seu conteúdo:

```js run
let user = { name: 'John' };

let admin = user;

*!*
admin.name = 'Pete'; // alterado pela referência em "admin"
*/!*

alert(user.name); // 'Pete', as mudanças são visíveis pela referência em "user"
```

É como se tivéssemos um gabinete com duas chaves e usamos uma delas (`admin`) para acessá-lo e fazer mudanças. Então, se mais tarde usarmos a outra chave (`user`), ainda iremos estar abrindo o mesmo gabinete e podemos acessar os conteúdos alterados.

## Comparação por referência

Dois objetos são iguais apenas se eles são o mesmo objeto.

Por exemplo, aqui `a` e `b` referenciam o mesmo objeto, então eles são iguais: 

```js run
let a = {};
let b = a; // copia a referência

alert( a == b ); // true (verdade), ambas variáveis referenciam o mesmo objeto 
alert( a === b ); // true (verdade)
```

E aqui dois objetos independentes não são iguais, embora sejam parecidos (ambos são vazios):

```js run
let a = {};
let b = {}; // dois objetos independentes

alert( a == b ); // false (falso)
```

Para comparações como `obj1 > obj2` ou para comparações com um primitivo `obj == 5`, objetos são convertidos para primitivos. Iremos estudar como conversões de objetos funcionam muito em breve, mas para falar a verdade, tais comparações são raramente necessárias - normalmente elas aparecem como resultado de um erro de programação.

## Clonando e fundindo, Object.assign [#cloning-and-merging-object-assign]

Então, copiar uma variável objeto cria mais uma referência para o mesmo objeto.

Mas e se precisarmos duplicar um objeto?

Podemos criar um novo objeto e replicar a estrutura do objeto existente, iterando por suas propriedades e copiando elas a nível de primitivos.

Tipo assim:

```js run
let user = {
  name: "John",
  age: 30
};

*!*
let clone = {}; // o novo objeto vazio

// vamos copiar todas as propriedades de `user` para ele
for (let key in user) {
  clone[key] = user[key];
}
*/!* 

// agora `clone` é um objeto totalmente independente com o mesmo conteúdo
clone.name = "Pete"; // alterada a informação nele

alert( user.name ); // ainda John no objeto original
```

Também podemos usar o método [Object.assign](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) para isso.

A sintaxe é:

```js run
Object.assign(dest, ...fontes)
```

- O primeiro argumento `dest` é o objeto destino.
- Argumentos adicionais `fontes` são os objetos fonte.

Ele copia as propriedades de todos os objetos fonte para o destino `dest`, e o retorna como resultado.

Por exemplo, temos um objeto `user`, vamos adicionar algumas permissões à ele:

```js run
let user = { name: "John" };

let permissions1 = { canView: true };
let permissions2 = { canEdit: true };

*!*
// copia todas as propriedades de permissions1 e permissions2 para user 
Object.assign(user, permissions1, permissions2);
*/!*

// agora user = { name: "John", canView: true, canEdit: true }
```

Se o nome da propriedade copiada já existir, ela é sobrescrita:

```js run
let user = { name: "John" };

Object.assign(user, { name: "Pete" });

alert(user.name); // agora user = { name: "Pete" }
```

Podemos também utilizar o `Object.assign` para clonagem simples:

```js run
let user = {
  name: "John",
  age: 30
};

*!*
let clone = Object.assign({}, user);
*/!*

alert(clone.name); // John
alert(clone.age); // 30
```

Aqui ele copia todas as propriedade de `user` para o objeto vazio e o retorna.

Também há outros métodos para clonagem de objeto, por exemplo usando a [sintaxe de espalhamento](info:rest-parameters-spread) `clone = {...user}`, coberta mais tarde no tutorial.

## Clonagem aninhada

Ate agora assumimos que todas as propriedades de `user` são primitivas. Mas propriedades podem ser referências para outros objetos.

Tipo assim:

```js run
let user = {
  name: "John",
  sizes: {
    height: 182,
    width: 50
  }
};

alert( user.sizes.height ); // 182
```

Agora não é suficiente copiar `clone.sizes = user.sizes`, como `user.sizes` é um objeto, ele será copiado por referência. Portanto, `clone` e `user` irão compartilhar os mesmos tamanhos:

```js run
let user = {
  name: "John",
  sizes: {
    height: 182,
    width: 50
  }
};

let clone = Object.assign({}, user);

alert( user.sizes === clone.sizes ); // true (verdade), mesmo objeto

// user e clone compartilham tamanhos
user.sizes.width++;       // altere a propriedade em um lugar
alert(clone.sizes.width); // 51, veja o resultado no outro
```

Para consertar isso e tornar `user` e `clone` objetos verdadeiramente separados, devemos usar um laço que examina cada valor de `user[key]` e, caso seja um objeto, replica sua estrutura também. Isso é chamado de "clonagem profunda", ou "clonagem estruturada". Há o método [structuredClone](https://developer.mozilla.org/en-US/docs/Web/API/structuredClone) que implementa clonagem profunda.

### structuredClone

A chamada `structuredClone(object)` clona o `object` com todas as propriedades aninhadas.

Aqui está com podemos usá-lo em nosso exemplo:

```js run
let user = {
  name: "John",
  sizes: {
    height: 182,
    width: 50
  }
};

*!*
let clone = structuredClone(user);
*/!*

alert( user.sizes === clone.sizes ); // falso (false), objetos diferentes

// Agora, user e clone são completamente independentes
user.sizes.width = 60;    // altera uma propriedade em um lugar
alert(clone.sizes.width); // 50, o outro não é afetado
```

O método `structuredClone` pode clonar a maioria dos tipos de dados, como objetos, arrays e valores primitivos.

Ele também oferece suporte a referências circulares, quando uma propriedade de um objeto referencia o próprio objeto (diretamente ou através de uma cadeia de referências)

Por exemplo:

```js run
let user = {};
// vamos criar uma referência circular:
// user.me é uma referência a `user`
user.me = user;

let clone = structuredClone(user);
alert(clone.me === clone); // verdadeiro (true)
```

Como você pode ver, `clone.me` faz referência a `clone`, não a `user`! Então a referência circular foi clonada corretamente também.

No entanto, existem casos em que `structuredClone` falha.

Por exemplo, quando um objeto possui uma propriedade que é uma função:

```js run
// Erro
structuredClone({
  f: function () {},
});
```

Propriedades que são funções não são suportadas.

Para lidar com casos complexos, podemos precisar usar uma combinação de métodos de clonagem, escrever nossa própria lógica de clonagem personalizada ou, para não reinventar a roda, usar uma implementação existente, como [_.cloneDeep(obj)](https://lodash.com/docs#cloneDeep) da biblioteca JavaScript [lodash](https://lodash.com).

## Resumo

Objetos são atribuídos e copiados por referência. Em outras palavras, uma variável não armazena o "valor do objeto", mas uma "referência" (endereço em memória) para o valor. Portanto, copiar a variável ou passá-la como argumento de uma função copia essa referência, não o objeto em si.

Todas as operações feitas através de referências copiadas (como adição/remoção de propriedades) são realizadas no mesmo objeto único.

Para fazer uma "cópia real" (um clone) podemos usar o `Object.assign`, caracterizando a chamada "cópia rasa" (objetos aninhados são copiados por referência), uma função `structuredClone` de "clonagem profunda", ou usar uma implementação de clonagem personalizada, como o [_.cloneDeep(obj)](https://lodash.com/docs#cloneDeep).
