# Iniciando com SYCL

## Como o SYCL funciona?

Uma implementação do SYCL consiste em dois componentes principais; um compilador SYCL, que compila seu código para dispositivos OpenCL, e uma biblioteca runtime, os quais fornecem uma interface de alto nível para escrever aplicações SYCL

## No que o SYCL é executado?

SYCL pode ser executado em uma grande gama de dispositivos OpenCL em qualquer sistema dado como: multi CPUs, GPU, FPGAs, DSPs e outros tipos de acelerados e processadores especializados. Por exemplo, a implementação que vamos utilizar neste tutorial permite que SYCL seja executado em Intel, AMD, ARM, Renesas e NVIDIA.

O pacote `computecpp` fornece algumas ferramentas que vão ser utilizadas, como `computecpp_info` que checa se o dispotivo possui suporte para a compilação e execução das aplicações.

A instalação e configuração do `computecpp` pode ser um pouco demorada dependendo dos dispositivos que você possui, por isso vamos utilizar este docker já configurado para a execução das aplicações.

Para fins de demonstração vamos executar aqui e analisar o output:

@[computecpp_info]({"command": "sh /project/target/validate.sh"})

* Este tutorial será inteiramente feito CPU Intel.

O comando `computecpp_info` retorna algumas informações, mas o que importa para nós é a seguinte linha:

`Device is supported                     : YES - Tested internally by Codeplay Software Ltd.`

Podemos ver também que o único device suportado é o "host" que é a CPU Intel;

