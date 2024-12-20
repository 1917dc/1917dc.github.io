+++
author = "Luiz"
title = 'Rust Ownership'
date = 2024-12-03T21:15:16-03:00
description = 'Stack & Heap in Rust'
draft = false
tags = ['rust', 'stack', 'heap']
categories = ['programming']
+++

- Ao invés de depender de garbage collection, que existem em linguagem de alto nível como Java e Python, ou de depender de alocação de memória explícita como C; o Rust executa isso de uma forma diferente: usando "ownership" que são checadas em tempo de compilação, caso alguma regra seja quebrada nesse sentido o programa não vai compilar.

** revisar ponteiros**

---

## Heap & Stack
The heap is less organized: when you put data on the heap, you request a certain amount of space. The memory allocator finds an empty spot in the heap that is big enough, marks it as being in use, and returns a _pointer_, which is the address of that location. This process is called _allocating on the heap_ and is sometimes abbreviated as just _allocating_ (pushing values onto the stack is not considered allocating). Because the pointer to the heap is a known, fixed size, you can store the pointer on the stack, but when you want the actual data, you must follow the pointer. Think of being seated at a restaurant. When you enter, you state the number of people in your group, and the host finds an empty table that fits everyone and leads you there. If someone in your group comes late, they can ask where you’ve been seated to find you.
  
Pushing to the stack is faster than allocating on the heap because the allocator never has to search for a place to store new data; that location is always at the top of the stack. Comparatively, allocating space on the heap requires more work because the allocator must first find a big enough space to hold the data and then perform bookkeeping to prepare for the next allocation.
Accessing data in the heap is slower than accessing data on the stack because you have to follow a pointer to get there. Contemporary processors are faster if they jump around less in memory. Continuing the analogy, consider a server at a restaurant taking orders from many tables. It’s most efficient to get all the orders at one table before moving on to the next table. Taking an order from table A, then an order from table B, then one from A again, and then one from B again would be a much slower process. By the same token, a processor can do its job better if it works on data that’s close to other data (as it is on the stack) rather than farther away (as it can be on the heap).
When your code calls a function, the values passed into the function (including, potentially, pointers to data on the heap) and the function’s local variables get pushed onto the stack. When the function is over, those values get popped off the stack.

---

- **Stack**: A stack precisa de valores de tamanho fixo, que são declarados na instanciação do dado, por exemplo um array, que necessariamente precisa ter um tamanho fixo para ser declarado, usa o método LIFO(Last In First Out), assim como na estrutura de pilha.

- **Heap**: A heap por outro lado tende a ser mais desorganizada, não necessariamente precisa de um tamanho fixo, aloca de forma avulsa onde a memória está livre, ou seja, usar a stack é mais rápida pelo simples fato dela não ter de verificar onde está ou não livre para alocações, ela só empurra o elemento para o topo da pilha ou remove. 

## Ownership rules:

- Cada variável tem um dono.
- Só pode ter um dono por vez.
- A variável é imediatamente descartada assim que o dono não existir mais.

```rust
fn main(){
	// test não é válido ainda
	// pois ainda não foi declarado
	{
		// a variável test existe dentro desse escopo 
		let test = "test";
	}
	// assim que esse escopo é concluído 
	// ela deixa de ser válida

	// * pensar no conceito de stack
}
```

- **Strings** (para entender melhor como funcionam a heap e a stack): Existe uma diferença entre `string literals` e `strings` propriamente ditas. Essa diferença pode não aparentar ser explícita, mas ela existe; quando declaramos um string literal estamos jogando o valor diretamente na  stack, pois a quantidade de memória a ser consumida já é de conhecimento do compilador e operações em tempo de runtime não vão precisar acontecer para cuidar desse valor em específico por ele ser imutável. Já com strings a coisa é totalmente diferente, já que não necessariamente sabemos a quantidade de memória que essa variável vai ocupar, temos que alocar ele na heap, ou seja, operações vão acontecer em tempo de runtime para que o valor da string possa ser mutável, por exemplo:

```rust
fn main(){

	// essa string é imutável em sua essência
	let string = String::from("foo");

	// também podemos fazer a string ser mutável:
	let mut string_mut = String::from("foo");

	// posso assim alterar o valor contido nessa string
	// nesse caso estou dando append em um string literal
	// dentro desse objeto.
	string_mut.push_str(" bar!"); 
	println!("{string_mut}");
}
```

```rust
fn main(){
	let s1 = String::from("foo");
	let s2 = s1;

	// erro vai estourar pois o "s1" já foi "enterrado"
	// e agora o valor que antes apontava para "s1" 
	// agora aponta para o "s2".
	println!("{s1}");
}
```

```rust
fn main(){
	let s1 = String::from("foo");
	let s2 = s1.clone();

	println!("foo = {s1}, bar = {s2}");
}
```

```rust
fn main() {
	// s comes into scope
    let s = String::from("hello");  

	// s's value moves into the function...
    takes_ownership(s);             

	// ... and so is no longer valid here
    let x = 5;                      // x comes into scope

	// x would move into the function,
    makes_copy(x);                  
	// but i32 is Copy, so it's okay to still
	// use x afterward
} // Here, x goes out of scope, then s. But because s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{some_string}");
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{some_integer}");
} // Here, some_integer goes out of scope. Nothing special happens.
```

```rust
fn main() {

	// gives_ownership moves its return
	// value into s1
    let s1 = gives_ownership();         

	// s2 comes into scope
    let s2 = String::from("hello");     

	// s2 is moved into
	// takes_and_gives_back, which also
	// moves its return value into s3
    let s3 = takes_and_gives_back(s2);  
                                        
                                        
} // Here, s3 goes out of scope and is dropped. s2 was moved, so nothing
  // happens. s1 goes out of scope and is dropped.

// gives_ownership will move its
fn gives_ownership() -> String {             
	// return value into the function
	// that calls it

	// some_string comes into scope
    let some_string = String::from("yours"); 

	// some_string is returned and
	// moves out to the calling
	// function
    some_string                              
}

// This function takes a String and returns one

fn takes_and_gives_back(a_string: String) -> String { 
	// a_string comes into
	// scope

	// a_string is returned and moves out to the calling function
    a_string  
}
```

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{s2}' is {len}.");
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() returns the length of a String

    (s, length)
}
```
