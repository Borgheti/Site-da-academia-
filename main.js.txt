// Verifica e exibe a tela de login ao iniciar
window.onload = () => {
  mostrarTela("login");
  atualizarListaAlunos();
};

// Função para mostrar uma tela e esconder as outras
function mostrarTela(tela) {
  const telas = ["login", "painelProfessor", "painelAluno"];
  telas.forEach(t => {
    document.getElementById(t).style.display = (t === tela) ? "block" : "none";
  });
  // Esconde também a seção de redefinir senha ao trocar de tela
  document.getElementById("redefinirSenha").style.display = "none";
}

// Faz o login do usuário
function logar() {
  const nome = document.getElementById("loginNome").value.trim();
  const senha = document.getElementById("loginSenha").value.trim();

  if (!nome || !senha) {
    alert("Preencha nome e senha.");
    return;
  }

  const alunos = JSON.parse(localStorage.getItem("alunos")) || [];

  // Verifica se o login é professor (exemplo: professor com nome "professor" e senha "123")
  if (nome.toLowerCase() === "professor" && senha === "123") {
    mostrarTela("painelProfessor");
    atualizarListaAlunos();
    carregarFeedbacks();
    return;
  }

  // Busca aluno
  const aluno = alunos.find(a => a.nome.toLowerCase() === nome.toLowerCase() && a.senha === senha);
  if (aluno) {
    mostrarTela("painelAluno");
    mostrarFichaAluno(aluno);
    alunoLogado = aluno;
  } else {
    alert("Usuário ou senha inválidos.");
  }
}

let alunoLogado = null;

// Logout - volta para login
function logout() {
  alunoLogado = null;
  mostrarTela("login");
  limparCamposLogin();
}

function limparCamposLogin() {
  document.getElementById("loginNome").value = "";
  document.getElementById("loginSenha").value = "";
  document.getElementById("nomeRedefinir").value = "";
  document.getElementById("novaSenha").value = "";
  document.getElementById("redefinirSenha").style.display = "none";
}

// Adicionar ou atualizar ficha do aluno
function adicionarAluno() {
  const nome = document.getElementById("nomeAluno").value.trim();
  const senha = document.getElementById("senhaAluno").value.trim();
  const treino = document.getElementById("treinoAluno").value.trim();

  if (!nome || !senha || !treino) {
    alert("Preencha todos os campos.");
    return;
  }

  let alunos = JSON.parse(localStorage.getItem("alunos")) || [];
  const index = alunos.findIndex(a => a.nome.toLowerCase() === nome.toLowerCase());

  if (index >= 0) {
    // Atualiza aluno existente
    alunos[index].senha = senha;
    alunos[index].treino = treino;
  } else {
    // Adiciona novo aluno
    alunos.push({ nome, senha, treino, feedbacks: [] });
  }

  localStorage.setItem("alunos", JSON.stringify(alunos));
  alert("Ficha salva com sucesso!");
  limparCamposProfessor();
  atualizarListaAlunos();
}

function limparCamposProfessor() {
  document.getElementById("nomeAluno").value = "";
  document.getElementById("senhaAluno").value = "";
  document.getElementById("treinoAluno").value = "";
}

// Atualiza a lista de alunos para o professor
function atualizarListaAlunos() {
  const lista = document.getElementById("listaAlunos");
  lista.innerHTML = "";
  const alunos = JSON.parse(localStorage.getItem("alunos")) || [];

  if (alunos.length === 0) {
    lista.innerHTML = "<p>Nenhum aluno cadastrado.</p>";
    return;
  }

  alunos.forEach(aluno => {
    const div = document.createElement("div");
    div.classList.add("aluno-card");
    div.innerHTML = `<strong>${aluno.nome}</strong><div class="treino">${aluno.treino}</div>`;
    lista.appendChild(div);
  });
}

// Filtrar alunos na busca
function filtrarAlunos() {
  const busca = document.getElementById("buscaAluno").value.trim().toLowerCase();
  const resultado = document.getElementById("resultadoBusca");
  resultado.innerHTML = "";

  const alunos = JSON.parse(localStorage.getItem("alunos")) || [];
  const encontrados = alunos.filter(a => a.nome.toLowerCase().includes(busca));

  if (encontrados.length === 0) {
    resultado.innerHTML = "<p>Nenhum aluno encontrado.</p>";
    return;
  }

  encontrados.forEach(aluno => {
    const div = document.createElement("div");
    div.classList.add("aluno-card");
    div.innerHTML = `
      <strong>${aluno.nome}</strong>
      <textarea class="treino" oninput="atualizarTreino('${aluno.nome}', this.value)">${aluno.treino}</textarea>
    `;
    resultado.appendChild(div);
  });
}

// Atualiza o treino do aluno no localStorage ao editar
function atualizarTreino(nomeAluno, novoTreino) {
  let alunos = JSON.parse(localStorage.getItem("alunos")) || [];
  const index = alunos.findIndex(a => a.nome === nomeAluno);
  if (index >= 0) {
    alunos[index].treino = novoTreino;
    localStorage.setItem("alunos", JSON.stringify(alunos));
    atualizarListaAlunos();
  }
}

// Mostrar ficha do aluno logado
function mostrarFichaAluno(aluno) {
  const div = document.getElementById("fichaAluno");
  div.innerHTML = `<h3>${aluno.nome}</h3><pre class="treino">${aluno.treino}</pre>`;
}

// ** NOVO ** - Mostrar campo para redefinir senha
function mostrarRedefinirSenha() {
  document.getElementById("redefinirSenha").style.display = "block";
}

// ** NOVO ** - Redefinir senha do aluno
function redefinirSenha() {
  const nome = document.getElementById("nomeRedefinir").value.trim();
  const novaSenha = document.getElementById("novaSenha").value.trim();

  if (!nome || !novaSenha) {
    alert("Preencha o nome e a nova senha.");
    return;
  }

  let alunos = JSON.parse(localStorage.getItem("alunos")) || [];
  const aluno = alunos.find(a => a.nome.toLowerCase() === nome.toLowerCase());

  if (!aluno) {
    alert("Aluno não encontrado.");
    return;
  }

  aluno.senha = novaSenha;
  localStorage.setItem("alunos", JSON.stringify(alunos));
  alert("Senha redefinida com sucesso!");
  document.getElementById("redefinirSenha").style.display = "none";
  limparCamposLogin();
}

// Funções de feedback (se quiser que eu implemente, me avise)
function enviarFeedback() {
  if (!alunoLogado) {
    alert("Você precisa estar logado para enviar feedback.");
    return;
  }

  const nota = parseInt(document.getElementById("notaFeedback").value);
  const comentario = document.getElementById("comentarioFeedback").value.trim();

  if (!nota || nota < 1 || nota > 5) {
    alert("Digite uma nota entre 1 e 5.");
    return;
  }

  let alunos = JSON.parse(localStorage.getItem("alunos")) || [];
  const index = alunos.findIndex(a => a.nome === alunoLogado.nome);
  if (index >= 0) {
    if (!alunos[index].feedbacks) alunos[index].feedbacks = [];
    alunos[index].feedbacks.push({ nota, comentario, data: new Date().toISOString() });
    localStorage.setItem("alunos", JSON.stringify(alunos));
    alert("Feedback enviado, obrigado!");
    document.getElementById("notaFeedback").value = "";
    document.getElementById("comentarioFeedback").value = "";
  }
}

function carregarFeedbacks() {
  const feedbacksDiv = document.getElementById("feedbacksRecebidos");
  feedbacksDiv.innerHTML = "";
  const alunos = JSON.parse(localStorage.getItem("alunos")) || [];

  alunos.forEach(aluno => {
    if (aluno.feedbacks && aluno.feedbacks.length > 0) {
      const divAluno = document.createElement("div");
      divAluno.innerHTML = `<h3>Feedbacks de ${aluno.nome}</h3>`;
      aluno.feedbacks.forEach(fb => {
        const divFb = document.createElement("div");
        divFb.innerHTML = `<strong>Nota:</strong> ${fb.nota} <br> <strong>Comentário:</strong> ${fb.comentario || "(Sem comentário)"} <br><small>${new Date(fb.data).toLocaleString()}</small><hr>`;
        divAluno.appendChild(divFb);
      });
      feedbacksDiv.appendChild(divAluno);
    }
  });
}