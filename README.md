



<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Controle Remoto do Robô</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            background: #222;
            color: #fff;
            text-align: center;
            margin: 0;
            overflow: hidden;
        }

        h1 {
            color: #ffcc00;
            font-size: 36px;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.7);
            margin: 20px;
        }

        .joystick-outer-container {
            width: 150px;
            height: 150px;
            overflow: hidden;
            position: absolute;
            top: 50%;
            left: 10px;
            transform: translate(0, -50%);
            border-radius: 50%;
        }

        .joystick-container {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100%;
            position: relative;
        }
    </style>
</head>
<body>
    <h1>Controle Remoto do Robô</h1>

    <div class="joystick-outer-container">
        <div class="joystick-container" id="joystick-container-1">
            <!-- O primeiro joystick será inserido aqui -->
        </div>
    </div>

    <script src="https://code.jquery.com/jquery-3.6.4.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/nipplejs@0.10.1"></script>
    <script>
        document.addEventListener("DOMContentLoaded", function() {
            const joystickContainer = document.getElementById('joystick-container-1');
            const joystick = nipplejs.create({
                zone: joystickContainer,
                mode: 'static',
                position: { right: '75px', top: '50%' },
                size: 150
            });
            let joystickActive = false;

            // Estabeleça uma conexão WebSocket com o servidor na Raspberry Pi
            const socket = new WebSocket('ws://endereco_da_sua_raspberry_pi:porta');
            
            socket.onopen = function(event) {
                console.log('Conexão WebSocket aberta');
            };

            socket.onmessage = function(event) {
                // Lide com mensagens recebidas da Raspberry Pi, se necessário
                console.log('Mensagem recebida:', event.data);
            };

            socket.onerror = function(error) {
                console.error('Erro na conexão WebSocket:', error);
            };

            socket.onclose = function(event) {
                if (event.wasClean) {
                    console.log(`Conexão WebSocket fechada limpidamente, código=${event.code}, razão=${event.reason}`);
                } else {
                    console.error('Conexão WebSocket abrupta');
                }
            };

            joystick.on('start', function (evt, data) {
                joystickActive = true;
                console.log('Início do Joystick');
            });

            joystick.on('end', function (evt, data) {
                joystickActive = false;
                console.log('Fim do Joystick');
            });

            joystick.on('move', function (evt, data) {
                if (joystickActive) {
                    const angle = data.angle.degree;
                    const distance = data.distance;

                    // Envie comandos via WebSocket para a Raspberry Pi
                    const command = { type: 'move', angle, distance };
                    socket.send(JSON.stringify(command));
                }
            });

            // Adicione outros eventos ou lógica conforme necessário

            // Adicione um evento para fechar a conexão quando a página for fechada
            window.addEventListener('beforeunload', function() {
                socket.close();
            });
        });
    </script>
</body>
</html>
