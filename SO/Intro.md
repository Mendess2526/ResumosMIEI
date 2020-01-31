# Contextualização

As histórias do Moura têm como objectivo justificar e contextualizar as decisões
tomadas sobre a evolução dos Sistemas Operativos. Infelizmente, o resultado
obtido é uma amálgama de informação com muito pouca utilidade.

Este capitulo tem como objectivos principais *resumir o contexto histórico* e
*apresentar o papel do sistema operativo e sua definição*.

# Contexto Histórico

Os primeiros sistemas informáticos não tinham sistema operativo, nem nada
semelhante, apenas quando o hardware começou a melhorar se começou a pensar em
optimizações a nível do software. Foram criados os primeiros programas que
simplificaram as operações do computador.

Estes programas, chamados de `monitores de controlo` permitem ao utilizador
carregar programas para a memória, editá-los e verificar a sua execução.

Cada utilizador tem um determinado tempo atribuído só para si, podem utilizar o
computador exclusivamente.

**Durante maior parte do tempo o computador está parado a espera do input do
utilizador ou de operações Input/Output.**


Seguiram-se então os sistemas de `tratamento em batch`. Sendo que **maior parte
do tempo de processador era gasto em operações I/O (espera activa)**, a solução
foi separar os periférico de I/O.
Começaram também a surgir os primeiros dispositivos de memória secundária.

Um computador auxiliar lia para a banda magnética os programas a executar,
quando o trabalho em curso terminasse o sistema operativo ia a lista de trabalho
e seleccionava o próximo a executar. Alem disso, em vez de imprimir o resultado
dos programas diretamente, os ficheiros são enviados para uma impressora quando
acabam de executar.

Os periféricos avisam o processador no fim da sua exeução, através de
interrupções.


A segurança do sistema operativo é assegurada pela gestão de memória. Nem todas
as regiões de memória podem ser acedidas pelo user, existem portanto dois niveis
`user space` e `kernel space`.

## Escalonamento

Este sistema simples ja permite mecanismos de optimização de gestão do
computador. Um utilizador pode indicar a duração máxima de uma operação, ou os
recursos utilizados. As operações poderão ser escolhidas com base no tempo de
execução. Vamos analisar estar 'escolha' mais à frente.

## Interrupções

Interrupções/excepções podem ser geradas tanto a partir do software como do
hardware.

Alguns exemplos:
1. Periféricos I/O
  * Quando acabam as suas operações "avisam" o sistema operativo através de uma
    chamada ao sistema (que gera uma interrupção).
2. Excepções
  * Quando uma instrução executa uma operação inválida, e.g: dividir por zero,
    aceder a uma zona de memória não permitida.
3. Timers do CPU.

A interrupção transfere o contexto de execução para a rotina apropriada que vai
lidar com o "problema". Visto que apenas algumas interrupções são permitidas
existe uma tabela de apontadores, chamada de `interrupt vector` para as rotinas
de tratamento de excepções. Cada rotina é chamada indiretamente através da
tabela, sem nenhuma rotina/programa intermédia/o. Lidar com interrupções fica
menos custoso, em termos de tempo, para o sistema operativo.

O controlo de interrupções é mais ou menos o seguinte:
1. Um processo do utilizador gera uma interrupção e o Program Counter e o estado
   do processador são guardados num stack especial do kernel, ficando o processo
   bloqueado.
2. O controlador de interrupções decide que tipo de interrupção foi gerada e
   chama a rotina apropriada do kernel para lidar com ela.
3. O estado geral do processador é guardado (visto que mudamos de contexto).
4. O kernel executa a rotina apropriada para lidar com a interrupção.
5. Se esta rotina foi rapida o kernel volta para o proesso que estava a
   executar, se não o processo entra para a lista de processos candidatos a CPU.
  * Assumindo que o processo não foi morto
6. O Estado do processo é carregado para o registos e o controlo é novamente
   transferido para o processo.

De notar que o CPU muda o contexto entre user space e kernel space, onde todas
as operações são permitidas. Esta mudança é feita internamente pelo hardware.

Dado as excepções serem assíncronas tem que haver forma de lidar com
aparecimento de várias excepções em simultaneo. Este controlo é feito com dois
mecanismos.
O primeiro inibe interrupções depois de uma ter sido aceite. Assim se uma
interrupção tiver a ser tratada, nao é possível mudança de contexto ao aparecer
outra.
O segundo inibe interrupções dado a sua gravidade, podendo interromper rotinas
de tratamento de excepções com menor prioridade.

O returno a modo de user é feito pela instrução de terno de interrupção  `RIT`.

## Multiprogramação

O mecanismo de interrupções permite multiplexar o processador.
A capacidade de alternar a execução **não** é limitada a `um programa e
periféricos I/O` mas pode ser estendida a **vários programas em memória**.

Um programa que queira aceder a um ficheiro em disco fica bloqueada enquanto o
controlador de disco actua. Durante este tempo outro programa pode ser executado
pelo processador.
Desta forma conseguimos optimizar a utilização do processador, tendo sempre algo
para fazer.

## Memória virtual

O CPU carrega instruções apenas a partir da memória, o que impllica que para que
os programas corram tenham que estar em memória. A grande maioria dos programas
correm os programas numa memória volatil, a RAM, que vai ser chamada de memória
principal.

Esta, e outras formas de memória, disponibilizam um `array de bytes`. Cada byte
tem o seu endereço. Os bytes interegarem entre si atravez de instruções de
`load` ou `store` referidas a endereços de memória especificos.
A `load` move um byte, ou uma `word`, da memória principal para um registo do
CPU e a `store` faz a operação inversa.
De notar que a unidade de memória apenas 've' uma série de endereços de memória,
é totalmente agnostica à forma como foram criados ou o seu conteúdo.

Idealmente os `processos` estariam todos presentes na memória virtual mas isso
não é possivel.

Surge então a memória virtual que simula um espaçõ de memória completo para cada
`processo` sendo que na realidade apenas parte do `processo` se encontra na
`memória principal`. Vai ser mais analisada mais a frente.

# Papel do Sistema Operativo

O papel do sistema operativo pode ser visto de várias ópticas.

## 1. Gestor de recursos
O sistema operativo gere recursos lógicos, mapeando-os para os seus
correspondentes físicos;
  * Ficheiros <-> Espaço em disco;
  * Processos <-> Processador;
  * Periféricos virtuais <-> Periféricos físicos.

Desta forma o sistema operativo torna os recursos lógicos **genéricos** e
reutilizáveis;

Lista de recursos lógicos geridos pelo sistema operativo;
 1. Processos (estudados detalhadamente nos próximos capitulos);
 2. Memória virtual (estudada também mais a frente);
    * Gestão de espaços de endereçamento.
 3. Sistema de ficheiros;
 4. Periféricos;
    * Gere qualquer periférico através de `periféricos digitais`, obtendo um
      grau de abstracção.
 5. Utilizadores.
    * Identidade;
    * Privilégios.

## 2. Interface
 * Interface operacional;
   * Shell (bash, zsh, etc);
   * GUI (Linux, Apple, Windows).
* System calls
   * Permitem executar operações associadas aos objectos do sistema;
   * Os objectos do sistema representa os recrusos lógicos e os seus metodos
     associados.
   * Fazem parte do modelo computacional dos sistemas opertativos.

## 3. Máquina virtual
 * O sistema operativo é uma maquina virtual sobre a máquina fisica.
 * Facilidade na sua utilização.

# Definição de Sistema Operativo

Como reparamos podemos ver o papel do Sistema Operativo de vários angulos
levando a várias definições, todas correctas.

Podemos pensar no sistema operativo como uma máquina virtual que abstrai os
detalhes da máquina física, disponibiliza também um conjunto de recursos lógicos
e interfaces para os gerir e programar.
