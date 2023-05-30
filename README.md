# salao_agenda

Guia de início rápido
Neste guia de início rápido, você aprenderá a começar a usar o Prisma do zero usando um projeto TypeScript simples e um arquivo de banco de dados SQLite local. Abrange modelagem de dados, migrações e consulta a um banco de dados.

Se você quiser usar o Prisma com seu próprio PostgreSQL, MySQL, MongoDB ou qualquer outro banco de dados suportado, vá aqui:

Comece com o Prisma do zero
Adicionar o Prisma a um projeto existente
Pré-requisitos
Você precisa do Node.js v14.17.0 ou superior para este guia (saiba mais sobre os requisitos do sistema).

1. Criar projeto TypeScript e configurar o Prisma
Como primeira etapa, crie um diretório de projeto e navegue até ele:

mkdir hello-prisma 
cd hello-prisma 
Em seguida, inicialize um projeto TypeScript usando npm:

npm init -y 
npm install typescript ts-node @types/node --save-dev 
Isso cria uma configuração inicial para seu aplicativo TypeScript.package.json

Consulte as instruções de instalação para saber como instalar o Prisma usando um gerenciador de pacotes diferente.

Agora, inicialize o TypeScript:

# npx tsc --init 
Em seguida, instale a CLI do Prisma como uma dependência de desenvolvimento no projeto:

npm install prisma --save-dev 
Por fim, configure o Prisma com o comando do Prisma CLI:init

#  npx prisma init --datasource-provider sqlite 
Isso cria um novo diretório com o arquivo de esquema Prisma e configura o SQLite como seu banco de dados. Agora você está pronto para modelar seus dados e criar seu banco de dados com algumas tabelas.prisma

2. Modele seus dados no esquema Prisma
O esquema Prisma fornece uma maneira intuitiva de modelar dados. Adicione os seguintes modelos ao arquivo:schema.prisma

prisma/schema.prisma
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}

model Post {
  id        Int     @id @default(autoincrement())
  title     String
  content   String?
  published Boolean @default(false)
  author    User    @relation(fields: [authorId], references: [id])
  authorId  Int
}
Os modelos no esquema Prisma têm duas finalidades principais:

Representar as tabelas no banco de dados subjacente
Servir como base para a API do Cliente Prisma gerada
Na próxima seção, você mapeará esses modelos para tabelas de banco de dados usando o Prisma Migrate.

3. Execute uma migração para criar suas tabelas de banco de dados com o Prisma Migrate
Neste ponto, você tem um esquema Prisma, mas ainda não tem um banco de dados. Execute o seguinte comando em seu terminal para criar o banco de dados SQLite e as tabelas e representadas por seus modelos:UserPost

# npx prisma migrate dev --name init 
Esse comando fez duas coisas:

Ele cria um novo arquivo de migração SQL para essa migração no diretório.prisma/migrations
Ele executa o arquivo de migração SQL no banco de dados.
Como o arquivo de banco de dados SQLite não existia antes, o comando também o criou dentro do diretório com o nome definido por meio da variável de ambiente no arquivo.prismadev.db.env

Parabéns, agora você já tem seu banco de dados e tabelas prontos. Vamos aprender como você pode enviar algumas consultas para ler e gravar dados!

4. Explore como enviar consultas para seu banco de dados com o Prisma Client
Para enviar consultas ao banco de dados, você precisará de um arquivo TypeScript para executar suas consultas do Prisma Client. Crie um novo arquivo chamado para esta finalidade:script.ts

touch script.ts 
Em seguida, cole o seguinte boilerplate nele:

script.ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  // ... you will write your Prisma Client queries here
}

main()
  .then(async () => {
    await prisma.$disconnect()
  })
  .catch(async (e) => {
    console.error(e)
    await prisma.$disconnect()
    process.exit(1)
  })
Esse código contém uma função que é invocada no final do script. Ele também instancia o que representa a interface de consulta para seu banco de dados.mainPrismaClient

4.1. Criar um novo registroUser
Vamos começar com uma pequena consulta para criar um novo registro no banco de dados e registrar o objeto resultante no console. Adicione o seguinte código ao arquivo:Userscript.ts

script.ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  const user = await prisma.user.create({
    data: {
      name: 'Alice',
      email: 'alice@prisma.io',
    },
  })
  console.log(user)
}

main()
  .then(async () => {
    await prisma.$disconnect()
  })
  .catch(async (e) => {
    console.error(e)
    await prisma.$disconnect()
    process.exit(1)
  })
Em vez de copiar o código, você pode digitá-lo em seu editor para experimentar o preenchimento automático fornecido pelo Prisma Client. Você também pode invocar ativamente o preenchimento automático pressionando as teclas + no teclado.CTRLSPACE

Em seguida, execute o script com o seguinte comando:

npx ts-node script.ts 
Ocultar resultados da CLI
{ id: 1, email: 'alice@prisma.io', name: 'Alice' }
Ótimo trabalho, você acabou de criar seu primeiro registro de banco de dados com o Prisma Client! 🎉

Na próxima seção, você aprenderá a ler dados do banco de dados.

4.2. Recuperar todos os registrosUser
O Prisma Client oferece diversas consultas para leitura de dados do seu banco de dados. Nesta seção, você usará a consulta que retorna todos os registros no banco de dados para um determinado modelo.findMany

Exclua a consulta anterior do Prisma Client e adicione a nova consulta:findMany

script.ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  const users = await prisma.user.findMany()
  console.log(users)
}

main()
  .then(async () => {
    await prisma.$disconnect()
  })
  .catch(async (e) => {
    console.error(e)
    await prisma.$disconnect()
    process.exit(1)
  })
Execute o script novamente:

npx ts-node script.ts 
Ocultar resultados da CLI
[{ id: 1, email: 'alice@prisma.io', name: 'Alice' }]
Observe como o único objeto agora está entre colchetes no console. Isso porque o retornou uma matriz com um único objeto dentro.UserfindMany

4.3. Explore consultas de relação com o Prisma
Uma das principais características do Prisma Client é a facilidade de trabalhar com relacionamentos. Nesta seção, você aprenderá a criar um e um registro em uma consulta de gravação aninhada. Depois, você verá como recuperar a relação do banco de dados usando a opção.UserPostinclude

Primeiro, ajuste o script para incluir a consulta aninhada:

script.ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  const user = await prisma.user.create({
    data: {
      name: 'Bob',
      email: 'bob@prisma.io',
      posts: {
        create: {
          title: 'Hello World',
        },
      },
    },
  })
  console.log(user)
}

main()
  .then(async () => {
    await prisma.$disconnect()
  })
  .catch(async (e) => {
    console.error(e)
    await prisma.$disconnect()
    process.exit(1)
  })
Execute a consulta executando o script novamente:

npx ts-node script.ts 
Ocultar resultados da CLI
{ id: 2, email: 'bob@prisma.io', name: 'Bob' }
Por padrão, o Prisma retorna apenas campos escalares nos objetos de resultado de uma consulta. É por isso que, embora você também tenha criado um novo registro para o novo registro, o console imprimiu apenas um objeto com três campos escalares: , e .PostUseridemailname

Para também recuperar os registros que pertencem a um , você pode usar a opção através do campo de relação:PostUserincludeposts

script.ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  const usersWithPosts = await prisma.user.findMany({
    include: {
      posts: true,
    },
  })
  console.dir(usersWithPosts, { depth: null })
}

main()
  .then(async () => {
    await prisma.$disconnect()
  })
  .catch(async (e) => {
    console.error(e)
    await prisma.$disconnect()
    process.exit(1)
  })
Execute o script novamente para ver os resultados da consulta de leitura aninhada:

npx ts-node script.ts 
Ocultar resultados da CLI
[
  { id: 1, email: 'alice@prisma.io', name: 'Alice', posts: [] },
  {
    id: 2,
    email: 'bob@prisma.io',
    name: 'Bob',
    posts: [
      {
        id: 1,
        title: 'Hello World',
        content: null,
        published: false,
        authorId: 2
      }
    ]
  }
]
Desta vez, você está vendo dois objetos sendo impressos. Ambos têm um campo (que está vazio para e preenchido com um único objeto para ) que representa os registros associados a eles.Userposts"Alice"Post"Bob"Post

Observe que os objetos na matriz também são totalmente digitados. Isso significa que você obterá o preenchimento automático e o compilador TypeScript impedirá que você os digite acidentalmente.usersWithPosts

5. Próximos passos
Neste guia de início rápido, você aprendeu como começar a usar o Prisma em um projeto TypeScript simples. Sinta-se à vontade para explorar a API do Prisma Client um pouco mais por conta própria, por exemplo, incluindo opções de filtragem, classificação e paginação na consulta ou explorando mais operações como e consultas.findManyupdatedelete

Explore os dados no Prisma Studio
O Prisma vem com uma GUI integrada para visualizar e editar os dados em seu banco de dados. Você pode abri-lo usando o seguinte comando:

npx prisma studio 
Configurar o Prisma com seu próprio banco de dados
Se você quiser avançar com o Prisma usando seu próprio PostgreSQL, MySQL, MongoDB ou qualquer outro banco de dados suportado, siga os guias Configurar o Prisma:

Comece com o Prisma do zero
Adicionar o Prisma a um projeto existente
Explore exemplos de Prisma prontos para serem executados
Confira o repositório prisma-examples no GitHub para ver como o Prisma pode ser usado com sua biblioteca favorita. O repositório contém exemplos com Express, NestJS, GraphQL, bem como exemplos fullstack com Next.js e Vue.js, e muito mais.