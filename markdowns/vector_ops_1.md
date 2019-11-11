# Configuração do Host e SYCL Queue

Esse exercício é uma versão levemente modificada do adição de vetores que fizemos no exemplo passado. Nós vamos completar o código juntos desta vez.

> Não esqueça de incluir - `#include <CL/sycl.hpp>`.

## Configuração do Host

### Descrição

O primeiro passo é inicializar o vetor com os dados no host

Nós estaremos usando:

```cpp
cl::sycl::float4
```

O que é um alias para:

```cpp
cl::sycl::vec<float, 4>
```

É um tipo do SYCL que fornece funcionalidades de um vetor OpenCL para o host e o dispositivo.

### Tarefa

Defina 2 vetores de entrada e 1 vetor de saída

Entradas:
 - vector `a`: `{1.0, 1.0, 1.0, 1.0}`
 - vector `b`: `{1.0, 1.0, 1.0, 1.0}`

Saída:
- vector `c`: `{0.0, 0.0, 0.0, 0.0}`

Complete no código a configuração de memória do host:

```cpp
// <<Setup host memory>>
```

<details><summary>Hint</summary>
<p>

```cpp
sycl::float4 a = { 1.0f, 1.0f, 1.0f, 1.0f }; // input 1
```

</p>
</details>

## Inicializando a SYCL Queue

### Descrição

SYCL queue é construída de uma seleção de um dispotivo suportado.

O sistema é configurado para sempre forçar a execução de exemplos SYCL na CPU. De qualquer forma, o seletor padrão selecionaria
a CPU por causa das heurísticas para identificar as plataformas suportadas e os dispositivos no nosso sistema.

### Tarefa

Inicialize a SYCL queue com a CPU ou com o seletor de dispositivo padrão.

Localização no código fonte:

```cpp
// <<Setup SYCL queue>>
```

<details><summary>Dica</summary>
<p>

```cpp
sycl::queue myQueue(sycl::default_selector{}); 
// explicitly target the CPU: sycl::cpu_selector{}
```

</p>
</details>

# Run it!

@[Hello World from SYCL]({"stubs": ["src/exercises/vector-ops_1.cpp"],"command": "sh /project/target/run.sh vector-ops_1"})
