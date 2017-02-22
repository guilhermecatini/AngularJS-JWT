# Acessando uma rota segura utilizando JSON Web Token e AngularJS

---

O JSON Web Token ( JWT ) é um padrão aberto baseado em JSON para criar tokens de acesso que afirmam um certo número de reivindicações. Por exemplo, um servidor pode gerar um token que tenha a reivindicação "logado como administrador" e fornecer isso a um cliente. O cliente poderia usar esse token para provar que ele está conectado como administrador. Os tokens são assinados pela chave do servidor, para que o cliente possa verificar se o token é legítimo. Os tokens são projetados para serem compactos, seguros de URL e utilizáveis especialmente no contexto de logon único de navegador. As reivindicações JWT podem ser tipicamente usadas para passar a identidade de usuários autenticados entre um provedor de identidade e um provedor de serviços , ou qualquer outro tipo de reivindicações conforme exigido por processos empresariais. Os tokens também podem ser autenticados e criptografados.

Hoje vamos utilizar o AngularJS para fazer uma requisição em nosso servidor, solicitar um token válido, e com esse token vamos fazer um cadastro (Create) e vamos também listar os cadastros (Retrieve);

## Requisitos

> - Iniciar um projeto utilizando o Express-Generator
> - Instalar o pacote AngularJS ( `bower install angular --save` )
> - Instalar o pacote Bootstrap( `bower install bootstrap --save` )

	O servidor que vamos usar para fazer os testes é o http://catini.org:3001

Bom, mãos na massa!

Vamos criar o arquivo `public/index.html` e deixar preparado para começar a fazer as requisições.

```html
<!DOCTYPE html>
<html data-ng-app="appJWT">

<head>
  <title>AngularJS - JWT</title>
  <link rel="stylesheet" href="./vendor/bootstrap/dist/css/bootstrap.min.css">
</head>

<body data-ng-controller="ctlJWT as vm">

  <script src="./vendor/jquery/dist/jquery.min.js"></script>
  <script src="./vendor/bootstrap/dist/js/bootstrap.min.js"></script>
  <script src="./vendor/angular/angular.min.js"></script>
  
  <script>
    angular.module('appJWT', [])
    .controller('ctlJWT', ctlJWT)

    function ctlJWT($http) {
      var vm = this

    }
  </script>
</body>

</html>
```

Primeiro vamos criar o campo de tela onde vamos receber o nosso Token do nosso servidor `http://catini.org:3001/api/users/getToken`

No html vamos fazer desse modo:
```html
  <div class="container">
    <div class="row">

      <div class="text-center">
        <h1>JSON Web Token</h1>
      </div>

      <hr>

      <div class="text-center">
        <button class="btn btn-primary" data-ng-click="vm.getToken()">Get Token</button>
      </div>

      <hr>

      <div class="jumbotron">
        <label>Token</label>
        <textarea data-ng-model="vm.myToken" rows="5" class="form-control"></textarea>
      </div>

    </div>
  </div>
```
e no script:
```javascript
vm.myToken = ''

vm.getToken = getToken
function getToken() {
  $http({
    method: 'GET',
    url: 'http://catini.org:3001/api/users/getToken'
  })
    .then(function (ret) {
      if (ret.data.error === false) {
        vm.myToken = ret.data.token
      }
    })
}
```
Pronto, agora nosso programa está conectando no servidor e recebendo um token válido para poder fazer um cadastro de usuário ou listar todos os usuários cadastrados.

### Cadastrando um Usuário
Vou criar um formulário simples onde vão ter três campos: **name**, **user** e **password**. Esse formulário vai conectar no link `http://catini.org:3001/api/users/create` e vai gravar o cadastro no banco de dados desde que o token seja válido no campo que criamos acima.

**HTML**
```html
<hr>
<h3>User Form ( Create )</h3>
<div class="jumbotron">
  <div class="container">
    <form name="myForm">

      <div class="col-xs-12 col-sm-12 col-md-12 col-lg-12">
        <label for="user">Name</label>
        <input type="text" class="form-control" name="name" data-ng-model="vm.user.name" required="">
      </div>

      <div class="col-xs-12 col-sm-12 col-md-12 col-lg-12">
        <label for="user">User</label>
        <input type="text" class="form-control" name="user" data-ng-model="vm.user.user" required="">
      </div>

      <div class="col-xs-12 col-sm-12 col-md-12 col-lg-12">
        <label for="user">Password</label>
        <input type="password" class="form-control" name="password" data-ng-model="vm.user.password" required="">
      </div>


      <div class="col-xs-12 col-sm-12 col-md-12 col-lg-12">
        <hr>
        <button data-ng-disabled="myForm.$invalid" class="btn btn-primary" data-ng-click="vm.Create()">Create User</button>
      </div>

      <div class="col-xs-12 col-sm-12 col-md-12 col-lg-12" data-ng-show="vm.messageCreate">
        <hr>
        <div class="alert alert-info">
          <button type="button" class="close" data-dismiss="alert" aria-hidden="true">&times;</button>
          <strong>Message:</strong> {{vm.messageCreate}}
        </div>
      </div>
    </form>
  </div>
</div>
```

Agora o Javascript

```javascript
vm.Create = Create
function Create() {
  $http({
    method: 'POST',
    url: '/api/users/create',
    data: vm.user,
    headers: {
      'Authorization': vm.myToken
    }
  })
    .then(function (ret) {
      vm.messageCreate = 'User created! ' + JSON.stringify(ret.data)
    }, function (err) {
      vm.messageCreate = err.data
    })
}
```
Observe que estamos enviando no header o **Authorization** com Token válido para autorizar o cadastro. Caso você tentar modificar o token ou não inserir um, vai exibir uma mensagem de erro na tela.

### Listando os Usuários Cadastrados

Agora vou Listar os usuários já cadastrados no banco usando o seguinte link da API: `http://catini.org:3001/api/users/retrieve`

**HTML**
```html
<hr>
<h3>List Users (Retrieve) </h3>
<div class="jumbotron">
  <div class="container">

    <div class="col-xs-12 col-sm-12 col-md-12 col-lg-12">
      <button class="btn btn-primary" data-ng-click="vm.Retrieve()">Find Users</button>
      <hr>
    </div>

    <div class="col-xs-12 col-sm-12 col-md-12 col-lg-12" data-ng-show="vm.messageRetrieve">
      <div class="alert alert-info">
        <button type="button" class="close" data-dismiss="alert" aria-hidden="true">&times;</button>
        <strong>Message:</strong> {{vm.messageRetrieve}}
      </div>
      <hr>
    </div>

    <div class="col-xs-12 col-sm-12 col-md-12 col-lg-12">

      <table class="table table-hover">
        <thead>
          <tr>
            <th>Name</th>
            <th>User</th>
            <th>Password</th>
          </tr>
        </thead>
        <tbody>
          <tr data-ng-repeat="user in vm.users">
            <td>{{user.name}}</td>
            <td>{{user.user}}</td>
            <td>{{user.password}}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</div>
```

**JavaScript**

```javascript
vm.Retrieve = Retrieve
function Retrieve() {
  $http({
    method: 'GET',
    url: '/api/users/retrieve',
    headers: {
      'Authorization': vm.myToken
    }
  })
    .then(function (ret) {
      vm.users = ret.data
      vm.messageRetrieve = 'Users have been listed!'
    }, function (err) {
      vm.users = {}
      vm.messageRetrieve = err.data
    })
}
```

Pronto, agora você consegue também listar os usuários.