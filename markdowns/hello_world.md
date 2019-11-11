# Hello World

Primeiro exercício que vamos fazer para mostrar como SYCL funciona 

## Including the SYCL Header File

Em todo início de arquivo precisamos colocar o header `CL/sycl.hpp`.

`#include <CL/sycl.hpp>`

## Configurando Host Storage

Na função main, nós iniciamos configurando o host storage para os dados que nós vamos operar. Nosso objetivo é calcular c = a + b, onde as variáveis são os vetores. SYCL provê  para nós o tipo vec<T, size>, que é um vetor escalar básico. Os parâmetros são o tipo do vetor `T` e o tamanho `size`. A ideia é usar mais como um vetor para geometria, e ele suporta um tamanho máximo de 16. Mas existem formas para se trabalhar com grandes quantidades de dados. Vamos usar float4, podemos declarar com vec<float, 4>. 

```
sycl::float4 a = { 1.0, 2.0, 3.0, 4.0 };
sycl::float4 b = { 4.0, 3.0, 2.0, 1.0 };
sycl::float4 c = { 0.0, 0.0, 0.0, 0.0 };
```

## Selecionando seu dispositivo

No SYCL existem muitas maneiras de selecionar o dispositivo que irá executar a aplicação. O seletor padrão do SYCL tenta selecionar o dispositivo mais apropriado no sistema. É possível criar um seletor customisável, mas como nós só temos um dispositivo vamos usar o default

`cl::sycl::default_selector selector;`

## Configurando uma SYCL Queue

Para que possamos enviar as tarefas para serem executadas no dispositivo nós precisamos criar uma `SYCL queue`. Vamos configurar a fila e passar nosso seletor então o seletor sabe qual dispositivo escolher quando rodar as tarefas.

`cl::sycl::queue myQueue(selector);`

## Configurando o Device Storage

Na maioria dos sistemas, o host e o dispositivo não compartilham memória física. Por exemplo, a CPU pode usar RAM e a GPU pode usar sua própria VRAM na matriz. O SYCL precisa saber quais dados serão compartilhados entre o host e os dispositivos.

Para esse fim, existem buffers SYCL. A classe <T, dims> do buffer é genérica sobre o tipo de elemento e o número de dimensões, que podem ser uma, duas ou três. Quando passado um ponteiro bruto, como neste caso, o construtor buffer (T * ptr, tamanho do intervalo) assume a propriedade da memória pela qual foi passado. Isso significa que absolutamente não podemos usar essa memória enquanto o buffer existe, e é por isso que começamos um escopo de C ++. No final de seu escopo, os buffers serão destruídos e a memória retornada ao usuário. O argumento size é um objeto de intervalo <dims>, que precisa ter o mesmo número de dimensões que o buffer e é inicializado com o número de elementos em cada dimensão. Aqui, temos uma dimensão com um elemento.

Os buffers não estão associados a uma fila ou contexto específico, portanto, eles são capazes de manipular dados de forma transparente entre vários dispositivos. Eles também não exigem informações de leitura / gravação, pois isso é especificado por operação.
```
sycl::buffer<sycl::float4, 1> a_sycl(&a, sycl::range<1>(1));
sycl::buffer<sycl::float4, 1> b_sycl(&b, sycl::range<1>(1));
sycl::buffer<sycl::float4, 1> c_sycl(&c, sycl::range<1>(1));
```

## Executando o Kernel

### Criando Um Grupo De Comandos

The whole thing is technically a single function call to `queue::submit`. `submit` accepts a function object parameter, which encapsulates a command group. For this purpose, the function object accepts a command group handler constructed by the SYCL runtime and handed to us as the argument. All operations using a given command group handler are part of the same command group.

`myQueue.submit([&](cl::sycl::handler &cgh)`

A command group is a way to encapsulate a device-side operation and all its data dependencies in a single object by grouping all the related commands (function calls). Effectively, what this achieves is preventing data race conditions, resource leaking and other problems by letting the SYCL runtime know the prerequisites for executing device-side code correctly.

### Data Accessors

In our command group, we first setup accessors. In general, these objects define the inputs and outputs of a device-side operation. The accessors also provide access to various forms of memory. In this case, they allow us to access the memory owned by the buffers created earlier. We passed ownership of our data to the buffer, so we can no longer use the float4 objects, and accessors are the only way to access data in buffer objects.

```
auto a_acc = a_sycl.get_access<sycl::access::mode::read>(cgh);
auto b_acc = b_sycl.get_access<sycl::access::mode::read>(cgh);
auto c_acc = c_sycl.get_access<sycl::access::mode::discard_write>(cgh);
```

The buffer::get_access(handler&) method takes an access mode argument. We use access::mode::read for the arguments and access::mode::discard_write for the result. discard_write can be used whenever we write to the whole buffer and do not care about its previous contents. Since it will be overwritten entirely, we can discard whatever was there before.

The second parameter is the type of memory we want to access the data from. We will see the available types of memory in the section on memory accesses. For now we use the default value.

### Defining a Kernel Function

In SYCL there are various ways to define a kernel function that will execute on a device depending on the kind of parallelism you want and the different features you require. The simplest of these is the `cl::sycl::handler::single_task` function, which takes a single parameter, being a C++ function object and executes that function object exactly once on the device. The C++ function object does not take any parameters, however it is important to note that if the function object is a lambda it must capture by value and if it is a struct or class it must define all members as value members.

```
   cgh.single_task<class vector_addition>([=] () {
      c_acc[0] = a_acc[0] + b_acc[0];
   });
});
```

## Cleaning Up

One of the features of SYCL is that it makes use of C++ RAII (resource aquisition is initialisation), meaning that there is no explicit cleanup and everything is done via the SYCL object destructors.
```
{
   ...
}
```

# Run it!

@[Hello World from SYCL]({"stubs": ["src/exercises/hello_world.cpp"],"command": "sh /project/target/run.sh hello_world", "layout": "aside"})
