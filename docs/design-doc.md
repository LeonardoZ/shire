# Shire - Design Docs

- **Author**: Leonardo Henrique Zapparoli
- **Updated on:** 2023-08-18
- **Version:** 1


## Overview

Shire é um sistema de busca para dados internos de uma organização. A ideia é que as pessoas que compõe uma organização possam pesquisar por termos e serem direcionadas para os resultados. Os resultados podem ser arquivos em um servidor de arquivos, páginas web de aplicações intranet, conteúdos de bases de dados SQL, entre outros.

## Contexto

Esse documento explica o processo decisório na elaboração da arquitetura do Shire, definindo como a solução funciona e pode ser implementada.

## Objetivos

O documento estará completo quando os seguintes artefatos forem entregues:
- System design definido, junto de diagrama C4 Nível 1 e 2 da arquitetura da solução;
- ADR (Architecture Decision Records) justificando a escolha de stack usada na implementação.

## Non-goals 

O documento não irá abranger:
- Definição de requisitos de negócio ou relacionados.

## System design 

### Visão simplificada da solução


![Architecture](/docs/resources/shire-design-docs-simpli.drawio.png)

De forma que um sistema de busca de dados internos de uma organização seja implementado, precisamos de três classes de componentes básicos:

1. Componente de captura de dados, vamos chamar de Crawler
2. Componente de gestão dos dados, permitindo inserção e busca de dados capturados
3. Um banco de dados de índice invertido.

Um crawler é implementado conforme a fonte dos dados, como por exemplo páginas web ou filesystems, ou seja podemos ter vários crawlers para diferentes fontes de dados. O crawler tem como objetivo identificar, extrair e enviar os dados de um elemento capturado para serem salvos no componente de gestão de dados.

O componente de gestão de dados é quem vai receber os dados, realizando os tratamentos e validações adequadas em relação ao formato, e irá salvar em um banco de dados de índice invertido. 

No banco de dados de índice invertido, cada elemento (página web, arquivo) será salvo em um documento individual dentro de uma coleção. Esse tipo de banco de dados realiza classificação e rankeamento de resultados quando uma busca é executada (que será suficiente para o sistema atual)

### Visão da solução

![Diagrama de Contexto - C1](/docs/resources/c4-n1.png)

O diagrama acima ilustra a implementação desejada do nosso sistema.

Inicialmente, vamos considerar apenas crawlers de arquivos nessa versão inicial.

A solução possue dois componentes principais. Vamos expandir o diagrama acima em uma versão mais detalhada

![Diagrama de Containers - C2](/docs/resources/c4-n2.png)

#### Shire Filysystem Agent

Componentes responsáveis pela captura do conteúdo de arquivos de um filesystem.

O filesystem agent pode ser instalado em um servidor, e esse agent irá realizar o pooling do sistema de arquivos dos diretórios desajados, salvando de forma incremental, no Filesystem service, os arquivos que tiverem alguma alteração. Apenas algumas extensões de arquivos, como por exemplo .txt, .pdf, .docx serão considerados.

O processo deverá ocorrer de forma assíncrona, com a utilização de um sistema de filas. O Filesystem service, irá fazer a leitura da fila, solicitando ao agent o conteúdo do arquivo. Com o conteúdo em mãos, o service irá realizar o parse do arquivo, extraíndo os conteúdos textuais, e os irá salvar em uma fila, que será consumida pelo Search Service, no Search Context.

#### Shire

Componentes responsáveis pelaas features de busca. O Search service pode ser dividido em duas features principais:

Ingestão de dados: Através da fila, irá receber os resultados dos agents, validando e adequando o formato dos dados, para então persistir no BD de Índice invertido. Pode afetar os dados cacheados. Dessa forma, toda atualização, inserção ou deleção ocorre de forma assíncrona.

Busca: Realiza uma consulta no banco de dados de índice invertido, podendo cachear o resultado das consultas para otimizara busca no serviço.

#### Search UI

Frontend da aplicação, em que será feita a busca e serão apresentados os resultados.

### Stack de implementação

Para implementação da arquitetura acima, vamos adotar as seguintes tecnologias:

- Filesystem agent: Binário construído com a linguagem GO
- Filesystem Service e Search Service: Rest Apis implementadas com a linguagem NodeJs e o framework NestJS
- Queue: Serviço de filas do Redis, manipulado através da biblioteca BullMQ pelos Services
- Inverted Index DB: Elasticsearch
- Cache: Key-value store do Redis

## Cross-cutting concerns 
TODO

## Architecture Decision Records
TODO