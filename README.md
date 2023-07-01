
# Hosting Nuxt2 App on VPS

Hospedar aplicação Nuxt.js em VPS ou hospedagem compartilhada.

## Pacotes utilizados

- NodeJS - v.*

- NPM - v.*

- PM2 - v.* 


## Configurando a aplicação dentro da VPS

Precisamos de uma .env para armazenar em qual porta nossa aplicação vai rodar. Por padrão ela roda em **3000**, mas você pode personalizar isso.

Vamos criar uma .env na raiz do projeto Nuxt e inserir a porta desejada, nesse caso, vou usar a porta **4000**.

#### .env
```bash
  PORT = 4000
```

Agora, vamos registrar a variável porta (PORT) que criamos na **.env** acima, dentro do arquivo **nuxt.config.js**, para que a aplicação reconheça essa variável. Basta adicionar a seguinte linha no topo do bloco **export default {}**:

#### variável port:
```bash
  server: {
    port: process.env.PORT
  },
```

#### inserindo ela no arquivo nuxt.config.js:
```bash
  export default {

    server: {
      port: process.env.PORT
    },

    // restante do código.
  }
```

Não podemos esquecer de remover, caso exista, esse trecho de código aqui:

```bash
  target: 'static' // default is 'server'
```

As aplicações em nuxt2 por padrão vem com o deploy estático, e nós não vamos utilizar ele, e sim hospedar nossa aplicação online.

Após isso, precisamos de um outro arquivo na raiz do projeto, esse arquivo vai conter informações para nossa aplicação não fechar quando o terminal fechar, usando o PM2.

#### ecosystem.config.js
```bash
  module.exports = {
    apps: [
        {
          name: 'NomeDoProjeto',
          exec_mode: 'cluster',
          instances: 1,
          script: './node_modules/nuxt/bin/nuxt.js',
          args: 'start',
          env: {
              PORT: process.env.PORT
          }
        }
    ]
  }
```

No parâmetro **name**, vamos inserir o nome que a aplicação vai ter ao rodar dentro da sua VPS.
Com esse nome em mãos, conseguimos iniciar, pausar, deletar e parar a nossa aplicação a qualquer momento.

Já no parâmetro **PORT**, vamos usar a nossa variável de ambeinte que criamos no primeiro passo, a **.env** que contém a porta na qual vamos hospedar nossa aplicação.

Por padrão, nossa aplicação vai ser hospedada em localhost:[porta].

Feito isso, já podemos rodar nossa aplicação!!

## Rodando a aplicação

Para rodar nossa aplicação vamos utilizar o **PM2**, que é responsável por deixar nossa aplicação 'online' mesmo fechando o terminal.

Para isso, vamos ter de ir até a pasta do projeto em que nossa aplicação se encontra, e rodar o seguinte comando:

```bash
  pm2 start NomeDoProjeto
```

Pronto! nossa aplicação está online dentro da nossa VPS!

Lembrando que, o **NomeDoProjeto** é o nome que usamos na configuração do arquivo **ecosystem.config.js**.

Se você quiser saber se sua aplicação está online, ou foi inciada, pode usar o comando

```bash
  pm2 ls
```

Esse comando vai listar as aplicações rodando dentro do PM2 e o status de cada uma.

Para mais comandos de como pausar, parar e deletar a aplicação, você pode conferir o link da documentação do PM2:

https://pm2.keymetrics.io/docs/usage/quick-start/

**OBS**: Tome cuidado com as portas, se você pretende rodar várias aplicações em uma VPS apenas, será necessário trocar as portas para cada aplicação, para evitar de todas elas usarem a mesma porta e o redirecionamento não funcionar!!

## Redirecionando nossa Aplicação

Por último, vamos redirecionar o domínio/subdomínio para a nossa aplicação de fato.

Vamos utilizar o apache para fazer isso, com um arquivo **.htaccess**.

Este arquivo deve ser criado na public_html do seu domínio ou dentro da public_html/pasta do seu subdomínio.

#### .htaccess

```bash
  RewriteEngine On

  RewriteCond %{SERVER_PORT} 443
  RewriteRule ^index.php(.*) http://localhost:{PORT}/$1 [P,L]
  RewriteRule (.*) http://localhost:{PORT}/$1 [P,L]
```

Neste arquivo, redirecionamos todas as solicitações para dentro de localhost:{PORT}(essa porta tem de ser colocada manualmente neste arquivo, pois, não conseguimos acessar uma variável aqui dentro).

Como utilizei a porta 4000 na .env que criamos acima, o .htaccess ficaria assim:

#### .htaccess

```bash
  RewriteEngine On

  RewriteCond %{SERVER_PORT} 443
  RewriteRule ^index.php(.*) http://localhost:4000/$1 [P,L]
  RewriteRule (.*) http://localhost:4000/$1 [P,L]
```

Com esse arquivo criado dentro da nossa public_html, podemos reiniciar o serviço do apache e nossa aplicação estará online e pronta para uso!!
