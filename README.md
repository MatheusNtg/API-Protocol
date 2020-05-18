# Lunatik State Manager API Protocol
## Operações oferecidas pela API

-   Criação de estados
-   Deleção de estados
-   Listagem de estados criados
-   Execução de código Lua a partir de um estado escolhido

Estruturas de dados necessárias para o atendimento das operaçõesUma estrura para encapsular o lua_State que será utilizado para todas as operações, uma forma de travar/destravar o lua_State que impossibilite múltipla execução de código em um determinado estado, um identificador único para cada estado, que possibilite rápida consulta.

```c
struct lunatik_state  
{  
	lua_State *L;  
	spinlock_t lock;  
	char *name; //Which represents a unique ID  
};
```
A estrutura acima representa um único estado lunatik, a API fica responsável por controlar estas estruturas em si, oferecendo somente uma interface de manipulação à mesma. Para armazenas várias destas estruturas (o que é interessante tendo em vista que um determinado usuário pode considerar criar múliplos estados) o uso de outra estrutura se faz necessário, algumas sugestões são:

-   Lista encadeada
-   Kernel Hash Table
-   Vetores

Dentre estas, a que se mostrou mais interessante até o presente momento é a Kernel Hash Table, pois, além de encontrar-se implementada no Kernel do Linux, oferece uma complexidade assintótica de limite superior igual as implementações utilizando listas encadeadas e vetores, dependendo somente da função de mapeamento. No entanto, em média, a consulta, assim como a inserção de elementos na hash table, possui uma complexidade assintótica constante, tornando-a assim uma ótima candidata para o armazenamento dos vários lunatik_state’s.

## Tratamento de erros oferecidos pela API

Refazer isso baseado nos erros utilizados pelo nflua

## Assinaturas e funcionamento das funções oferecidas pela API

As funções com o prefixo “klua_” (de kernel lua) são aquelas expostas e oferecidas pela API, o uso desse prefixo tem por objetivo identificar previamente as funções oferecidas pela API e consequentemente expostas ao usuário.  

`int klua_createstate(const char *name);`

Função que tem por objetivo criar um lunatik_state  
e retornar o resultado dessa operação. Recebe como único parâmetro o nome `name` do estado que servirá como identificador único para o estado lua respectivo ao lunatik_state.  

`int klua_deletestate(const char *name);`

Função que procura por um estado nomeado `name` na estrutura interna da API e caso exista o deleta.  

`void klua_liststates(void);`

Lista (no dmesg) os estados criados que se encontram armazenados na estrutura interna da API.  

`int klua_execute(const char *name, const char *code);`

Função de execução de código `code` escrito em Lua a partir da escolha de um estado `name` passado como argumento. Essa função pode gerar erro nos seguintes casos:

-   Estado desejado para executar o código encontra-se ocupado (em uso)
-   Estado passado como argumento não existe
