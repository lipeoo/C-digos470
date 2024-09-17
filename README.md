# SCRIPTS
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  # Script do Personagem

Este script fornece controle de movimento para um personagem em Unity, incluindo movimento, animação e pulo. Utiliza o componente `CharacterController` para gerenciar o movimento e o pulo, e o componente `Animator` para ajustar as animações do personagem com base nas ações do jogador.

## Funcionalidades

- Movimento do personagem com base na orientação da câmera
- Animação de andar, correr e pular
- Aplicação de gravidade e controle do pulo
- Teleporte do personagem para a posição inicial se cair fora do mapa

## Código

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.VisualScripting;

public class Move : MonoBehaviour
{
    // Referência ao componente CharacterController usado para gerenciar o movimento
    CharacterController character;
    
    // Referência ao componente Animator usado para gerenciar animações
    Animator animator;
    
    // Armazena as entradas do jogador (movimento horizontal e vertical)
    Vector3 inputs;
    
    // Armazena a força e a direção do pulo
    Vector3 jump;

    // Referência à câmera para ajustar a direção do movimento
    public Transform cameraTransform; 

    // Velocidade padrão do movimento
    float velocidade = 5f;
    
    // Força aplicada ao pulo
    float jumpForce = 5f;
    
    // Valor da gravidade
    float gravity = -9.81f;
    
    // Estado que indica se o personagem está pulando
    bool isJumping = false;

    // Método chamado ao iniciar o jogo
    void Start()
    {
        // Obtém os componentes CharacterController e Animator do GameObject
        character = GetComponent<CharacterController>();
        animator = GetComponent<Animator>();
        
        // Inicializa o vetor de pulo como zero
        jump = Vector3.zero;
    }

    // Método chamado a cada frame
    void Update()
    {
        // Obtém os inputs do jogador para movimento horizontal e vertical
        inputs.Set(Input.GetAxis("Horizontal"), 0, Input.GetAxis("Vertical"));

        // Calcula a direção do movimento com base na orientação da câmera
        Vector3 moveDirection = cameraTransform.right * inputs.x + cameraTransform.forward * inputs.z;
        moveDirection.y = 0f; // Mantém o movimento no plano horizontal

        // Move o personagem na direção calculada
        character.Move(moveDirection * Time.deltaTime * velocidade);

        // Atualiza o estado da animação com base no input do jogador
        if (inputs != Vector3.zero)
        {
            // Ativa a animação de andar
            animator.SetBool("Andando", true);
            animator.SetBool("Parado", false);

            // Gira o personagem na direção do movimento
            transform.forward = moveDirection;
        }
        else
        {
            // Ativa a animação de parar
            animator.SetBool("Parado", true);
            animator.SetBool("Andando", false);
        }

        // Ajusta a velocidade e a animação se o jogador estiver correndo
        if (inputs != Vector3.zero && Input.GetKey(KeyCode.LeftShift))
        {
            velocidade = 9f; // Aumenta a velocidade para correr
            animator.SetBool("Parado", false);
            animator.SetBool("Correndo", true);
        }
        else
        {
            velocidade = 5f; // Define a velocidade padrão
            animator.SetBool("Correndo", false);
        }

        // Verifica se o personagem está no chão
        if (character.isGrounded)
        {
            // Zera a velocidade de queda ao tocar o chão
            jump.y = 0f;
            isJumping = false; // Reseta o estado de pulo

            // Inicia o pulo quando a tecla Espaço é pressionada
            if (Input.GetKey(KeyCode.Space))
            {
                jump.y = jumpForce; // Define a força do pulo
                isJumping = true;
                animator.SetBool("Pulando", true);
                animator.SetBool("Correndo", false);
                animator.SetBool("Andando", false);
                animator.SetBool("Parado", false);
            }
        }
        else
        {
            // Aplica a gravidade se o personagem não está no chão
            jump.y += gravity * Time.deltaTime;
            animator.SetBool("Pulando", false);
        }

        // Move o personagem com base no pulo e na gravidade
        character.Move(jump * Time.deltaTime);

        // Atualiza o estado da animação de pulo
        if (isJumping && character.isGrounded)
        {
            animator.SetBool("Pulando", false);
        }

        // Teleporta o personagem para a posição inicial se cair fora do mapa
        if (transform.position.y < -10f)
        {
            ForaDoMapa();
        }
    }

    // Método chamado para teletransportar o personagem de volta ao ponto inicial
    void ForaDoMapa()
    {
        // Desabilita o CharacterController para evitar problemas de colisão
        character.enabled = false;
        
        // Teleporta o personagem para a posição (0, 0, 0)
        transform.position = Vector3.zero;
        
        // Reabilita o CharacterController
        character.enabled = true;
    }
}
```
## Explicação do Código

### Declarações de Variáveis

- CharacterController character: Referência ao componente CharacterController que controla o movimento do personagem.
- Animator animator: Referência ao componente Animator que controla as animações do personagem.
- Vector3 inputs: Armazena as entradas de movimento do jogador (horizontal e vertical).
- Vector3 jump: Armazena a força e a direção do pulo.
- public Transform cameraTransform: Referência à câmera para ajustar a direção do movimento.
- float velocidade = 5f: Velocidade padrão para o movimento.
- float jumpForce = 5f: Força aplicada ao pulo.
- float gravity = -9.81f: Valor da gravidade para simular a queda.
- bool isJumping = false: Estado que indica se o personagem está pulando.

### Método `Start`

- Inicializa os componentes character e animator com os componentes do GameObject.
- Define jump como Vector3.zero.

### Método `Update`

- Obtém os inputs do jogador para movimento horizontal e vertical.
- Calcula a direção do movimento com base na orientação da câmera.
- Move o personagem na direção calculada.
- Atualiza o estado da animação com base nas ações do jogador.
- Gerencia o pulo e a aplicação de gravidade.
- Teleporta o personagem para a posição inicial se ele cair fora do mapa.

### Método `ForaDoMapa`

- Desabilita o CharacterController para evitar problemas de colisão.
- Teletransporta o personagem para a posição (0, 0, 0).
- Reabilita o CharacterController.


## Explicação das Contas

### Movimento e Velocidade

- **`moveDirection * Time.deltaTime * velocidade`**: 
  - **`moveDirection`**: Direção para onde o personagem deve se mover. Calculada com base na orientação da câmera e nas entradas do jogador.
  - **`Time.deltaTime`**: Tempo desde o último frame. Garante que o movimento seja suave e consistente, independentemente da taxa de quadros.
  - **`velocidade`**: Velocidade com que o personagem se move. Ajusta a distância percorrida por segundo.

  **Como funciona**: O movimento é ajustado pela `velocidade` e o tempo desde o último frame, garantindo uma movimentação uniforme e suave.

### Pulo e Gravidade

- **`jump.y = jumpForce`**: 
  - **`jump.y`**: Velocidade vertical do pulo.
  - **`jumpForce`**: Força aplicada ao pulo, determinando a altura do pulo.

- **`jump.y += gravity * Time.deltaTime`**:
  - **`gravity`**: Valor da gravidade que puxa o personagem para baixo.
  - **`Time.deltaTime`**: Garante que a aplicação da gravidade seja consistente ao longo do tempo.

  **Como funciona**: Quando o personagem está no chão, a velocidade de pulo é definida pela `jumpForce`. No ar, a gravidade é aplicada para simular a queda, ajustada pelo tempo decorrido desde o último frame.

## Explicação das Funções e Métodos

### `Input.GetAxis`

- **`Input.GetAxis("Horizontal")` e `Input.GetAxis("Vertical")`**:
  - Retornam valores entre -1 e 1 baseados na entrada horizontal e vertical do jogador (teclas de seta ou joystick).

  **Como funciona**: Obtém as entradas do jogador para movimentar o personagem, proporcionando um controle suave e contínuo.

### `GetComponent<T>`

- **`GetComponent<CharacterController>()` e `GetComponent<Animator>()`**:
  - Obtém o componente do tipo especificado (`CharacterController` ou `Animator`) no GameObject.

  **Como funciona**: Permite que o script interaja com outros componentes no GameObject, controlando o movimento e as animações do personagem.

### `character.Move(Vector3)`

- **`character.Move(moveDirection * Time.deltaTime * velocidade)`** e **`character.Move(jump * Time.deltaTime)`**:
  - Move o personagem na direção especificada pelo vetor `Vector3`.

  **Como funciona**: Aplica o movimento ao personagem com base nas entradas do jogador e na física do jogo, como gravidade e pulo.

### `animator.SetBool`

- **`animator.SetBool("Andando", true)`**:
  - Define um parâmetro booleano no `Animator`, controlando as animações.

  **Como funciona**: Ajusta o estado da animação do personagem de acordo com as ações do jogador (andar, correr, pular).

### `Input.GetKey`

- **`Input.GetKey(KeyCode.Space)`**:
  - Retorna `true` se a tecla especificada (neste caso, a tecla de espaço) estiver pressionada.

  **Como funciona**: Verifica se o jogador pressionou a tecla para executar uma ação, como pular.

### `character.isGrounded`

- **`character.isGrounded`**:
  - Retorna `true` se o personagem estiver em contato com o chão.

  **Como funciona**: Verifica se o personagem está no chão, necessário para determinar se o pulo pode ser iniciado ou se a gravidade deve ser aplicada.

### `ForaDoMapa`

- **Método para teletransportar o personagem**:
  - Desativa o `CharacterController`, teletransporta o personagem para a posição `(0, 0, 0)`, e reativa o `CharacterController`.

  **Como funciona**: Teletransporta o personagem de volta ao ponto inicial se ele cair fora do mapa, evitando que o personagem se perca ou fique preso fora do cenário.

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  

  # Script de Câmera

Este script controla a câmera para seguir o jogador em um jogo Unity. Ele ajusta a posição e a rotação da câmera com base na posição do jogador e na entrada do mouse.

## Código e Explicação

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CameraFollow : MonoBehaviour
{
    // Referência ao transform do jogador
    public Transform player;

    // Distância da câmera ao jogador
    public float distance = 10.0f;  // Define a distância da câmera ao jogador
    public float height = 5.0f;     // Define a altura da câmera em relação ao jogador

    // Sensibilidade da rotação da câmera com o mouse
    public float mouseSensitivity = 100.0f;  // Controla a sensibilidade da rotação da câmera com o mouse

    // Limite de rotação vertical
    public float minYAngle = -20f;  // Limita a rotação vertical mínima
    public float maxYAngle = 60f;   // Limita a rotação vertical máxima

    private float currentX = 0f;  // Armazena a rotação atual no eixo X
    private float currentY = 0f;  // Armazena a rotação atual no eixo Y

    void Start()
    {
        // Inicializa a posição do mouse (opcional)
        Cursor.lockState = CursorLockMode.Locked;  // Tranca o cursor no centro da tela
    }

    void Update()
    {
        // Atualiza os valores de rotação com base no movimento do mouse
        currentX += Input.GetAxis("Mouse X") * mouseSensitivity * Time.deltaTime;
        currentY -= Input.GetAxis("Mouse Y") * mouseSensitivity * Time.deltaTime;

        // Limita a rotação vertical para não ultrapassar os limites definidos
        currentY = Mathf.Clamp(currentY, minYAngle, maxYAngle);
    }

    void LateUpdate()
    {
        // Cria uma posição de offset com base na rotação calculada pelo mouse
        Vector3 direction = new Vector3(0, height, -distance);  // Define a posição da câmera com base na altura e distância
        Quaternion rotation = Quaternion.Euler(currentY, currentX, 0);  // Cria uma rotação baseada nas entradas do mouse

        // Aplica a posição e rotação ao transform da câmera
        transform.position = player.position + rotation * direction;  // Calcula a posição final da câmera
        transform.LookAt(player.position);  // Faz a câmera olhar para o jogador
    }
}
```
## Explicação do Código

### Declarações de Variáveis

- **`public Transform player`**: Referência ao transform do jogador. Determina a posição do jogador para ajustar a posição da câmera.
- **`public float distance = 10.0f`**: Distância da câmera em relação ao jogador. Define o quão longe a câmera fica do jogador.
- **`public float height = 5.0f`**: Altura da câmera em relação ao jogador. Ajusta a altura da câmera acima do jogador.
- **`public float mouseSensitivity = 100.0f`**: Sensibilidade da rotação da câmera com o mouse. Controla a rapidez com que a câmera reage ao movimento do mouse.
- **`public float minYAngle = -20f`**: Limite mínimo para a rotação vertical da câmera. Impede que a câmera gire muito para cima.
- **`public float maxYAngle = 60f`**: Limite máximo para a rotação vertical da câmera. Impede que a câmera gire muito para baixo.

### Método `Start`

- **`Cursor.lockState = CursorLockMode.Locked`**: Inicializa a posição do cursor do mouse para que ele fique preso no centro da tela durante o jogo.

### Método `Update`

- **`currentX += Input.GetAxis("Mouse X") * mouseSensitivity * Time.deltaTime`**: Atualiza o valor da rotação horizontal da câmera com base no movimento do mouse.
- **`currentY -= Input.GetAxis("Mouse Y") * mouseSensitivity * Time.deltaTime`**: Atualiza o valor da rotação vertical da câmera com base no movimento do mouse.
- **`currentY = Mathf.Clamp(currentY, minYAngle, maxYAngle)`**: Limita a rotação vertical da câmera para que não ultrapasse os ângulos definidos por `minYAngle` e `maxYAngle`.

### Método `LateUpdate`

- **`Vector3 direction = new Vector3(0, height, -distance)`**: Cria uma direção para a posição da câmera com base na altura e distância do jogador.
- **`Quaternion rotation = Quaternion.Euler(currentY, currentX, 0)`**: Cria uma rotação para a câmera com base nos valores calculados de `currentY` e `currentX`.
- **`transform.position = player.position + rotation * direction`**: Ajusta a posição da câmera para seguir o jogador, aplicando a rotação calculada.
- **`transform.LookAt(player.position)`**: Faz com que a câmera olhe para a posição do jogador, garantindo que o jogador esteja sempre no centro da tela.

## Explicação das Contas

### Roteamento da Câmera

- **`currentX += Input.GetAxis("Mouse X") * mouseSensitivity * Time.deltaTime` e `currentY -= Input.GetAxis("Mouse Y") * mouseSensitivity * Time.deltaTime`**:
  - **`Input.GetAxis("Mouse X")` e `Input.GetAxis("Mouse Y")`**: Obtêm os valores do movimento do mouse, que variam entre -1 e 1.
  - **`mouseSensitivity`**: Multiplica a entrada do mouse para ajustar a rapidez da rotação.
  - **`Time.deltaTime`**: Garante que a rotação seja ajustada suavemente, independentemente da taxa de quadros.
  - **Como funciona**: O valor de rotação da câmera é ajustado com base no movimento do mouse e na sensibilidade configurada, garantindo uma resposta suave e proporcional ao movimento do jogador.

### Limitação da Rotação Vertical

- **`currentY = Mathf.Clamp(currentY, minYAngle, maxYAngle)`**:
  - **`Mathf.Clamp`**: Limita o valor de `currentY` para ficar dentro dos limites definidos por `minYAngle` e `maxYAngle`.
  - **Como funciona**: Impede que a rotação vertical da câmera ultrapasse os limites definidos, evitando ângulos de visão desconfortáveis ou impraticáveis.

### Posicionamento e Rotação da Câmera

- **`transform.position = player.position + rotation * direction`**:
  - **`rotation * direction`**: Aplica a rotação calculada ao vetor de direção para determinar a posição final da câmera.
  - **Como funciona**: Posiciona a câmera atrás e acima do jogador, ajustando a altura e a distância com base na rotação atual para fornecer uma visão adequada.

## Explicação das Funções e Métodos

### `Input.GetAxis`

- **`Input.GetAxis("Mouse X")` e `Input.GetAxis("Mouse Y")`**:
  - **`Input.GetAxis("Mouse X")`**: Obtém a entrada do movimento horizontal do mouse. Retorna um valor entre -1 e 1 que representa a direção e a magnitude do movimento horizontal.
  - **`Input.GetAxis("Mouse Y")`**: Obtém a entrada do movimento vertical do mouse. Retorna um valor entre -1 e 1 que representa a direção e a magnitude do movimento vertical.

  **Como funciona**: Esses métodos fornecem os dados de entrada do mouse, que são usados para calcular a rotação da câmera. A rotação da câmera é ajustada com base nesses valores, permitindo que a câmera siga o movimento do mouse.

### `Cursor.lockState`

- **`Cursor.lockState = CursorLockMode.Locked`**:
  - **`CursorLockMode.Locked`**: Define o estado do cursor para que ele fique preso no centro da tela e não se mova fora da janela do jogo.

  **Como funciona**: Ao bloquear o cursor, o script evita que o cursor se mova fora da área de visualização da câmera, proporcionando uma experiência de jogo mais fluida e controlada. Isso é especialmente útil para jogos em primeira pessoa ou para controle de câmera com o mouse.

### `Mathf.Clamp`

- **`Mathf.Clamp(currentY, minYAngle, maxYAngle)`**:
  - **`Mathf.Clamp`**: Garante que o valor de `currentY` permaneça dentro do intervalo definido pelos parâmetros `minYAngle` e `maxYAngle`.

  **Como funciona**: Este método impede que o valor de rotação vertical da câmera ultrapasse os limites definidos, evitando que a câmera se incline excessivamente para cima ou para baixo e proporcionando uma visão mais controlada e confortável.

### `Quaternion.Euler`

- **`Quaternion.Euler(currentY, currentX, 0)`**:
  - **`Quaternion.Euler`**: Cria uma rotação em torno dos eixos X, Y e Z com base nos valores fornecidos. Neste caso, `currentY` e `currentX` determinam a rotação vertical e horizontal, respectivamente.

  **Como funciona**: Este método converte os ângulos de rotação em um quaternion, que é uma representação matemática da rotação. Isso permite que a câmera seja orientada na direção desejada com base nas entradas do mouse.

### `transform.position`

- **`transform.position = player.position + rotation * direction`**:
  - **`rotation * direction`**: Aplica a rotação calculada ao vetor de direção para determinar a posição final da câmera.

  **Como funciona**: Ajusta a posição da câmera com base na posição do jogador e na rotação calculada. A câmera é posicionada atrás e acima do jogador, criando uma visão ajustada e controlada com base na rotação atual.

### `transform.LookAt`

- **`transform.LookAt(player.position)`**:
  - **`transform.LookAt`**: Faz com que a câmera olhe para um ponto específico no espaço, neste caso, a posição do jogador.

  **Como funciona**: Garante que a câmera esteja sempre voltada para o jogador, mantendo-o no centro da tela e proporcionando uma perspectiva consistente durante o jogo.
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Script das Luzes

Este script controla a ativação de luzes spot na cena com base na proximidade do jogador. As luzes são ativadas quando o jogador se aproxima dentro de uma distância especificada.

## Funcionalidades

- Desativa todas as luzes spot ao iniciar o jogo.
- Ativa as luzes spot quando o jogador está dentro de uma distância definida.

## Código

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Light : MonoBehaviour
{
    public float activationDistance = 20f;  // Distância em que as luzes são ativadas
    public Transform player;                // Referência do jogador

    private UnityEngine.Light[] spotLights; // Array para armazenar todas as Spot Lights

    void Start()
    {
        // Busca todas as luzes na cena
        spotLights = FindObjectsOfType<UnityEngine.Light>();

        // Certifica-se de que todas as luzes comecem desligadas
        foreach (UnityEngine.Light light in spotLights)
        {
            if (light.type == LightType.Spot)  // Usando explicitamente o LightType do Unity
            {
                light.enabled = false;
            }
        }
    }

    void Update()
    {
        // Verifica a distância do jogador para cada Spot Light na cena
        foreach (UnityEngine.Light light in spotLights)
        {
            if (light.type == LightType.Spot)  // Usando explicitamente o LightType do Unity
            {
                float distanceToPlayer = Vector3.Distance(player.position, light.transform.position);

                // Se o jogador estiver dentro da distância de ativação, liga a luz
                if (distanceToPlayer <= activationDistance)
                {
                    light.enabled = true;
                }
                else
                {
                    light.enabled = false;
                }
            }
        }
    }
}
```

## Explicação do Código

### Declarações de Variáveis

- **`public float activationDistance = 20f`**: Define a distância a partir da qual as luzes spot são ativadas. Esse valor especifica o raio dentro do qual as luzes spot serão ligadas quando o jogador se aproxima.
- **`public Transform player`**: Referência ao transform do jogador. Usado para calcular a distância entre o jogador e as luzes spot na cena.
- **`private UnityEngine.Light[] spotLights`**: Array para armazenar todas as luzes spot na cena. Armazena os componentes `Light` encontrados na cena para serem manipulados pelo script.

### Método `Start`

- **`spotLights = FindObjectsOfType<UnityEngine.Light>()`**: Encontra e armazena todos os componentes `Light` presentes na cena. Isso permite que todas as luzes sejam manipuladas pelo script.
- **`foreach (UnityEngine.Light light in spotLights)`**: Itera sobre todas as luzes encontradas na cena.
  - **`if (light.type == LightType.Spot)`**: Verifica se a luz é do tipo Spot. Isso é necessário para garantir que apenas as luzes spot sejam manipuladas.
  - **`light.enabled = false`**: Desliga todas as luzes spot inicialmente para garantir que comecem desligadas.

### Método `Update`

- **`foreach (UnityEngine.Light light in spotLights)`**: Itera sobre todas as luzes armazenadas no array.
  - **`if (light.type == LightType.Spot)`**: Verifica se a luz é do tipo Spot. Isso garante que apenas as luzes spot sejam avaliadas.
  - **`float distanceToPlayer = Vector3.Distance(player.position, light.transform.position)`**:
    - **`player.position`**: Posição atual do jogador.
    - **`light.transform.position`**: Posição atual da luz.
    - **`Vector3.Distance`**: Calcula a distância entre o jogador e a luz.
  - **`if (distanceToPlayer <= activationDistance)`**: Verifica se o jogador está dentro da distância de ativação definida.
    - **`light.enabled = true`**: Liga a luz se o jogador estiver dentro da distância de ativação.
    - **`else`**: Caso o jogador esteja fora da distância de ativação.
      - **`light.enabled = false`**: Desliga a luz se o jogador estiver fora da distância de ativação.

## Explicação das Contas

### Movimento e Distância

- **`Vector3.Distance(player.position, light.transform.position)`**:
  - **`player.position`**: Representa a posição do jogador no espaço 3D.
  - **`light.transform.position`**: Representa a posição da luz spot no espaço 3D.
  - **`Vector3.Distance`**: Calcula a distância euclidiana entre o jogador e a luz. Esta distância é essencial para determinar se a luz deve ser ativada com base na proximidade do jogador.

  **Como funciona**: A distância entre o jogador e a luz é calculada utilizando a fórmula da distância euclidiana, que mede o comprimento da linha reta entre dois pontos no espaço 3D. Esse valor é comparado com a `activationDistance` para decidir se a luz deve ser ligada ou desligada.

### Controle de Ativação da Luz

- **`if (distanceToPlayer <= activationDistance)`**:
  - **`distanceToPlayer`**: Valor obtido pela função `Vector3.Distance`, representando a distância entre o jogador e a luz.
  - **`activationDistance`**: Valor fixo definido na variável pública que especifica o raio de ativação das luzes.

  **Como funciona**: Se a distância calculada (`distanceToPlayer`) é menor ou igual ao valor de `activationDistance`, isso significa que a luz está dentro do alcance do jogador e deve ser ativada (`light.enabled = true`). Caso contrário, a luz é desativada (`light.enabled = false`).

## Explicação das Funções e Métodos

### `FindObjectsOfType<T>()`

- **`FindObjectsOfType<UnityEngine.Light>()`**: Retorna um array contendo todos os objetos do tipo especificado (`Light`) encontrados na cena.
  - **Como funciona**: Permite encontrar e armazenar todas as luzes na cena para que possam ser manipuladas pelo script.

### `Vector3.Distance`

- **`Vector3.Distance(player.position, light.transform.position)`**:
  - **`player.position`**: Posição do jogador no espaço.
  - **`light.transform.position`**: Posição da luz no espaço.
  - **`Vector3.Distance`**: Calcula a distância euclidiana entre o jogador e a luz. Esse valor é utilizado para determinar se a luz deve ser ativada com base na proximidade do jogador.

### `foreach`

- **`foreach (UnityEngine.Light light in spotLights)`**: Itera sobre todos os componentes `Light` armazenados no array `spotLights`.
  - **Como funciona**: Permite realizar uma ação para cada luz spot na cena, como verificar a distância do jogador e ativar ou desativar a luz conforme necessário.

### `light.enabled`

- **`light.enabled = true` e `light.enabled = false`**: Propriedade que controla se a luz está ligada ou desligada.
  - **Como funciona**: Liga ou desliga as luzes spot com base na distância do jogador. Se o jogador está dentro do raio definido (`activationDistance`), a luz é ativada; caso contrário, é desativada.
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
