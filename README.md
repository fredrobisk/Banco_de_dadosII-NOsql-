# Sistema de Eventos Academicos - MongoDB

Este trabalho modela uma base de dados para armazenar eventos academicos, seus organizadores e seus palestrantes.

O modelo possui tres entidades:

- `ORGANIZADOR`
- `EVENTO`
- `PALESTRANTE`

Relacionamentos:

- `ORGANIZADOR 1:N EVENTO`: um organizador pode organizar varios eventos, e cada evento pertence a um organizador.
- `EVENTO N:N PALESTRANTE`: um evento pode ter varios palestrantes, e um palestrante pode participar de varios eventos.

## Checklist do enunciado

- Modelo ER com 3 entidades.
- Pelo menos um relacionamento `1:N`.
- Pelo menos um relacionamento `N:N`.
- Versao 1 usando `Embedded Relationships`.
- Versao 2 usando `Referenced Relationships`.
- Insercao de dados nas duas versoes.
- Duas consultas para cada versao.
- Todas as consultas envolvem pelo menos duas entidades e possuem restricao.
- Pelo menos uma consulta considera o relacionamento `N:N`.

## Modelo ER

```mermaid
erDiagram
    ORGANIZADOR ||--o{ EVENTO : "organiza (1:N)"
    EVENTO }o--o{ PALESTRANTE : "possui_palestrantes (N:N)"

    ORGANIZADOR {
        int id_organizador PK
        string nome_organizador
        string email
    }

    EVENTO {
        int id_evento PK
        string nome_evento
        string area
        date data_evento
    }

    PALESTRANTE {
        int id_palestrante PK
        string nome_palestrante
        string especialidade
    }
```

## Executar MongoDB com Docker

Esta secao mostra os comandos para executar o MongoDB usando Docker. Para a apresentacao, o objetivo e deixar o MongoDB rodando e depois abrir o `mongosh` para colar os comandos das versoes embedded e referenced.

### Verificar se o Docker esta funcionando

Abra o Docker Desktop e espere ele terminar de carregar. Depois, no PowerShell, execute:

```bash
docker --version
```

```bash
docker ps
```

Se `docker ps` executar sem erro, o Docker esta pronto.

### Baixar imagem MongoDB

```bash
docker pull mongodb/mongodb-community-server:latest
```

### Criar container MongoDB

```bash
docker run --name mongodb-eventos -p 27017:27017 -d mongodb/mongodb-community-server:latest
```

### Verificar se o container esta rodando

```bash
docker ps
```

Deve aparecer um container com o nome `mongodb-eventos` e a porta `27017`.

### Abrir o MongoDB Shell dentro do container

Use este comando se voce nao tiver `mongosh` instalado no Windows:

```bash
docker exec -it mongodb-eventos mongosh
```

Depois disso, cole os comandos das secoes `Versao 1: Embedded Relationships` e `Versao 2: Referenced Relationships`.

### Abrir o MongoDB Shell pelo Windows

Use este comando somente se o `mongosh` estiver instalado no Windows:

```bash
mongosh --port 27017
```

### Comandos uteis do Docker

Parar o container:

```bash
docker stop mongodb-eventos
```

Iniciar novamente:

```bash
docker start mongodb-eventos
```

Entrar novamente no MongoDB:

```bash
docker exec -it mongodb-eventos mongosh
```

Ver containers rodando:

```bash
docker ps
```

Ver todos os containers, incluindo parados:

```bash
docker ps -a
```

Remover o container, caso precise recriar:

```bash
docker stop mongodb-eventos
docker rm mongodb-eventos
```

### Se o Docker disser que o container ja existe

Se este comando:

```bash
docker run --name mongodb-eventos -p 27017:27017 -d mongodb/mongodb-community-server:latest
```

mostrar erro dizendo que o nome `mongodb-eventos` ja existe, use:

```bash
docker start mongodb-eventos
docker exec -it mongodb-eventos mongosh
```

### Se o Docker Desktop ficar carregando infinitamente

Abra o PowerShell como Administrador e tente:

```bash
wsl --shutdown
net stop com.docker.service
net start com.docker.service
```

Depois abra o Docker Desktop novamente.

Se continuar travando, tente atualizar/instalar o WSL:

```bash
wsl --update
wsl --install -d Ubuntu
```

Depois reinicie o computador e abra o Docker Desktop novamente.

### Alternativa sem Docker

Se o Docker nao funcionar no dia da apresentacao, instale o MongoDB diretamente no Windows:

- MongoDB Community Server
- MongoDB Shell (`mongosh`)

Depois abra o PowerShell e execute:

```bash
mongosh
```

A partir disso, os comandos MongoDB deste README continuam iguais.

## Versao 1: Embedded Relationships

Na versao embedded, os dados relacionados ficam dentro do documento principal.

Neste caso, a colecao principal sera `eventos`. Cada evento tera o organizador embutido e uma lista de palestrantes embutida.

Vantagem: consultas mais simples para buscar evento, organizador e palestrantes juntos.

Desvantagem: pode gerar duplicacao de dados, principalmente quando o mesmo palestrante participa de mais de um evento.

### Criar banco da versao embedded

```javascript
use eventos_embedded
```

### Criar colecao

```javascript
db.createCollection("eventos")
```

### Inserir documentos

```javascript
db.eventos.insertMany([
  {
    id_evento: 1,
    nome_evento: "Semana Academica de Tecnologia",
    area: "Tecnologia",
    data_evento: ISODate("2026-06-10"),
    organizador: {
      id_organizador: 1,
      nome_organizador: "Coordenacao de Computacao",
      email: "computacao@faculdade.edu.br"
    },
    palestrantes: [
      {
        id_palestrante: 1,
        nome_palestrante: "Ana Souza",
        especialidade: "Banco de Dados"
      },
      {
        id_palestrante: 2,
        nome_palestrante: "Carlos Lima",
        especialidade: "Desenvolvimento Web"
      }
    ]
  },
  {
    id_evento: 2,
    nome_evento: "Workshop de Banco de Dados NoSQL",
    area: "Banco de Dados",
    data_evento: ISODate("2026-06-12"),
    organizador: {
      id_organizador: 1,
      nome_organizador: "Coordenacao de Computacao",
      email: "computacao@faculdade.edu.br"
    },
    palestrantes: [
      {
        id_palestrante: 1,
        nome_palestrante: "Ana Souza",
        especialidade: "Banco de Dados"
      },
      {
        id_palestrante: 3,
        nome_palestrante: "Marina Costa",
        especialidade: "Cloud Computing"
      }
    ]
  },
  {
    id_evento: 3,
    nome_evento: "Encontro de Inovacao e Empreendedorismo",
    area: "Empreendedorismo",
    data_evento: ISODate("2026-06-18"),
    organizador: {
      id_organizador: 2,
      nome_organizador: "Nucleo de Inovacao",
      email: "inovacao@faculdade.edu.br"
    },
    palestrantes: [
      {
        id_palestrante: 3,
        nome_palestrante: "Marina Costa",
        especialidade: "Cloud Computing"
      },
      {
        id_palestrante: 4,
        nome_palestrante: "Joao Pereira",
        especialidade: "Empreendedorismo"
      }
    ]
  }
])
```

### Consulta 1 - Eventos organizados pela Coordenacao de Computacao

Esta consulta envolve `EVENTO` e `ORGANIZADOR`, com restricao pelo nome do organizador.

```javascript
db.eventos.find(
  { "organizador.nome_organizador": "Coordenacao de Computacao" },
  {
    _id: 0,
    nome_evento: 1,
    area: 1,
    data_evento: 1,
    "organizador.nome_organizador": 1
  }
)
```

### Consulta 2 - Eventos com palestrantes especialistas em Banco de Dados

Esta consulta envolve `EVENTO` e `PALESTRANTE`, considerando o relacionamento `N:N`.

```javascript
db.eventos.find(
  { "palestrantes.especialidade": "Banco de Dados" },
  {
    _id: 0,
    nome_evento: 1,
    area: 1,
    palestrantes: {
      $elemMatch: {
        especialidade: "Banco de Dados"
      }
    }
  }
)
```

## Versao 2: Referenced Relationships

Na versao referenced, os dados ficam em colecoes separadas e os relacionamentos sao representados por IDs.

Neste caso, serao usadas tres colecoes:

- `organizadores`
- `eventos`
- `palestrantes`

O relacionamento `1:N` e representado em `eventos` pelo campo `id_organizador`.

O relacionamento `N:N` e representado em `eventos` pelo campo `ids_palestrantes`, que armazena uma lista com os IDs dos palestrantes participantes.

Vantagem: evita duplicacao de dados.

Desvantagem: as consultas sao mais complexas, pois precisam usar agregacao com `$lookup` para juntar informacoes de colecoes diferentes.

### Criar banco da versao referenciada

```javascript
use eventos_referenced
```

### Criar colecoes

```javascript
db.createCollection("organizadores")
db.createCollection("eventos")
db.createCollection("palestrantes")
```

### Inserir organizadores

```javascript
db.organizadores.insertMany([
  {
    id_organizador: 1,
    nome_organizador: "Coordenacao de Computacao",
    email: "computacao@faculdade.edu.br"
  },
  {
    id_organizador: 2,
    nome_organizador: "Nucleo de Inovacao",
    email: "inovacao@faculdade.edu.br"
  }
])
```

### Inserir palestrantes

```javascript
db.palestrantes.insertMany([
  {
    id_palestrante: 1,
    nome_palestrante: "Ana Souza",
    especialidade: "Banco de Dados"
  },
  {
    id_palestrante: 2,
    nome_palestrante: "Carlos Lima",
    especialidade: "Desenvolvimento Web"
  },
  {
    id_palestrante: 3,
    nome_palestrante: "Marina Costa",
    especialidade: "Cloud Computing"
  },
  {
    id_palestrante: 4,
    nome_palestrante: "Joao Pereira",
    especialidade: "Empreendedorismo"
  }
])
```

### Inserir eventos

```javascript
db.eventos.insertMany([
  {
    id_evento: 1,
    nome_evento: "Semana Academica de Tecnologia",
    area: "Tecnologia",
    data_evento: ISODate("2026-06-10"),
    id_organizador: 1,
    ids_palestrantes: [1, 2]
  },
  {
    id_evento: 2,
    nome_evento: "Workshop de Banco de Dados NoSQL",
    area: "Banco de Dados",
    data_evento: ISODate("2026-06-12"),
    id_organizador: 1,
    ids_palestrantes: [1, 3]
  },
  {
    id_evento: 3,
    nome_evento: "Encontro de Inovacao e Empreendedorismo",
    area: "Empreendedorismo",
    data_evento: ISODate("2026-06-18"),
    id_organizador: 2,
    ids_palestrantes: [3, 4]
  }
])
```

### Consulta 1 - Eventos organizados pela Coordenacao de Computacao

Esta consulta envolve `EVENTO` e `ORGANIZADOR`, com restricao pelo nome do organizador.

```javascript
db.eventos.aggregate([
  {
    $lookup: {
      from: "organizadores",
      localField: "id_organizador",
      foreignField: "id_organizador",
      as: "organizador"
    }
  },
  {
    $unwind: "$organizador"
  },
  {
    $match: {
      "organizador.nome_organizador": "Coordenacao de Computacao"
    }
  },
  {
    $project: {
      _id: 0,
      nome_evento: 1,
      area: 1,
      data_evento: 1,
      nome_organizador: "$organizador.nome_organizador",
      email_organizador: "$organizador.email"
    }
  }
])
```

### Consulta 2 - Eventos com palestrantes especialistas em Banco de Dados

Esta consulta envolve `EVENTO` e `PALESTRANTE`, considerando o relacionamento `N:N`.

```javascript
db.eventos.aggregate([
  {
    $lookup: {
      from: "palestrantes",
      localField: "ids_palestrantes",
      foreignField: "id_palestrante",
      as: "palestrantes"
    }
  },
  {
    $unwind: "$palestrantes"
  },
  {
    $match: {
      "palestrantes.especialidade": "Banco de Dados"
    }
  },
  {
    $project: {
      _id: 0,
      nome_evento: 1,
      area: 1,
      data_evento: 1,
      nome_palestrante: "$palestrantes.nome_palestrante",
      especialidade: "$palestrantes.especialidade"
    }
  }
])
```

## Explicacao para apresentacao

A modelagem possui um relacionamento `1:N` entre `ORGANIZADOR` e `EVENTO`, pois um organizador pode organizar varios eventos, mas cada evento tem apenas um organizador.

Tambem existe um relacionamento `N:N` entre `EVENTO` e `PALESTRANTE`, pois um evento pode ter varios palestrantes, e um palestrante pode participar de varios eventos.

Na versao embedded, os dados do organizador e dos palestrantes ficam dentro do documento do evento. Essa abordagem facilita consultas em que o evento e sempre buscado junto com seus dados relacionados, mas pode duplicar informacoes.

Na versao referenced, os dados ficam separados em colecoes diferentes. O evento guarda apenas o `id_organizador` e a lista `ids_palestrantes`. Essa abordagem reduz duplicacao, mas exige consultas com `$lookup` para reunir os dados relacionados.
