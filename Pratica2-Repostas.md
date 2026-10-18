Questão 1: O que representa o nó apresentado pelo comando kubectl get nodes? 

Representa a máquina virtual (ou conteiner Docker) criada pelo Minikube para simular o cluster. Num ambiente de teste como o Minikube, este nó único atua simultaneamente como o plano de controle (Control Plane), que gere todo o sistema, e como o nó de trabalho (Worker Node), onde os seus conteineres da aplicação e do Redis irão rodar na prática.

Questão 2: Por que a aplicação deve utilizar redis em REDIS_HOST, em vez do endereço IP de um Pod Redis?   

No Kubernetes, os Pods são temporários, o que significa que podem ser destruídos e recriados a qualquer momento (devido a falhas ou atualizações). Sempre que um Pod é recriado, recebe um novo endereço IP. Se utilizássemos o IP diretamente, a aplicação perderia a ligação. Ao usar o nome redis, o Kubernetes utiliza o seu DNS interno para encaminhar automaticamente o tráfego para o IP atualizado do Pod do Redis através do Service.

Questão 3: Por que a aplicação possui 3 réplicas enquanto o Redis possui apenas 1 réplica nesta prática?

A aplicação Flask é stateless (sem estado), ou seja, não guarda dados localmente. Por isso, podemos ter várias cópias (réplicas) executando simultâneamente para distribuir o tráfego (balanceamento de carga) sem gerar conflitos. O Redis, por outro lado, é stateful (com estado), pois é ele que guarda a contagem. Se tivéssemos múltiplas réplicas do Redis sem uma configuração complexa de sincronização de dados (clusterização), cada Pod do Redis teria uma contagem diferente, gerando inconsistência nos resultados.

Questão 4: Qual é a função do Service do Redis?

A função do Service do Redis é criar um ponto de acesso fixo e estável (com um nome DNS interno e um IP virtual estático) para o Pod do Redis. Isto garante que a aplicação Flask consiga sempre comunicar com o banco de dados, independentemente de quantas vezes o Pod do Redis seja reiniciado ou mude de IP.

Questão 5: Como o Service contador sabe quais Pods devem receber as requisições?

Service utiliza um mecanismo chamado selector. Na declaração do Service, definimos o seletor app: contador. O Service procura constantemente no cluster por todos os Pods que tenham exatamente essa label (rótulo) configurada. Quando as requisições chegam ao Service, ele distribui o tráfego de forma igualitária apenas pelos Pods que correspondam a esse rótulo.

Questão 6: O que aconteceu com os Pods quando o número de réplicas foi alterado de 3 para 5 e depois para 2?

Quando alteramos para 5 réplicas, o Kubernetes iniciou rapidamente 2 novos Pods para atender ao estado desejado. Ao reduzirmos para 2, ele encerrou (colocou em status Terminating) os 3 Pods excedentes. Isso demonstra a capacidade de escalabilidade da aplicação, permitindo aumentar ou diminuir os recursos de forma dinâmica de acordo com a demanda.

Questão 7: O Pod removido foi simplesmente perdido? Explique o que aconteceu após sua exclusão.

O Pod removido foi de fato encerrado, mas não houve perda na disponibilidade do sistema graças ao recurso de Self-Healing (autorrecuperação) do Kubernetes. Assim que o Pod foi deletado, o controlador do Deployment detectou que o estado atual (menos Pods do que o exigido) não correspondia ao estado desejado. Imediatamente, ele iniciou a criação de um novo Pod para substituir o excluído, garantindo que o número correto de instâncias continuasse rodando.

Questão 8: O que caracteriza um Rolling Update? Por que ele é útil em aplicações que precisam permanecer disponíveis durante uma atualização?

Um Rolling Update caracteriza-se pela substituição gradual das instâncias (Pods) da versão antiga pelas da nova versão, em vez de desligar todas as instâncias simultaneamente. Ele é extremamente útil porque garante que não haja tempo de inatividade (zero downtime). Durante a atualização, o Kubernetes mantém a aplicação disponível roteando o tráfego para os Pods antigos enquanto os novos estão sendo iniciados e validados, assegurando uma transição contínua e transparente para o usuário.

Questão 9: Qual é a função do comando kubectl rollout undo?

A função do comando kubectl rollout undo é executar um rollback, ou seja, desfazer uma atualização e reverter o Deployment imediatamente para a sua revisão anterior. Ele funciona como um botão de emergência: se uma nova versão implantada apresentar falhas ou instabilidades, esse comando restaura a versão antiga que operava de forma estável, mitigando rapidamente o problema.