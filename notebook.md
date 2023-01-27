# Jornada DevOps de Elite

## Aula 1

- 5 aulas com desafios
- Aprender DevOps requer método, e não tutoriais esparços. É necessário conhecer os fundamentos.
- Cultura DevOps: modelo CALMS (Culture, Automation, Lean, Measurement and Sharing)
  - Cultura: mudança de pensamento nas empresas, promovendo colaboração entre as equipes e autonomia aos times de infra e desenvolvimento.
    - Culpa compartilhada, ou não tem culpa, todos são responsáveis e estão aprendendo
  - Lean: gestão ágil, processo dinâmico de entrega ao cliente
  - Métricas: ajudam na otimização do processo e do próprio software
  - Sharing: diálogos como o que é feito, como é reparado, justificativa de uso de determinada ferramenta. Equipes trocam conhecimento e experiências.

- Método waterfall (cascata) distancia muito o projeto da entrega final ao cliente.
- Engenheiro DevOps é uma das profissões mais procuradas no Brasil. E há poucos no mercado.
  - O valor que este profissional agrega é essencial: empresas estão migrando para a nuvem
- DevOps não é um cargo, mas uma filosofia.
  - Seus conhecimentos não são restritos a quem trabalha exatamente com isso, mas se estende a devs e QAs, que precisam aprender.

### Docker

- Como já sei, não vou fazer muitas anotações.
- Arquitetura do Docker
  - Docker daemon: gerencia os objetos do Docker (containers, imagens, redes e volumes). É onde os containers são executados.
  - Docker client (CLI) - interface de controle do docker daemon. Não precisa estar na mesma máquina que ele.
  - Docker registry: repositório que armazena as imagens.

#### Principais comandos

- ```bash
  docker container run hello-world
  ```

  - Executa um container. Faz pull da imagem se ela não existit localmente.
  - `--name` para dar um nome ao container (senão o Docker cria um aleatório)
  - `--rm` para que o container se exclua automaticamente ao ser parado
  - `--it` para o modo interativo
  - `-d` para rodar em background, sem travar o terminal.

- ```bash
  docker container ls
  ```

  - Exibe os containers em execução
  - `-a` para também mostrar os containers parados

- ```bash
  docker container rm <id>
  ```

  - Inserir o id ou o nome para remover o container. 
  - Ele precisa estar parado para isso
  - `-f` para forçar a remoção de um container que está rodando.

- ```bash
  docker container run -it -rm ubuntu /bin/bash
  ```

  - Entrar no modo interativo

- ```sh
  docker container exec -it <id> /bin/bash
  ```

  - Acessar o terminal de um container em execução.

- ```sh
  docker container run -d -p 8080:80 nginx
  ```

  - Port binding `minha-porta:porta-container`

- ```sh
  docker container run -d -p 5432:5432 -e POSTGRES_PASSWORD=12345 -e POSTGRES_USER=admin --name banco-dados-1 postgres
  ```

  - Devemos passar as variáveis de ambiente com `-e`
  - Ver a documentação oficial da imagem
  - Nas nossas imagens personalizadas, é possível criar variáveis de ambiente.

- ```dockerfile
  FROM ubuntu
  RUN apt update
  RUN apt install curl --yes
  ```

- ```sh
  docker build -t ubuntu-curl .
  ```

  - ```sh
    docker container run -it ubuntu-curl /bin/bash
    ```

    - A imagem já vai ter o curl instalado.
    - `curl https://google.com`

- ```sh
  docker image ls
  ```

- A construção de uma imagem é feita por camadas. Ao executar um novo build, somente as camadas necessárias são refeitas.

  - Cada comando no Dockerfile é uma camada.
  - A imagem é sempre read-only
  - Alterar a ordem das camadas faz o cache ser perdido e as camadas inferiores são refeitas.

- **Cuidado**. No exemplo acima, `apt update` e os comandos de instalação estão em linhas diferentes. Neste caso, o *cache pode te prejudicar*, pois o `apt update` não será executado a cada build, e o pacote pode não ser encontrado por ser muito recente. É boa prática adicionar o `apt update` antes de cada comando de instalação.

  - ```dockerfile
    FROM ubuntu
    RUN apt update && apt install git vim curl --yes
    ```

- ```sh
  docker image prune
  ```

  - Remove as imagens desnecessárias que foram criadas durante o build

  - Como a imagem é um filesystem read-only, e o container é uma camada de leitura e escrita acima desta imagem, não é possível remover uma imagem que está sendo utilizada por um container.

  - ```sh
    docker image rm nginx ubuntu
    ```

- ![image-20230127131222533](notebook.assets/image-20230127131222533.png)

#### Docker registry

- Repositório de imagens. Ex.: DockerHub, AWS ECR, Gitlab CR

- ```
  docker login
  ```

- Importante: padrão de nomenclatura de imagens do DockerHub

  - `namespace/repositorio:versao`
  - `otaviodioscanio/api-conversao:latest`

- É boa prática sempre criar e subir a tag latest.

- ```sh
  docker build -t otaviodioscanio/ubuntu-curl:v1 .
  docker tag otaviodioscanio/ubuntu-curl:v1 otaviodioscanio/ubuntu-curl:latest
  ```

- ```sh
  docker push otaviodioscanio/ubuntu-curl:v1
  docker push otaviodioscanio/ubuntu-curl:latest
  ```

  - Envia as camadas inexistentes no DockerHub

### Projeto: Conversão de Temperatura

- https://github.com/KubeDev/conversao-temperatura

- ```dockerfile
  # Imagem base sempre deve ser versionada, para garantir sempre o mesmo comportamento se a latest mudar.
  FROM node:18.11.0
  # Criar um diretório e entrar em seguida
  WORKDIR /app 
  # Instalar os pacotes do Node JS (que estão no package.json). Copiamos o package.json separado dos demais arquivos.
  COPY package*.json ./
  RUN npm install
  # Copiar depois porque é mais comum modificar o código fonte do que as dependências
  COPY . .
  EXPOSE 8080
  CMD ["node", "server.js"]
  ```

- > sudo chown -R otavio ../../

- ```dockerfile
  docker build -t otaviodioscanio/conversao-temperatura:v1 .
  ```

  