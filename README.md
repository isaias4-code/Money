
<!DOCTYPE html>
<html>
<head>
  <title>Sinal Automático Deriv com Telegram</title>
</head>
<body style="font-family: sans-serif; text-align: center; padding-top: 50px;">
  <h2>Sugestão automática para Deriv (DIGITEVEN / DIGITODD)</h2>
  <button id="sugestao" style="font-size: 30px; padding: 20px 40px;">Aguardando...</button>
  <p id="info"></p>

  <!-- Alerta sonoro -->
  <audio id="alerta" src="https://www.soundjay.com/buttons/sounds/button-16.mp3" preload="auto"></audio>

  <script>
    const token = "7841818902:AAGrFwN9h6kc-x9BTLuM6W-ualsqRMiq5TM";
    const chatId = "7841818902";

    function enviarTelegram(mensagem) {
      const url = `https://api.telegram.org/bot${token}/sendMessage`;
      const params = {
        chat_id: chatId,
        text: mensagem,
      };
      fetch(url, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(params),
      });
    }

    const ws = new WebSocket("wss://ws.binaryws.com/websockets/v3?app_id=1089");
    const simbolo = "R_100";
    let ultimosDigitos = [];
    let sugestaoAnterior = "";

    const btn = document.getElementById("sugestao");
    const info = document.getElementById("info");
    const alertaSom = document.getElementById("alerta");

    ws.onopen = () => {
      ws.send(JSON.stringify({ ticks: simbolo, subscribe: 1 }));
    };

    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      if (data.msg_type === "tick") {
        const valor = data.tick.quote.toFixed(2);
        const digito = parseInt(valor.slice(-1));
        ultimosDigitos.push(digito);
        if (ultimosDigitos.length > 25) ultimosDigitos.shift();

        if (ultimosDigitos.length === 25) {
          const pares = ultimosDigitos.filter(n => n % 2 === 0).length;
          const impares = 25 - pares;
          const probPar = pares / 25;
          const sugestao = probPar >= 0.5 ? "DIGITEVEN" : "DIGITODD";

          if (sugestao !== sugestaoAnterior) {
            alertaSom.play();
            sugestaoAnterior = sugestao;
            const mensagem = `Nova sugestão: ${sugestao}\nPares: ${pares} | Ímpares: ${impares}\nProb. Par: ${(probPar * 100).toFixed(1)}%`;
            enviarTelegram(mensagem);
          }

          btn.textContent = sugestao;
          btn.style.backgroundColor = sugestao === "DIGITEVEN" ? "#28a745" : "#dc3545";
          btn.style.color = "#fff";
          info.innerHTML = `Pares: ${pares} | Ímpares: ${impares}<br>Prob. Par: ${(probPar * 100).toFixed(1)}%`;
        }
      }
    };
  </script>
</body>
</html>
