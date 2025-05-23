<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8" />
  <title>Vários Contadores Independentes</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 30px;
      background-color: #f5f5f5;
    }
    h1 {
      text-align: center;
      margin-bottom: 40px;
    }
    .contador-container {
      background: white;
      border-radius: 8px;
      padding: 20px;
      margin-bottom: 30px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
      max-width: 450px;
    }
    .contador-header {
      font-size: 22px;
      font-weight: bold;
      margin-bottom: 15px;
      color: #2c3e50;
      text-align: center;
    }
    .contador-tempo {
      font-size: 40px;
      font-weight: bold;
      color: #2c3e50;
      text-align: center;
      margin-bottom: 20px;
      font-variant-numeric: tabular-nums;
    }
    label {
      display: block;
      margin-top: 12px;
      font-weight: bold;
    }
    input[type="text"] {
      width: 100%;
      padding: 7px;
      font-size: 15px;
      border: 1px solid #ccc;
      border-radius: 5px;
      box-sizing: border-box;
      margin-bottom: 8px;
    }
    .botoes {
      text-align: center;
      margin-top: 15px;
    }
    button {
      font-size: 16px;
      padding: 8px 18px;
      margin: 0 8px;
      border: none;
      border-radius: 5px;
      color: white;
      cursor: pointer;
      transition: background-color 0.3s;
    }
    button:disabled {
      background-color: #999;
      cursor: not-allowed;
    }
    button.iniciar {
      background-color: #3498db;
    }
    button.iniciar:hover:not(:disabled) {
      background-color: #2980b9;
    }
    button.parar {
      background-color: #e74c3c;
    }
    button.parar:hover:not(:disabled) {
      background-color: #c0392b;
    }
    .dados-inseridos {
      margin-top: 15px;
      font-size: 16px;
      color: #34495e;
      white-space: pre-wrap;
      min-height: 50px;
      border-top: 1px solid #eee;
      padding-top: 10px;
    }
  </style>
</head>
<body>

  <h1>TEMPO MÉDIO DE CARREGAMENTO SSP13 (Vários Contadores)</h1>

  <div id="contadores-wrapper" style="display: flex; flex-wrap: wrap; gap: 20px; justify-content: center;">
    <!-- contadores serão inseridos aqui -->
  </div>

  <div style="text-align:center; margin-top: 30px;">
    <button id="btnAdicionarContador" style="font-size:18px; padding: 10px 25px; border-radius: 6px; background:#27ae60; color:#fff; border:none; cursor:pointer;">
      + Adicionar novo contador
    </button>
  </div>

  <script>
    const URL_WEB_APP = "URL_DO_SEU_WEB_APP"; // Altere para sua URL real
    const SENHA_CORRETA = "MELI123";

    // Função para formatar segundos em hh:mm:ss
    function formatarTempo(segundos) {
      const hrs = Math.floor(segundos / 3600);
      const mins = Math.floor((segundos % 3600) / 60);
      const segs = segundos % 60;
      return String(hrs).padStart(2,'0') + ':' +
             String(mins).padStart(2,'0') + ':' +
             String(segs).padStart(2,'0');
    }

    // Função para atualizar cor de fundo da div do contador conforme tempo
    function corPorTempo(segundos, elemento) {
      if (segundos <= 15 * 60) {
        elemento.style.backgroundColor = 'green';
        elemento.style.color = 'white';
      } else if (segundos <= 25 * 60) {
        elemento.style.backgroundColor = 'yellow';
        elemento.style.color = '#333';
      } else {
        elemento.style.backgroundColor = 'red';
        elemento.style.color = 'white';
      }
    }

    // Função fábrica que cria um contador independente e retorna um objeto para controlar
    function criarContador(id) {
      // Criar container
      const container = document.createElement('div');
      container.className = 'contador-container';

      // Header do contador
      const header = document.createElement('div');
      header.className = 'contador-header';
      header.textContent = `Contador ${id}`;
      container.appendChild(header);

      // Elemento do tempo
      const tempoDisplay = document.createElement('div');
      tempoDisplay.className = 'contador-tempo';
      tempoDisplay.textContent = '00:00:00';
      container.appendChild(tempoDisplay);

      // Inputs
      const inputPlacaLabel = document.createElement('label');
      inputPlacaLabel.setAttribute('for', `placa-${id}`);
      inputPlacaLabel.textContent = 'Placa:';
      container.appendChild(inputPlacaLabel);
      const inputPlaca = document.createElement('input');
      inputPlaca.type = 'text';
      inputPlaca.id = `placa-${id}`;
      inputPlaca.placeholder = 'Digite a placa';
      container.appendChild(inputPlaca);

      const inputDocaLabel = document.createElement('label');
      inputDocaLabel.setAttribute('for', `doca-${id}`);
      inputDocaLabel.textContent = 'Doca:';
      container.appendChild(inputDocaLabel);
      const inputDoca = document.createElement('input');
      inputDoca.type = 'text';
      inputDoca.id = `doca-${id}`;
      inputDoca.placeholder = 'Digite a doca';
      container.appendChild(inputDoca);

      const inputTransportadoraLabel = document.createElement('label');
      inputTransportadoraLabel.setAttribute('for', `transportadora-${id}`);
      inputTransportadoraLabel.textContent = 'Transportadora:';
      container.appendChild(inputTransportadoraLabel);
      const inputTransportadora = document.createElement('input');
      inputTransportadora.type = 'text';
      inputTransportadora.id = `transportadora-${id}`;
      inputTransportadora.placeholder = 'Digite a transportadora';
      container.appendChild(inputTransportadora);

      // Botões
      const botoesDiv = document.createElement('div');
      botoesDiv.className = 'botoes';

      const botaoIniciar = document.createElement('button');
      botaoIniciar.textContent = 'Iniciar';
      botaoIniciar.className = 'iniciar';
      botaoIniciar.disabled = true;

      const botaoParar = document.createElement('button');
      botaoParar.textContent = 'Parar';
      botaoParar.className = 'parar';
      botaoParar.disabled = true;

      botoesDiv.appendChild(botaoIniciar);
      botoesDiv.appendChild(botaoParar);
      container.appendChild(botoesDiv);

      // Area para mostrar dados iniciados
      const dadosDiv = document.createElement('div');
      dadosDiv.className = 'dados-inseridos';
      container.appendChild(dadosDiv);

      // Estado do contador
      let intervalo = null;
      let totalSegundos = 0;
      const maxSegundos = 40 * 60; // 40 minutos (pode ajustar)

      // Função para validar campos e ativar botão iniciar
      function validarCampos() {
        const valido = 
          inputPlaca.value.trim() !== '' &&
          inputDoca.value.trim() !== '' &&
          inputTransportadora.value.trim() !== '';
        botaoIniciar.disabled = !valido;
      }

      // Atualizar cor e tempo
      function atualizarTempo() {
        totalSegundos++;
        tempoDisplay.textContent = formatarTempo(totalSegundos);
        corPorTempo(totalSegundos, tempoDisplay);

        if(totalSegundos >= maxSegundos) {
          clearInterval(intervalo);
          intervalo = null;
          botaoIniciar.disabled = false;
          botaoParar.disabled = true;
        }
      }

      // Evento inputs
      [inputPlaca, inputDoca, inputTransportadora].forEach(input => {
        input.addEventListener('input', validarCampos);
      });

      // Botão iniciar
      botaoIniciar.addEventListener('click', () => {
        if(intervalo) return;

        dadosDiv.textContent = 
          `Dados Iniciados:\nPlaca: ${inputPlaca.value.trim()}\nDoca: ${inputDoca.value.trim()}\nTransportadora: ${inputTransportadora.value.trim()}`;

        inputPlaca.disabled = true;
        inputDoca.disabled = true;
        inputTransportadora.disabled = true;

        botaoIniciar.disabled = true;
        botaoParar.disabled = false;

        intervalo = setInterval(atualizarTempo, 1000);
      });

      // Botão parar
      botaoParar.addEventListener('click', () => {
        if(!intervalo) return;

        const senha = prompt("Digite a senha para parar:");
        if(senha !== SENHA_CORRETA){
          alert("Senha incorreta! Operação cancelada.");
          return;
        }

        clearInterval(intervalo);
        intervalo = null;

        enviarDados({
          placa: inputPlaca.value.trim(),
          doca: inputDoca.value.trim(),
          transportadora: inputTransportadora.value.trim(),
          tempo: formatarTempo(totalSegundos)
        });

        // Resetar estado
        totalSegundos = 0;
        tempoDisplay.textContent = formatarTempo(totalSegundos);
        tempoDisplay.style.backgroundColor = '';
        tempoDisplay.style.color = '#2c3e50';

        inputPlaca.value = "";
        inputDoca.value = "";
        inputTransportadora.value = "";

        inputPlaca.disabled = false;
        inputDoca.disabled = false;
        inputTransportadora.disabled = false;

        dadosDiv.textContent = "";

        botaoIniciar.disabled = true;
        botaoParar.disabled = true;
      });

      // Inicializar estado
      botaoIniciar.disabled = true;
      botaoParar.disabled = true;

      return container;
    }

    // Função para enviar dados para URL_WEB_APP
    function enviarDados(dados) {
      fetch(URL_WEB_APP, {
        method: "POST",
        mode: "cors",
        headers: {
          "Content-Type": "application/json"
        },
        body: JSON.stringify(dados)
      })
      .then(resp => {
        if(!resp.ok) throw new Error(`HTTP error! Status: ${resp.status}`);
        return resp.json();
      })
      .then(data => {
        if(data.status === "success") {
          alert("Dados enviados com sucesso!");
        } else {
          alert("Erro ao enviar dados: " + (data.message || "Desconhecido"));
        }
      })
      .catch(e => {
        alert("Falha ao enviar dados: " + e.message);
      });
    }

    // Inserir contadores na página
    const wrapper = document.getElementById('contadores-wrapper');
    let contadorCount = 0;

    function adicionarContador() {
      contadorCount++;
      const novoContador = criarContador(contadorCount);
      wrapper.appendChild(novoContador);
    }

    document.getElementById('btnAdicionarContador').addEventListener('click', adicionarContador);

    // Adiciona um contador inicial para começar
    adicionarContador();

  </script>

</body>
</html>
