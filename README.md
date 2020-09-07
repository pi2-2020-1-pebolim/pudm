# Pebolim Unified data Model

O Pebolim Unified Data Model (PUDM) é uma modelagem de dados criada para permitir a utilização de diversos Machine Controllers na arquitetura da mesa de pebolim. A ideia deste modelo de dados surge a partir do conceito de Interface nas linguagem de programação orientadas a objetos, onde esta representa uma estrutura que funciona como um contrato, garantindo que as classes que a pertencem implementem determinados métodos. O PUDIM faz este papel de contrato, porém para os sistemas envolvidos na mesa de pebolim automatizada, governando como deve ser feita a comunicação entre o controlador desta mesa com o Servidor de Decisão e garantindo a compatibilidade entre os sistemas.

![Fluxograma](img/fluxogram.png)

## Definição de comunicação

O PUDM funciona em uma arquitetura cliente-servidor, envolvendo sistemas compatíves com o protocolo de comunicação. A comunicação é feita por meio de sockets, utilizando a tecnologia SocketIO.

### Convenções de Dados

#### Mesa
Como o PUDM representa dados reais, precisamos padrozinar as marcações de referência, por isso, sempre consideramos que a mesa de pebolim será retangular e representada da seguinte forma:


```plaintext

    0,0 ******************** x,0
        ********************
        ********************
        ********************
        ********************
maquina ********************  humano
        ********************
        ********************
        ********************
        ********************
    0,y                      x,y

```
Onde x representa a largura e y o comprimento da mesa.

#### Motores

O lado clinte do PUDM deve garantir que a posição inicial de um motor seja:
- Motores de rotação: boneco virado para baixo, perpendicular a mesa
- Motores de movimento: O boneco deve estar posicionado o mais próximo possível da lateral da maquina na mesa

Observe que, em geral, motores de movimentação são também de rotação, porém montados de forma diferente, geralmente movendo uma esteira. Para estes motores os comandos serão dados em relação a movimentação horizontal e será responsavilidade do PUDMClient converter nas rotações necessárias.



### PUDMServer

Lado servidor do PUDM, deve disponibilizar um endpoint de socket na tecnologia SocketIO. Antes de aceitar uma conexão de um cliente, deve verificar se os dados da versão são compatíveis.
Envia os eventos:
- ActionEvent


### PUDMCliente

Lado Cliente do PUDM, deve ser conectado ao PUDMServer antes de iniciar execução.
Envia os eventos:
- RegisterEvent
- StatusUpdateEvent


### Eventos

Os eventos são as mensagens enviadas por meio do socket. Eles enviam sua estrutura de dados no formato JSON. Dados comuns a todos os eventos:

```javascript

{
    "timestamp": int,
    "version": int,
    "eventType": string
}

```
Descrição dos campos:
- Timestamp: momento em que o evento foi gerado
- Version: versão do PUDM
- eventType: string representando o tipo do evento

#### EventTypes

Um EventType é a definição de o que determinado evento representa. Observe que todas as descrições a seguir contém os dados base citados anteriormente.

#### RegisterEvent

```javascript

{
    "evenType": "register",
    "fieldDefinition": {   // contem definições do campo de jogo
        "dimensions": [float, float], // largura e comprimento
    },
    "cameraSettings": { // informações da camera integrada a mesa
        "framerate": int,
        "resolution": [int, int]
    }
}

```

Observação: este evento precisa ser enviado com ACK, enquanto não receber confirmação do ACK, reenviar.

#### StatusUpdateEvent


```javascript

{
    "evenType": "status_update",
    "camera": {
        "image": string  // bytes da imagem codificados em uma string por meio de um encoder base64
    },
    "lanes": [ // descrição de cada linha controlada pela maquina
        {
            "laneID": int, // id da linha
            "yPosition": float // posição Y da linha 
            "playerPositions": [float, ...] // posição x para cada jogador, ordenado do menor para o maior
            "rotation": float // rotação atual da linha, sendo 0º o boneco na posição inicial
            "limits": [float, float] // posição X minima e posição X máxima para os jogadores
        }
    ]

}

```

#### ActionEvent

```javascript

{
    "evenType": "action",
    "desiredState": [ // estado que o PUDMServer deseja para a mesa 
        {
            "laneID": int, // id da linha que este comando ageta
            "position": float, // posição para onde a linha deve se mover
            "rotation": float // rotação da linha
        },
        ...
    ]
}

```


Comportamento: O PUDMClient sempre tentará chegar ao estado desejado mais recente e manter sua posição.
