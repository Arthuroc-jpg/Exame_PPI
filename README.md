import express from "express";
import cookieParser from "cookie-parser";
import session from "express-session";

const host = "0.0.0.0";
const porta = 3000;

var vet_interessados = [];
var vet_pets = [];
var usuario = false;
const server = express();

var usuario_logado = false;

server.use(session({
  secret: "S3cr3tK3y",
  resave: true,
  saveUninitialized: true,
  cookie: { maxAge: 1000*60*15}
}));

server.use(express.urlencoded({extended: true}));
server.use(cookieParser());

server.get("/",verLogin,(req,res) => {

    let ultimoAcesso = req.cookies?.ultimoAcesso;

    if(usuario_logado){
    res.redirect("/login");
  }

const data = new Date();
res.cookie("ultimoAcesso",data.toLocaleString());

res.setHeader("Content-Type","text/html");
res.write(`
    <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Menu</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <nav class="navbar navbar-expand-lg bg-body-tertiary">
  <div class="container-fluid">
    <a class="navbar-brand" href="#">Menu</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navbarNav">
      <ul class="navbar-nav">
        <li class="nav-item">
          <a class="nav-link active" aria-current="page" href="/cadastroInteressados">Cadastro de Interessados</a>
        </li>
        <li class="nav-item">
          <a class="nav-link active" href="/cadastroPets">Cadastro de Pets</a>
        </li>
        <li class="nav-item">
          <a class="nav-link active" href="/adotarPet">Adotar um Pet</a>
        </li>
         <li class="nav-item">
          <a class="nav-link active" href="/login">Login</a>
        </li>
         <li class="nav-item">
          <p  style="font-size:15px">Útlimo acesso: ${ultimoAcesso || "Primeiro Acesso"}</p>
        </li>
      </ul>
    </div>
  </div>
  <hr/>
</nav>
</body>
</html>
    `);
    res.end();
});

//CADASTRO DE INTERESSADOS
server.get("/cadastroInteressados",verLogin,(req,res) => {
    res.send(
        `
        <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cadastro Interessados</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div style="margin-top: 15%;">
        <h2 style="text-align: center; font-family: cursive;">Cadastro de Interessados</h2>
        <br/>
        <form class="row g-3 needs-validation" style="margin-left: 2%;">
  <div class="col-md-4">
    <label for="validationCustom01" class="form-label">Nome</label>
    <input type="text" class="form-control" id="validationCustom01">
  </div>
  <div class="col-md-4">
    <label for="validationCustom02" class="form-label">E-mail</label>
    <input type="text" class="form-control" id="validationCustom02">
  </div>
    <div class="col-md-3">
    <label for="validationCustom05" class="form-label">Telefone</label>
    <input type="text" class="form-control" id="validationCustom03">
    </div>
    <br/>
     <div class="col-12">
    <button class="btn btn-primary" type="submit" style="margin-left: 45%;">Enviar</button>
  </div>
  </div>  
  </form>
</div>
</body>
</html>
        `
    )
});

server.post("/adicionarInteressado",verLogin,(req,res)=> {
    const nome = req.body.validationCustom1;
    const email = req.body.validationCustom2;
    const fone = req.body.validationCustom3;

    if(nome && email && fone) {
        vet_interessados.push({nome,email,fone});
        res.redirect("/listarInteressados");
    }

    else {
        let conteudo =
        `
                <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cadastro Interessados</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div style="margin-top: 15%;">
        <h2 style="text-align: center; font-family: cursive;">Cadastro de Interessados</h2>
        <br/>
        <form class="row g-3 needs-validation" style="margin-left: 2%;">
  <div class="col-md-4">
    <label for="validationCustom01" class="form-label">Nome</label>
    <input type="text" class="form-control" id="validationCustom01">
        `;

           if(!nome) {
        conteudo+=
    ` <div>
  <p class="text-danger">Informe o nome do interessado</p>
  </div>`
    }

    conteudo+=
    `
    </div>
  <div class="col-md-4">
    <label for="validationCustom02" class="form-label">E-mail</label>
    <input type="text" class="form-control" id="validationCustom02">
    `;

    if(!email) {
        conteudo+=
    ` <div>
  <p class="text-danger">Informe um email</p>
  </div>`
    }

    conteudo +=
    `
     </div>
    <div class="col-md-3">
    <label for="validationCustom05" class="form-label">Telefone</label>
    <input type="text" class="form-control" id="validationCustom03">
    `;

    
    if(!fone) {
        conteudo+=
    ` <div>
  <p class="text-danger">Informe um telefone</p>
  </div>`
    }

    conteudo+=
    `
    </div>
    <br/>
     <div class="col-12">
    <button class="btn btn-primary" type="submit" style="margin-left: 45%;">Enviar</button> (VERIFICAR)
  </div>
  </div>  
  </form>
</div>
</body>
</html>
    `;

    res.send(conteudo);
    }
});

server.get("/listarInteressados",verLogin,(req,res)=>{
    let conteudo =
    `
    <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cadastro Interessados</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div style="margin-top: 5%;">
        <h2 style="text-align: center; font-family: cursive;">Lista de Interessados</h2>
        <br/>
        <table class="table">
            <thead>
            <tr>
    `;

    for(let i=0;i<vet_interessados.length;i++) {
        conteudo+=
        `
        <th scope="col">1. ${vet_interessados[i].nome}</th>
        `;
    }

    conteudo+=
    `
    </tr>
         </thead>
        </table>
          <br/>
  <div style="text-align: center;">
  <a href="/" class="nav-link active">Voltar ao menu</a>
  <a href="/cadastroInteressados" class="nav-link active">Continuar Cadastrando</a>
  </div>
</div>
</body>
</html>
    `;

    res.send(conteudo);
});

server.get("/cadastroPets",verLogin,(req,res)=>{
    `
            <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cadastro Pets</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div style="margin-top: 15%;">
        <h2 style="text-align: center; font-family: cursive;">Cadastro de Pets</h2>
        <br/>
        <form class="row g-3 needs-validation" style="margin-left: 2%;">
  <div class="col-md-4">
    <label for="validationCustom04" class="form-label">Nome</label>
    <input type="text" class="form-control" id="validationCustom04">
  </div>
  <div class="col-md-4">
    <label for="validationCustom05" class="form-label">Raça</label>
    <input type="text" class="form-control" id="validationCustom05">
  </div>
    <div class="col-md-3">
    <label for="validationCustom06" class="form-label">Idade</label>
    <input type="number" min="0" max="30" class="form-control" id="validationCustom06">
    </div>
    <br/>
     <div class="col-12">
    <button class="btn btn-primary" type="submit" style="margin-left: 45%;">Enviar</button>
  </div>
  </div>  
  </form>
</div>
</body>
</html>
    `
});

server.post("/adicionarPets",(req,res)=>{
    const nome = req.body.validationCustom4;
    const raca = req.body.validationCustom5;
    const age = req.body.validationCustom6;

    if(nome && raca && age) {
        vet_pets.push({nome,raca,age});
        res.redirect("/listarPets");
    }

    else {
        let conteudo =
        `
                    <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cadastro Pets</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div style="margin-top: 15%;">
        <h2 style="text-align: center; font-family: cursive;">Cadastro de Pets</h2>
        <br/>
        <form class="row g-3 needs-validation" style="margin-left: 2%;">
  <div class="col-md-4">
    <label for="validationCustom04" class="form-label">Nome</label>
    <input type="text" class="form-control" id="validationCustom04">
        `;

           if(!nome) {
        conteudo+=
    ` <div>
  <p class="text-danger">Informe o nome do pet</p>
  </div>`
    }

    conteudo+=
    `
     </div>
  <div class="col-md-4">
    <label for="validationCustom05" class="form-label">Raça</label>
    <input type="text" class="form-control" id="validationCustom05">
    `;

    if(!raca) {
        conteudo+=
    ` <div>
  <p class="text-danger">Informe a raça</p>
  </div>`
    }

    conteudo +=
    `
    </div>
    <div class="col-md-3">
    <label for="validationCustom06" class="form-label">Idade</label>
    <input type="number" min="0" max="30" class="form-control" id="validationCustom06">
    `;

    
    if(!age) {
        conteudo+=
    ` <div>
  <p class="text-danger">Informe a idade</p>
  </div>`
    }

    conteudo+=
    `
    </div>
    <br/>
     <div class="col-12">
    <button class="btn btn-primary" type="submit" style="margin-left: 45%;">Enviar</button> (VERIFICAR)
  </div>
  </div>  
  </form>
</div>
</body>
</html>
    `;

    res.send(conteudo);
    }

});

server.get("/listarPets",verLogin,(req,res)=>{
    let conteudo =
    `
    <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cadastro Interessados</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div style="margin-top: 5%;">
        <h2 style="text-align: center; font-family: cursive;">Lista de Pets</h2>
        <br/>
        <table class="table">
            <thead>
            <tr>
    `;

    for(let i=0;i<vet_pets.length;i++) {
        conteudo+=
        `
        <th scope="col">1. ${vet_pets[i].nome}</th>
        `;
    }

    conteudo+=
    `
    </tr>
         </thead>
        </table>
          <br/>
  <div style="text-align: center;">
  <a href="/" class="nav-link active">Voltar ao menu</a>
  <a href="/cadastroPets" class="nav-link active">Continuar Cadastrando</a>
  </div>
</div>
</body>
</html>
    `;

    res.send(conteudo);
});

server.get("/adotarPet",(req,res)=> {
    const nome_interessado = req.body.validationCustom1;
    const nome_pet = req.body.validationCustom4;

    let conteudo =
    `
            <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cadastro Interessados</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div style="margin-top: 15%;">
        <h2 style="text-align: center; font-family: cursive;">Adotar um Pet</h2>
        <br/>
        <form class="row g-3 needs-validation" style="margin-left: 2%;">
  <div class="col-md-4">
    <label for="validationCustom01" class="form-label">Interessados</label>
    <select class="form-select" id="validationCustom07">
    `
      for(let i=0;i<vet_interessados.length;i++) {
        conteudo+=
        `
        <option>${vet_interessados[i].nome_interessado}</option>
        `;
    }

    conteudo+= `
    </select>
    </div>
    div class="col-md-4">
    <label for="validationCustom01" class="form-label">Pets</label>
    <select class="form-select" id="validationCustom08">
    `;

         for(let i=0;i<vet_pets.length;i++) {
        conteudo+=
        `
        <option>${vet_pets[i].nome_pet}</option>
        `;
    }

    conteudo+=
    `
    </select>
    </div>
    <div class="col-md-4">
    <label for="validationCustom01" class="form-label">Interessados</label>
    <input type="date" class="form-select" id="validationCustom09">
  </div>
    <br/>
     <div class="col-12">
    <button class="btn btn-primary" type="submit" style="margin-left: 45%;">Enviar</button>
  </div>
  </div>  
  </form>
</div>
</body>
</html>
    `;

    res.send(conteudo);
});

server.get("/login",(req,res)=>{
res.send(`
          <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div style="margin-top: 15%;">
        <h2 style="text-align: center; font-family: cursive;">Login</h2>
        <hr/>
        <br/>
        <form class="row g-3 needs-validation" style="margin-left: 2%;">
  <div class="col-md-4">
    <label for="user" class="form-label">Usuário</label>
    <input type="text" class="form-control" id="user">
  </div>
  <div class="col-md-4">
    <label for="senha" class="form-label">Senha</label>
    <input type="password" class="form-control" id="senha">
  </div>
    <br/>
     <div class="col-12">
    <button class="btn btn-primary" type="submit" style="margin-left: 45%;">Enviar</button>
  </div>
  </div>  
  </form>
</div>
</body>
</html>
  `)
});

server.post("/login",(req,res)=>{
  const {user,senha} = req.body;

  if(user=="inter" && senha=="inter") {
    req.session.dadosLogin = true
  req.session.dadosLogin = {nome:"Inter",logado:true};
  res.redirect("/");
  }

  else {
    res.send(
      `
          <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div style="margin-top: 15%;">
        <h2 style="text-align: center; font-family: cursive;">Login</h2>
        <hr/>
        <br/>
        <form class="row g-3 needs-validation" style="margin-left: 2%;">
  <div class="col-md-4">
    <label for="user" class="form-label">Usuário</label>
    <input type="text" class="form-control" id="user">
  </div>
  <div class="col-md-4">
    <label for="senha" class="form-label">Senha</label>
    <input type="password" class="form-control" id="senha">
  </div>
    <br/>
     <div class="col-12">
    <button class="btn btn-primary" type="submit" style="margin-left: 45%;">Enviar</button>
  </div>
  </div>  
  </form>
   <div>
    <p class="text-danger">Usuário ou senha inválidos</p>
    </div>
</div>
</body>
</html>
  `
    )
  }

});

//FINAL
  server.get("/logout",(req,res)=>{
    req.session.destroy();
    res.redirect("/login");
  })

  function verLogin(req,res,next) {
    if(req.session.dadosLogin?.logado)
      next();

    else 
      res.redirect("/login");
  }

  server.listen(porta, host, () => {
console.log(`Servidor rodando em http://${host}:${porta}`) 
});
