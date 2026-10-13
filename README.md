# pratica-docker-SD

Prática de Docker da disciplina de Sistemas Distribuídos.

## Respostas da prática

### 1. Os serviços precisam se comunicar. Como fazer isso?

Para que os serviços se comuniquem, não basta rodá-los isoladamente. A solução é criar uma rede virtual customizada no Docker (usando o comando `docker network create`) e anexar ambos os containers (a aplicação Flask e o banco de dados Redis) a essa mesma rede. Ao estarem na mesma rede interna, o Docker cuida do roteamento, permitindo a comunicação segura entre eles.

### 2. Por que utilizar o nome do serviço é mais adequado do que utilizar diretamente o IP do container?

Utilizar o endereço IP diretamente é inadequado porque os containers são efêmeros, ou seja, são destruídos e recriados com frequência. A cada nova inicialização, o Docker pode atribuir um IP diferente ao container do Redis. Se a aplicação dependesse do IP fixo, ela perderia a conexão após uma reinicialização. Utilizar o nome do serviço (ou do container) resolve o problema, pois o Docker atualiza dinamicamente o DNS interno, garantindo que o nome aponte sempre para o IP atualizado.

### 4.  Qual é a função do Dockerfile na aplicação? 

Sua função é definir o ambiente base (como a versão do Python), instalar as dependências necessárias (Flask e Redis), copiar o código-fonte para dentro do container e definir qual comando deve ser executado quando o container for iniciado. Ele garante que a aplicação execute de forma isolada e idêntica em qualquer computador.

### 6. Como o container Flask consegue encontrar o Redis sem conhecer seu endereço IP?

O container Flask encontra o Redis utilizando o recurso de DNS embutido (embedded DNS) que o Docker disponibiliza para redes customizadas. Quando a aplicação tenta acessar o host configurado como `redis` através da variável de ambiente, o servidor DNS do próprio Docker intercepta a requisição e resolve o nome do container para o endereço IP interno correto.

### 3. por que foi necessário utilizar `--build`?

Foi necessário utilizar a flag `--build` porque o código-fonte da aplicação foi alterado (a nova mensagem). Por defeito, se a imagem da aplicação (`app-flask`) já existir localmente, o comando `docker compose up -d` vai simplesmente reaproveitá-la e iniciar o container com o código antigo. A flag `--build` obriga o Docker Compose a ler o Dockerfile e a reconstruir a imagem do zero, garantindo que as alterações mais recentes no código sejam incorporadas.

### 5. Qual é a função do Redis nessa arquitetura?

Resposta sugerida: O Redis atua como a base de dados (ou sistema de armazenamento em memória) da aplicação. A sua função específica nesta arquitetura é guardar, atualizar e fornecer o número exato de acessos contabilizados pela aplicação Flask de forma rápida e centralizada.

### 7. Qual é a função do volume utilizado pelo Redis?

Resposta sugerida: A função principal do volume é garantir a persistência dos dados. Como os containers são efêmeros, todos os dados guardados dentro deles são perdidos quando o container é removido. Sem o volume, o contador de acessos voltaria a zero a cada reinicialização. O volume cria um ponto de montagem seguro no computador anfitrião, preservando a base de dados do Redis mesmo que o container seja destruído e criado novamente.

### 8. Qual problema o Docker Compose resolve em relação à execução manual dos containers?

Resposta sugerida: O Docker Compose resolve o problema da complexidade operacional e do trabalho manual repetitivo. Em vez de o programador ter de memorizar e digitar múltiplos comandos no terminal (como criar a rede manualmente, fazer o build, gerir portas, variáveis de ambiente e iniciar containers um a um na ordem certa), o Compose centraliza toda a infraestrutura num único ficheiro declarativo (`docker-compose.yml`). Isto permite subir e encerrar todo o ambiente de forma orquestrada com um único comando.