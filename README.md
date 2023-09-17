# kubernates
## Tutorial de Kurbenates
![image](https://github.com/Jilosofo/kubernates/assets/126982541/9f53f386-d0dd-4a47-8818-c6eead19b6a3)

*******
`Era da implantação tradicional:` No início, as organizações executavam aplicações em servidores físicos. Não havia como definir limites de recursos para aplicações em um mesmo servidor físico, e isso causava problemas de alocação de recursos. Por exemplo, se várias aplicações fossem executadas em um mesmo servidor físico, poderia haver situações em que uma aplicação ocupasse a maior parte dos recursos e, como resultado, o desempenho das outras aplicações seria inferior. Uma solução para isso seria executar cada aplicação em um servidor físico diferente. Mas isso não escalava, pois os recursos eram subutilizados, e se tornava custoso para as organizações manter muitos servidores físicos.

*******
`Era da implantação virtualizada:` Como solução, a virtualização foi introduzida. Esse modelo permite que você execute várias máquinas virtuais (VMs) em uma única CPU de um servidor físico. A virtualização permite que as aplicações sejam isoladas entre as VMs, e ainda fornece um nível de segurança, pois as informações de uma aplicação não podem ser acessadas livremente por outras aplicações.

A virtualização permite melhor utilização de recursos em um servidor físico, e permite melhor escalabilidade porque uma aplicação pode ser adicionada ou atualizada facilmente, reduz os custos de hardware e muito mais. Com a virtualização, você pode apresentar um conjunto de recursos físicos como um cluster de máquinas virtuais descartáveis.

Cada VM é uma máquina completa que executa todos os componentes, incluindo seu próprio sistema operacional, além do hardware virtualizado.

*******
`Era da implantação em contêineres:` Contêineres são semelhantes às VMs, mas têm propriedades de isolamento flexibilizados para compartilhar o sistema operacional (SO) entre as aplicações. Portanto, os contêineres são considerados leves. Semelhante a uma VM, um contêiner tem seu próprio sistema de arquivos, compartilhamento de CPU, memória, espaço de processo e muito mais. Como eles estão separados da infraestrutura subjacente, eles são portáveis entre nuvens e distribuições de sistema operacional.
*******
## Control kuerbenates:
* **etcd** Importante efetuar sempre backup onde salva todas informações do cluster

