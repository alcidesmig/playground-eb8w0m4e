## O que é SYCL?

O SYCL (pronunciado 'sícol') é uma camada de abstração cross-platform, isenta de royalties, que se baseia em conceitos existentes como, portabilidade e eficiência do OpenCL, que permite que o código para processadores heterogêneos seja escrito no estilo "single-source" usando apenas C++. O SYCL permite o desenvolvimento, em que as template functions do C++ podem conter tanto o código de host como do dispositivo para construir algoritmos complexos que usam a aceleração OpenCL e, reutilizá-los em todo o código-fonte em diferentes tipos de dados.

O SYCL é C++ padrão, portanto, não há extensões necessárias para o desenvolvimento.

Este tutorial tem como objetivo ensinar os fundamentos do SYCL através de aplicações simples.

Usaremos o ComputeCpp, uma implementação compatível com SYCL v1.2.1 para compilar e executar os aplicativos.

