# Lunatik State Manager API Protocol
## Operations offered by the API

-   States creation
-   States deletion
-   Created states listing
-   Lua code execution from a choiced state

## Needed data structures to accomplish the operations

In order to accomplish the operations offered by the API, it was created the klua_state which represents a state from lunatik. To do what the API needs we need to have a way to storage multiples instances of `klua_state`s, for this purpose we have the varible called `node` which will be explained ahead, the lua state responsible for the execution of lua code is storaged in the variable `L`, `lock` is a variable responsable for the concorrency control, last but not least we have `name` which work as a unique indetifier for the state.

```c
struct klua_state  
{
	struct hlist_node node;
	lua_State *L;  
	spinlock_t lock;  
	char name[KLUA_MAX_NAMESIZE]; //Which represents a unique ID  
};
```
As stated above, we need to have a way to store multiples instances of `struct klua_state`s, for this we use a kernel data structure called kernel hash table. Since the kernel hash table is composed by an array of buckets which storages generic data, we need to have a pointer to the beggining of each bucket, for this we have the variable `node` present in the definition of `struct klua_state`. One hash function is used to map the name of the state to a integer which will serve as key for the state in the kernel hash table.

## Error handling offered by the API

Refazer isso baseado nos erros utilizados pelo nflua

## API functions signatures and working

The functions with the prefix "klua_" (from kernel in lua) are those exposed and offered by the API for the user.

`int klua_createstate(const char *name);`

Function to create a klua_state. Receives only one parameter `name` which is the name of the state to be created.

`int klua_deletestate(const char *name);`

Function to delete a state with the identifier `name`.

`void klua_liststates(void);`

List (at dmesg) current states in the Lunatik.

`int klua_execute(const char *name, const char *code);`

Function to execute a lua code in a given state `name`.
