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

Tecnicamente uma única chamada de função para `queue :: submit`. `submit` aceita um parâmetro de objeto de função, que encapsula um grupo de comandos. Para esse propósito, o objeto da função aceita um manipulador de grupo de comandos construído no tempo de execução. Todas as operações usando um determinado manipulador de grupo de comandos fazem parte do mesmo grupo de comandos.

`myQueue.submit([&](cl::sycl::handler &cgh)`


### Data Accessors

No nosso grupo de comandos, nós primeiro configuramos os acessors. No geral, esses objetos definem as entradas e as saídas de uma operação device-side. Os accessors também provém acesso a várias formas de memória. Nesse caso, eles permitem que nós acessemos a memória possúida pelas buffers criados anteriormente. Passamos a propriedade de nossos dados para o buffer, para que não possamos mais usar os objetos float4, e os acessadores são a única maneira de acessar dados em objetos de buffer.

```
auto a_acc = a_sycl.get_access<sycl::access::mode::read>(cgh);
auto b_acc = b_sycl.get_access<sycl::access::mode::read>(cgh);
auto c_acc = c_sycl.get_access<sycl::access::mode::discard_write>(cgh);
```

O método buffer::get_access(handler&) usa um argumento do modo de acesso. Usamos access::mode::read para os argumentos e access:: mode::discard_write para o resultado. discard_write pode ser usado sempre que escrevemos em todo o buffer e não nos importamos com o conteúdo anterior. Como ele será totalmente sobrescrito, podemos descartar o que havia antes.

### Definindo uma Kernel Function

No SYCL, existem várias maneiras de definir uma kernel function que será executada em um dispositivo, dependendo do tipo de paralelismo desejado e dos diferentes recursos necessários. A mais simples delas é a função `cl::sycl::handler::single_task`, que usa um único parâmetro, sendo um objeto de função C ++ e executa esse objeto de função exatamente uma vez no dispositivo. O objeto da função C++ não aceita nenhum parâmetro, no entanto, é importante observar que, se o objeto da função for um lambda, ele deverá capturar por valor e, se for uma estrutura ou classe, deverá definir todos os membros como membros do valor.

```
   cgh.single_task<class vector_addition>([=] () {
      c_acc[0] = a_acc[0] + b_acc[0];
   });
});
```

## Limpando

Um dos features do SYCL é a utilização de C++ RAII (resource aquisition is initialisation), significando que não há limpeza explícita e tudo é feito por meio dos destruidores de objetos SYCL.
   
```
{
   ...
}
```

# Run it!

@[Hello World]({"stubs": ["src/exercises/hello_world.cpp"],"command": "sh /project/target/run.sh hello_world", "layout": "aside"})
