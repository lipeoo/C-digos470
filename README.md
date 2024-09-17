# SCRIPTS
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  # Script do Personagem

Este script fornece controle para o personagem principal, incluindo movimento, animação e pulo. Utiliza o componente `CharacterController` para gerenciar o movimento e o pulo, e o componente `Animator` para ajustar as animações do personagem com base nas ações do jogador.

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
- Vector3 inputs: Váriavel que armazena as entradas de movimento do jogador (horizontal e vertical).
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
- Chama o método 'ForaDoMapa' que teleporta o personagem para a posição inicial se ele cair fora do mapa.

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

Este script controla a câmera para seguir o jogador. Ele ajusta a posição e a rotação da câmera com base na posição do jogador e na entrada do mouse.

## Código

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

  **Como funciona**: Ao bloquear o cursor, o script evita que o cursor se mova fora da área de visualização da câmera. Isso é especialmente útil para jogos em primeira pessoa ou para controle de câmera com o mouse.

### `Mathf.Clamp`

- **`Mathf.Clamp(currentY, minYAngle, maxYAngle)`**:
  - **`Mathf.Clamp`**: Garante que o valor de `currentY` permaneça dentro do intervalo definido pelos parâmetros `minYAngle` e `maxYAngle`.

  **Como funciona**: Este método impede que o valor de rotação vertical da câmera ultrapasse os limites definidos, evitando que a câmera se incline excessivamente para cima ou para baixo.

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

Este script controla a ativação de luzes spot na cena com base na proximidade do jogador. As luzes são ativadas quando o jogador se aproxima dentro de uma distância especificada. Foi feito com o objetivo de fazer o jogo funcionar de forma mais leve (PC travava).

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

- **`foreach (UnityEngine.Light light in spotLights)`**: Repete sobre todas as luzes armazenadas no array.
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

- **`foreach (UnityEngine.Light light in spotLights)`**: Repete sobre todos os componentes `Light` armazenados no array `spotLights`.
  - **Como funciona**: Permite realizar uma ação para cada luz spot na cena, como verificar a distância do jogador e ativar ou desativar a luz conforme necessário.

### `light.enabled`

- **`light.enabled = true` e `light.enabled = false`**: Propriedade que controla se a luz está ligada ou desligada.
  - **Como funciona**: Liga ou desliga as luzes spot com base na distância do jogador. Se o jogador está dentro do raio definido (`activationDistance`), a luz é ativada; caso contrário, é desativada.
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Script do Botão Menu

Este script controla a funcionalidade de um botão. Permite que o botão inicie uma cena específica ou saia do jogo quando clicado. Utiliza o `SceneManager` para gerenciar as cenas e o `Application` para sair do jogo.

## Funcionalidades

- Carregamento da cena do jogo quando o botão é pressionado
- Saída do jogo e exibição de uma mensagem de log quando o botão é pressionado

## Código

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class Button : MonoBehaviour
{
    // Função para carregar a cena do jogo
    public void StartGame()
    {
        SceneManager.LoadScene("game"); 
    }

    // Função para sair do jogo
    public void QuitGame()
    {
        Debug.Log("Saindo do Jogo..."); 
        Application.Quit(); 
    }
}
```
## Explicação do Código

### Declarações de Variáveis

Não há variáveis declaradas no script. Todas as funcionalidades são implementadas diretamente nos métodos.

### Método `StartGame`

- **`SceneManager.LoadScene("game")`**:
  - **`SceneManager`**: Classe responsável por gerenciar as cenas no Unity.
  - **`LoadScene`**: Método que carrega a cena especificada pelo nome ("game").
  
  **Como funciona**: Quando o método `StartGame` é chamado, ele solicita ao `SceneManager` que carregue a cena com o nome "game". Isso altera a cena atual para a nova cena especificada, permitindo a transição entre diferentes partes do jogo.

### Método `QuitGame`

- **`Debug.Log("Saindo do Jogo...")`**:
  - **`Debug`**: Classe usada para emitir mensagens de log no console do Unity.
  - **`Log`**: Método que exibe uma mensagem no console.

  **Como funciona**: Quando o método `QuitGame` é chamado, ele exibe a mensagem "Saindo do Jogo..." no console do Unity.

- **`Application.Quit()`**:
  - **`Application`**: Classe que fornece métodos para interagir com a aplicação.
  - **`Quit`**: Método que encerra a aplicação.

  **Como funciona**: Após exibir a mensagem de log, o método `QuitGame` chama `Application.Quit()`, que encerra o aplicativo ou jogo.
## Explicação das Funções e Métodos

### `SceneManager.LoadScene`

- **`SceneManager.LoadScene("game")`**:
  - Carrega a cena com o nome especificado ("game").

  **Como funciona**: Permite iniciar uma nova cena no jogo.

### `Debug.Log`

- **`Debug.Log("Saindo do Jogo...")`**:
  - Exibe uma mensagem no console do Unity para fins de depuração.

  **Como funciona**: Útil para fornecer feedback durante o desenvolvimento e depuração, mostrando que uma ação (neste caso, sair do jogo) está prestes a ocorrer.

### `Application.Quit`

- **`Application.Quit()`**:
  - Encerra a aplicação.

  **Como funciona**: Utilizado para sair do jogo quando o botão é pressionado, fechando o aplicativo de forma programática.
## Explicação das Contas

O script não realiza cálculos matemáticos ou operações complexas.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Script de Troca de Cena

Este script permite que um jogador troque de cena quando se aproxima de uma porta e pressiona a tecla "E". Utiliza o sistema de colisão do Unity para detectar quando o jogador está perto da porta e carrega uma nova cena quando a interação é confirmada.

## Funcionalidades

- Detecta se o jogador está perto de um objeto marcado como porta.
- Permite que o jogador carregue uma nova cena ao pressionar a tecla "E".
- Exibe uma mensagem no console indicando que a tecla deve ser pressionada para abrir a porta.

## Código

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class ChangeMap : MonoBehaviour
{
    private bool playerNearby = false;  // Verifica se o jogador está perto da porta
    public string sceneName;  // Nome da cena para a qual a porta vai levar

    void Update()
    {
        // Verifica se o jogador está perto da porta e apertou "E"
        if (playerNearby && Input.GetKeyDown(KeyCode.E))
        {
            LoadScene();  // Carrega a nova cena
        }
    }

    // Detecta se o jogador entrou no Trigger da porta
    private void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            playerNearby = true;  // Marca que o jogador está perto da porta
            Debug.Log("Aperte 'E' para abrir a porta.");
        }
    }

    // Detecta se o jogador saiu do Trigger da porta
    private void OnTriggerExit(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            playerNearby = false;  // Marca que o jogador não está mais perto da porta
        }
    }

    // Método para carregar a cena
    private void LoadScene()
    {
        if (!string.IsNullOrEmpty(sceneName))  // Verifica se o nome da cena não está vazio
        {
            SceneManager.LoadScene(sceneName);  // Carrega a cena especificada
        }
        else
        {
            Debug.LogError("Nome da cena não foi especificado!");
        }
    }
}
```
## Explicação do Código

### Declarações de Variáveis

- **`private bool playerNearby`**: Verifica se o jogador está perto da porta. Inicialmente configurado como `false` e atualizado quando o jogador entra ou sai da área de colisão da porta.
- **`public string sceneName`**: Nome da cena para a qual a porta levará. Este valor deve ser definido no Inspetor do Unity ou por outro meio.

### Método `Update`

- **`if (playerNearby && Input.GetKeyDown(KeyCode.E))`**: Verifica se o jogador está próximo da porta e se a tecla "E" foi pressionada. Se ambas as condições forem verdadeiras, o método `LoadScene` é chamado para carregar a nova cena.

### Método `OnTriggerEnter`

- **`private void OnTriggerEnter(Collider other)`**: Detecta quando o jogador entra no trigger da porta.
  - **`if (other.CompareTag("Player"))`**: Verifica se o objeto que entrou no trigger é o jogador.
    - **`playerNearby = true`**: Marca que o jogador está perto da porta.
    - **`Debug.Log("Aperte 'E' para abrir a porta.")`**: Exibe uma mensagem no console para o jogador.

### Método `OnTriggerExit`

- **`private void OnTriggerExit(Collider other)`**: Detecta quando o jogador sai do trigger da porta.
  - **`if (other.CompareTag("Player"))`**: Verifica se o objeto que saiu do trigger é o jogador.
    - **`playerNearby = false`**: Marca que o jogador não está mais perto da porta.

### Método `LoadScene`

- **`private void LoadScene()`**: Carrega a cena especificada pelo nome.
  - **`if (!string.IsNullOrEmpty(sceneName))`**: Verifica se o nome da cena não está vazio.
    - **`SceneManager.LoadScene(sceneName)`**: Carrega a cena especificada.
    - **`Debug.LogError("Nome da cena não foi especificado!")`**: Exibe uma mensagem de erro se o nome da cena não estiver definido.

## Explicação das Contas

### Detecção de Proximidade e Interação

- **`if (playerNearby && Input.GetKeyDown(KeyCode.E)`**:
  - **`playerNearby`**: Valor booleano que indica se o jogador está perto da porta.
  - **`Input.GetKeyDown(KeyCode.E)`**: Verifica se a tecla "E" foi pressionada.

  **Como funciona**: O código verifica se o jogador está na proximidade da porta e se a tecla "E" foi pressionada. Se ambas as condições forem verdadeiras, a cena especificada é carregada.

### Método `LoadScene`

- **`SceneManager.LoadScene(sceneName)`**:
  - **`sceneName`**: Nome da cena a ser carregada.

  **Como funciona**: Utiliza o `SceneManager` para carregar a cena especificada pelo nome. Se o nome da cena estiver vazio, exibe um erro no console.

## Explicação das Funções e Métodos

### `Input.GetKeyDown`

- **`Input.GetKeyDown(KeyCode.E)`**:
  - Retorna `true` se a tecla especificada (neste caso, "E") for pressionada durante o frame atual.

  **Como funciona**: Detecta a entrada do jogador para realizar a troca de cena, garantindo que a ação de carregar a cena ocorra apenas quando a tecla é pressionada.

### `OnTriggerEnter`

- **`OnTriggerEnter(Collider other)`**:
  - Chamado automaticamente pelo Unity quando um objeto entra na área de colisão do trigger.

  **Como funciona**: Permite que o script detecte quando o jogador está perto da porta, ativando a interação e exibindo uma mensagem no console.

### `OnTriggerExit`

- **`OnTriggerExit(Collider other)`**:
  - Chamado automaticamente pelo Unity quando um objeto sai da área de colisão do trigger.

  **Como funciona**: Permite que o script detecte quando o jogador não está mais perto da porta, desativando a interação.

### `SceneManager.LoadScene`

- **`SceneManager.LoadScene(sceneName)`**:
  - Carrega a cena especificada pelo nome.

  **Como funciona**: Utiliza o sistema de gerenciamento de cenas do Unity para mudar para a cena desejada, baseada no nome fornecido.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Script de Diálogo

Este script gerencia um sistema de diálogo em Unity, permitindo que o jogador interaja com um NPC para visualizar e avançar através de linhas de diálogo. Também inclui a funcionalidade para o NPC olhar para o jogador e dar um item quando o diálogo é concluído.

## Funcionalidades

- Exibe um painel de diálogo quando o jogador está perto e pressiona a tecla "E".
- Avança para a próxima linha de diálogo ou oculta o painel ao terminar o diálogo.
- Faz o NPC olhar para o jogador quando ele está próximo.
- Permite ao NPC dar um item ao jogador após o término do diálogo.

## Código

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using TMPro;
using UnityEngine.UI;
using UnityEngine.SceneManagement;

public class Dialogue : MonoBehaviour
{
    public GameObject dialogueUI;               // Painel da UI de diálogo
    public TextMeshProUGUI dialogueText;        // Campo de texto para o diálogo
    public string[] dialogueLines;              // Linhas de diálogo que serão exibidas
    private int currentLineIndex = 0;           // Índice da linha atual do diálogo
    private bool playerNearby = false;          // Flag para verificar se o jogador está perto
    private bool dialogueActive = false;        // Para controlar se o diálogo está ativo ou não
    public bool hasItem = false;                // Boolean para verificar se o jogador já tem o item

    public Transform player;                    // Referência ao jogador (para o fantasma olhar para ele)

    void Start()
    {
        dialogueUI.SetActive(false);  // Começa com o diálogo oculto
    }

    void Update()
    {
        // Verifica se o jogador está perto e aperta "E"
        if (playerNearby && Input.GetKeyDown(KeyCode.E))
        {
            // Se o diálogo não estiver ativo, ativar e mostrar a primeira linha
            if (!dialogueActive)
            {
                dialogueUI.SetActive(true);
                dialogueText.text = dialogueLines[currentLineIndex];
                dialogueActive = true;  // Marca o diálogo como ativo
            }
            // Se o diálogo estiver ativo, avançar para a próxima linha
            else
            {
                currentLineIndex++;

                // Se houver mais linhas de diálogo, atualizar o texto
                if (currentLineIndex < dialogueLines.Length)
                {
                    dialogueText.text = dialogueLines[currentLineIndex];
                }
                // Se acabar as linhas de diálogo, ocultar o UI, resetar e dar o item ao jogador
                else
                {
                    dialogueUI.SetActive(false);
                    currentLineIndex = 0;
                    dialogueActive = false;

                    // Dar o item ao jogador
                    if (!hasItem)
                    {
                        hasItem = true;  // O jogador ganha o item
                        Debug.Log("Item recebido!");
                    }
                }
            }
        }

        // O fantasma deve sempre olhar para o jogador se ele estiver por perto
        if (playerNearby)
        {
            LookAtPlayer();
        }
    }

    // Detecta quando o jogador entra na área do fantasma
    private void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            playerNearby = true;  // Jogador está perto
        }
    }

    // Detecta quando o jogador sai da área do fantasma
    private void OnTriggerExit(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            playerNearby = false;  // Jogador não está mais perto
            dialogueUI.SetActive(false);  // Oculta o diálogo se o jogador sair da área
            currentLineIndex = 0;  // Reseta o diálogo para o começo
            dialogueActive = false;  // Garante que o diálogo seja desativado
        }
    }

    // Faz o fantasma olhar para o jogador
    private void LookAtPlayer()
    {
        Vector3 direction = player.position - transform.position; // Direção do jogador
        direction.y = 0;  // Para evitar rotação no eixo Y (vertical)
        Quaternion rotation = Quaternion.LookRotation(direction); // Calcula a rotação para olhar na direção do jogador
        transform.rotation = Quaternion.Slerp(transform.rotation, rotation, Time.deltaTime * 5f); // Suaviza a rotação
    }
}
```
## Explicação do Código

### Declarações de Variáveis

- **`public GameObject dialogueUI`**: 
  - Painel da UI de diálogo que será mostrado ou ocultado durante a interação com o jogador.

- **`public TextMeshProUGUI dialogueText`**: 
  - Campo de texto onde as linhas de diálogo são exibidas.

- **`public string[] dialogueLines`**: 
  - Array contendo as linhas de diálogo que serão apresentadas ao jogador.

- **`private int currentLineIndex = 0`**: 
  - Índice atual da linha de diálogo que está sendo exibida.

- **`private bool playerNearby = false`**: 
  - Flag que indica se o jogador está próximo ao NPC.

- **`private bool dialogueActive = false`**: 
  - Flag para verificar se o diálogo está ativo ou não.

- **`public bool hasItem = false`**: 
  - Booleano para verificar se o jogador já recebeu um item.

- **`public Transform player`**: 
  - Referência ao transform do jogador para permitir que o NPC olhe para ele.

### Método `Start`

- **`dialogueUI.SetActive(false)`**: 
  - Inicialmente oculta o painel de diálogo para que ele não apareça até que o jogador interaja com o NPC.

### Método `Update`

- **`if (playerNearby && Input.GetKeyDown(KeyCode.E))`**:
  - Verifica se o jogador está perto e se a tecla "E" foi pressionada.
  - **`if (!dialogueActive)`**:
    - Se o diálogo não estiver ativo, exibe o painel e mostra a primeira linha de diálogo.
  - **`else`**:
    - Se o diálogo já estiver ativo, avança para a próxima linha ou encerra o diálogo se não houver mais linhas.
  - **`if (!hasItem)`**:
    - Se o jogador ainda não tiver o item, ele é concedido e uma mensagem é registrada no console.

- **`if (playerNearby)`**:
  - Se o jogador está perto, faz o NPC olhar para ele utilizando o método `LookAtPlayer`.

### Método `OnTriggerEnter`

- **`if (other.CompareTag("Player"))`**:
  - Quando o jogador entra na área do NPC, define `playerNearby` como verdadeira para indicar que o jogador está perto.

### Método `OnTriggerExit`

- **`if (other.CompareTag("Player"))`**:
  - Quando o jogador sai da área do NPC, define `playerNearby` como falsa e oculta o painel de diálogo, resetando o índice da linha e o estado do diálogo.

### Método `LookAtPlayer`

- **`Vector3 direction = player.position - transform.position`**:
  - Calcula a direção do jogador em relação ao NPC.

- **`direction.y = 0`**:
  - Define o componente Y da direção como 0 para evitar rotação vertical.

- **`Quaternion rotation = Quaternion.LookRotation(direction)`**:
  - Calcula a rotação necessária para que o NPC olhe na direção do jogador.

- **`transform.rotation = Quaternion.Slerp(transform.rotation, rotation, Time.deltaTime * 5f)`**:
  - Suaviza a rotação do NPC para que ele vire lentamente para olhar para o jogador.

## Explicação das Contas

### Movimento e Direção

- **`Vector3 direction = player.position - transform.position`**:
  - **`player.position`**: Posição atual do jogador.
  - **`transform.position`**: Posição atual do NPC.
  - **`direction`**: Vetor que aponta da posição do NPC para a posição do jogador.

  **Como funciona**: Calcula a direção do jogador para que o NPC possa girar e olhar diretamente para ele.

### Rotação Suavizada

- **`Quaternion.Slerp(transform.rotation, rotation, Time.deltaTime * 5f)`**:
  - **`transform.rotation`**: Rotação atual do NPC.
  - **`rotation`**: Rotação desejada para olhar para o jogador.
  - **`Time.deltaTime`**: Tempo desde o último frame, usado para suavizar a rotação.
  - **`5f`**: Fator de suavização, ajusta a velocidade da rotação.

  **Como funciona**: Suaviza a transição da rotação atual para a nova rotação, fazendo com que o NPC vire lentamente para olhar para o jogador.

## Explicação das Funções e Métodos

### `Input.GetKeyDown`

- **`Input.GetKeyDown(KeyCode.E)`**:
  - Retorna `true` se a tecla "E" for pressionada no frame atual.

  **Como funciona**: Detecta se o jogador pressionou a tecla "E" para iniciar ou avançar o diálogo.

### `OnTriggerEnter`

- **`OnTriggerEnter(Collider other)`**:
  - Método chamado quando outro colisor entra na área do colisor do NPC.

  **Como funciona**: Detecta a entrada do jogador na área do NPC para ativar a interação.

### `OnTriggerExit`

- **`OnTriggerExit(Collider other)`**:
  - Método chamado quando outro colisor sai da área do colisor do NPC.

  **Como funciona**: Detecta a saída do jogador da área do NPC para desativar o diálogo.

### `LookAtPlayer`

- **`LookAtPlayer()`**:
  - Método para fazer o NPC olhar para o jogador.

  **Como funciona**: Calcula a rotação necessária para o NPC olhar para o jogador e aplica uma rotação suavizada.
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Script do Fantasma

Este script controla o movimento de um fantasma em Unity, permitindo que ele se mova horizontalmente ou verticalmente dentro de limites definidos. Além disso, o script lida com a colisão com o jogador, reposicionando o jogador para a posição inicial ao colidir.

## Funcionalidades

- Movimento do fantasma em um eixo horizontal ou vertical
- Reposiciona o jogador para a posição inicial se o fantasma colidir com ele
- Mantém a posição Y do fantasma fixa durante o movimento

## Código

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Ghost : MonoBehaviour
{
    public enum MovementType
    {
        Horizontal, // Movimento da esquerda para a direita
        Vertical    // Movimento frente para trás
    }

    public MovementType movementType = MovementType.Horizontal;  // Tipo de movimento do fantasma
    public float moveSpeed = 5f;  // Velocidade de movimento do fantasma
    public float leftBoundary = -10f;  // Limite à esquerda
    public float rightBoundary = 10f;  // Limite à direita
    public float frontBoundary = -10f; // Limite frontal
    public float backBoundary = 10f;   // Limite traseiro
    public float fixedYPosition = 1f;  // Valor fixo do eixo Y do fantasma
    private bool movingForward = true;  // Controla a direção do movimento

    public Transform player;  // Referência ao jogador para reposicioná-lo
    public CharacterController playerController;  // Referência ao CharacterController do jogador (se houver)

    void Update()
    {
        // Mantém a posição Y fixa
        Vector3 currentPosition = transform.position;
        currentPosition.y = fixedYPosition;

        switch (movementType)
        {
            case MovementType.Horizontal:
                // Movimento do fantasma no eixo X
                if (movingForward)
                {
                    currentPosition.x += moveSpeed * Time.deltaTime;
                    if (currentPosition.x >= rightBoundary)
                    {
                        movingForward = false;  // Muda a direção para a esquerda
                    }
                }
                else
                {
                    currentPosition.x -= moveSpeed * Time.deltaTime;
                    if (currentPosition.x <= leftBoundary)
                    {
                        movingForward = true;  // Muda a direção para a direita
                    }
                }
                break;

            case MovementType.Vertical:
                // Movimento do fantasma no eixo Z
                if (movingForward)
                {
                    currentPosition.z += moveSpeed * Time.deltaTime;
                    if (currentPosition.z >= backBoundary)
                    {
                        movingForward = false;  // Muda a direção para trás
                    }
                }
                else
                {
                    currentPosition.z -= moveSpeed * Time.deltaTime;
                    if (currentPosition.z <= frontBoundary)
                    {
                        movingForward = true;  // Muda a direção para frente
                    }
                }
                break;
        }

        // Atualiza a posição do fantasma
        transform.position = currentPosition;
    }

    private void OnTriggerEnter(Collider other)
    {
        // Verifica se o fantasma colidiu com o jogador
        if (other.gameObject.CompareTag("Player"))
        {
            // Desabilita temporariamente o CharacterController, se existir, para evitar conflitos
            if (playerController != null)
            {
                playerController.enabled = false;
            }

            // Reposiciona o jogador na posição (0, 0, 0)
            player.position = Vector3.zero;

            // Reativa o CharacterController depois de reposicionar
            if (playerController != null)
            {
                playerController.enabled = true;
            }

            Debug.Log("Player foi reposicionado para a posição inicial!");
        }
    }
}
```
## Explicação do Código

### Declarações de Variáveis

- **`public enum MovementType`**: Define o tipo de movimento que o fantasma pode ter, podendo ser Horizontal ou Vertical.
- **`public MovementType movementType = MovementType.Horizontal`**: Define o tipo de movimento do fantasma, que por padrão é Horizontal.
- **`public float moveSpeed = 5f`**: Define a velocidade de movimento do fantasma.
- **`public float leftBoundary = -10f`**: Define o limite à esquerda do movimento horizontal do fantasma.
- **`public float rightBoundary = 10f`**: Define o limite à direita do movimento horizontal do fantasma.
- **`public float frontBoundary = -10f`**: Define o limite frontal do movimento vertical do fantasma.
- **`public float backBoundary = 10f`**: Define o limite traseiro do movimento vertical do fantasma.
- **`public float fixedYPosition = 1f`**: Define o valor fixo do eixo Y do fantasma, mantendo a altura constante.
- **`private bool movingForward = true`**: Controla a direção do movimento do fantasma (se está indo para frente ou para trás).
- **`public Transform player`**: Referência ao transform do jogador para reposicioná-lo se necessário.
- **`public CharacterController playerController`**: Referência ao `CharacterController` do jogador, se existir.

### Método `Update`

- **`Vector3 currentPosition = transform.position`**: Obtém a posição atual do fantasma.
- **`currentPosition.y = fixedYPosition`**: Mantém a posição Y do fantasma fixa.

- **`switch (movementType)`**: Verifica o tipo de movimento do fantasma (Horizontal ou Vertical).
  - **`case MovementType.Horizontal`**: Movimento do fantasma ao longo do eixo X.
    - **`if (movingForward)`**: Se o fantasma está se movendo para frente, aumenta a posição X.
    - **`if (currentPosition.x >= rightBoundary)`**: Se o fantasma alcança o limite direito, muda a direção para a esquerda.
    - **`else`**: Se o fantasma está se movendo para trás, diminui a posição X.
    - **`if (currentPosition.x <= leftBoundary)`**: Se o fantasma alcança o limite esquerdo, muda a direção para a direita.
  - **`case MovementType.Vertical`**: Movimento do fantasma ao longo do eixo Z.
    - **`if (movingForward)`**: Se o fantasma está se movendo para frente, aumenta a posição Z.
    - **`if (currentPosition.z >= backBoundary)`**: Se o fantasma alcança o limite traseiro, muda a direção para trás.
    - **`else`**: Se o fantasma está se movendo para trás, diminui a posição Z.
    - **`if (currentPosition.z <= frontBoundary)`**: Se o fantasma alcança o limite frontal, muda a direção para frente.

- **`transform.position = currentPosition`**: Atualiza a posição do fantasma com a nova posição calculada.

### Método `OnTriggerEnter`

- **`if (other.gameObject.CompareTag("Player"))`**: Verifica se o objeto com o qual o fantasma colidiu tem a tag "Player".
  - **`if (playerController != null)`**: Desabilita temporariamente o `CharacterController` do jogador, se existir, para evitar conflitos ao reposicionar o jogador.
  - **`player.position = Vector3.zero`**: Reposiciona o jogador para a posição inicial `(0, 0, 0)`.
  - **`if (playerController != null)`**: Reativa o `CharacterController` do jogador depois de reposicionar.
## Explicação das Contas

### Movimento do Fantasma

- **`currentPosition.x += moveSpeed * Time.deltaTime`**:
  - **`moveSpeed`**: Velocidade com que o fantasma se move. Define a distância que o fantasma percorrerá por segundo.
  - **`Time.deltaTime`**: Tempo decorrido desde o último frame. Garante que o movimento seja suave e independente da taxa de quadros.

  **Como funciona**: O movimento do fantasma é ajustado multiplicando a velocidade (`moveSpeed`) pelo tempo desde o último frame (`Time.deltaTime`). Isso assegura que o movimento seja consistente e suave, independentemente da taxa de quadros do jogo.

- **`currentPosition.x -= moveSpeed * Time.deltaTime`**:
  - **`moveSpeed`**: Velocidade com que o fantasma se move para trás.
  - **`Time.deltaTime`**: Tempo decorrido desde o último frame.

  **Como funciona**: Similar ao cálculo para movimento para frente, mas subtrai a velocidade multiplicada pelo tempo desde o último frame, movendo o fantasma para trás.

### Limites de Movimento

- **`if (currentPosition.x >= rightBoundary)`** e **`if (currentPosition.x <= leftBoundary)`**:
  - **`rightBoundary`** e **`leftBoundary`**: Limites que determinam até onde o fantasma pode se mover horizontalmente.

  **Como funciona**: Esses cálculos garantem que o fantasma pare de se mover ao atingir os limites definidos. Quando o fantasma atinge um limite, a direção do movimento é invertida para que ele não ultrapasse os limites estabelecidos.

- **`if (currentPosition.z >= backBoundary)`** e **`if (currentPosition.z <= frontBoundary)`**:
  - **`backBoundary`** e **`frontBoundary`**: Limites que determinam até onde o fantasma pode se mover verticalmente.

  **Como funciona**: Semelhante aos limites horizontais, esses cálculos garantem que o fantasma não ultrapasse os limites frontais e traseiros, invertendo a direção do movimento quando um limite é alcançado.

### Reposicionamento do Jogador

- **`player.position = Vector3.zero`**:
  - **`Vector3.zero`**: Representa a posição `(0, 0, 0)` no espaço 3D.

  **Como funciona**: Reposiciona o jogador para a origem do mundo (ponto `(0, 0, 0)`) quando ocorre uma colisão com o fantasma, garantindo que o jogador seja retornado a uma posição inicial conhecida.

### Controles de Movimento

- **`movingForward`**:
  - **`movingForward`**: Booleano que controla a direção do movimento.

  **Como funciona**: Esse controle determina se o fantasma está se movendo para frente ou para trás, trocando a direção quando o limite é alcançado.

## Explicação das Funções e Métodos

### `transform.position`

- **`transform.position`**: Obtém ou define a posição do GameObject no espaço 3D.
  - **Como funciona**: Permite acessar e modificar a posição do fantasma para movimentá-lo ou ajustá-lo conforme necessário.

### `Vector3`

- **`Vector3`**: Estrutura que representa um ponto no espaço 3D com coordenadas X, Y e Z.
  - **Como funciona**: Usada para definir e manipular posições e vetores de movimento no espaço 3D.

### `Time.deltaTime`

- **`Time.deltaTime`**: Tempo desde o último frame.
  - **Como funciona**: Garante que o movimento do fantasma seja suave e independente da taxa de quadros, ajustando a movimentação de acordo com o tempo decorrido.

### `OnTriggerEnter(Collider other)`

- **`OnTriggerEnter(Collider other)`**: Método chamado quando o GameObject colide com outro Collider com a configuração de trigger ativada.
  - **Como funciona**: Permite verificar e reagir a colisões com outros objetos, como o jogador, e executar ações específicas, como reposicionar o jogador.

### `CompareTag`

- **`CompareTag("Player")`**: Verifica se o objeto tem a tag especificada ("Player").
  - **Como funciona**: Facilita a identificação de objetos específicos com base em suas tags, permitindo que o script execute ações específicas para objetos com tags correspondentes.
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Script de Porta

Este script controla a abertura e o fechamento de uma porta com base na proximidade do jogador e na interação. A porta se move suavemente entre a posição fechada e a posição aberta com uma velocidade definida.

## Funcionalidades

- Movimentação da porta entre uma posição fechada e uma posição aberta
- Alternância do estado da porta (aberta ou fechada) com base na interação do jogador
- Verificação da proximidade do jogador para permitir a interação

## Código

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Door : MonoBehaviour
{
    public Vector3 openPositionOffset = new Vector3(2f, 0, 0);  // Offset da posição aberta da porta (direção e distância de movimento)
    public float openSpeed = 2f;       // Velocidade de abertura/fechamento
    public Transform player;           // Referência ao jogador
    public float interactionDistance = 3f; // Distância mínima para interação
    private bool isOpen = false;       // Se a porta está aberta ou fechada
    private bool isPlayerNear = false; // Se o jogador está perto da porta

    private Vector3 closedPosition;    // Posição original da porta (fechada)
    private Vector3 openPosition;      // Posição aberta da porta

    void Start()
    {
        // Armazena a posição original da porta como a posição fechada
        closedPosition = transform.position;
        // Calcula a posição aberta baseada no offset
        openPosition = closedPosition + openPositionOffset;
    }

    void Update()
    {
        // Verifica a distância entre o jogador e a porta
        float distanceToPlayer = Vector3.Distance(transform.position, player.position);

        // Se o jogador está dentro da distância de interação
        if (distanceToPlayer <= interactionDistance)
        {
            isPlayerNear = true;

            // Verifica se o jogador pressiona a tecla 'E'
            if (Input.GetKeyDown(KeyCode.E))
            {
                isOpen = !isOpen;  // Alterna o estado da porta (abre ou fecha)
            }
        }
        else
        {
            isPlayerNear = false;
        }

        // Move a porta suavemente para a posição aberta ou fechada
        if (isOpen)
        {
            transform.position = Vector3.Lerp(transform.position, openPosition, Time.deltaTime * openSpeed);
        }
        else
        {
            transform.position = Vector3.Lerp(transform.position, closedPosition, Time.deltaTime * openSpeed);
        }
    }
}
```
## Explicação do Código

### Declarações de Variáveis

- **`public Vector3 openPositionOffset = new Vector3(2f, 0, 0)`**: Define o deslocamento da posição aberta da porta, especificando a direção e a distância do movimento.

- **`public float openSpeed = 2f`**: Velocidade com que a porta se abre e fecha.

- **`public Transform player`**: Referência ao transform do jogador, usado para calcular a distância entre o jogador e a porta.

- **`public float interactionDistance = 3f`**: Distância mínima para interação com a porta.

- **`private bool isOpen = false`**: Indica se a porta está aberta ou fechada.

- **`private bool isPlayerNear = false`**: Indica se o jogador está perto da porta.

- **`private Vector3 closedPosition`**: Posição original da porta quando está fechada.

- **`private Vector3 openPosition`**: Posição para a qual a porta se move quando está aberta.

### Método `Start`

- **`closedPosition = transform.position`**: Armazena a posição original da porta como a posição fechada.

- **`openPosition = closedPosition + openPositionOffset`**: Calcula a posição para onde a porta se moverá quando estiver aberta, adicionando o deslocamento à posição fechada.

### Método `Update`

- **`float distanceToPlayer = Vector3.Distance(transform.position, player.position)`**: Calcula a distância entre a porta e o jogador.

- **`if (distanceToPlayer <= interactionDistance)`**: Verifica se o jogador está dentro da distância de interação.
  - **`isPlayerNear = true`**: Atualiza o estado para indicar que o jogador está perto da porta.
  - **`if (Input.GetKeyDown(KeyCode.E))`**: Verifica se a tecla 'E' foi pressionada para alternar o estado da porta.
    - **`isOpen = !isOpen`**: Alterna o estado da porta (abre se estiver fechada e fecha se estiver aberta).

- **`else`**: Se o jogador não está perto da porta.
  - **`isPlayerNear = false`**: Atualiza o estado para indicar que o jogador não está mais perto da porta.

- **`if (isOpen)`**: Se a porta está aberta.
  - **`transform.position = Vector3.Lerp(transform.position, openPosition, Time.deltaTime * openSpeed)`**: Move a porta suavemente para a posição aberta.

- **`else`**: Se a porta está fechada.
  - **`transform.position = Vector3.Lerp(transform.position, closedPosition, Time.deltaTime * openSpeed)`**: Move a porta suavemente para a posição fechada.

## Explicação das Contas

### Movimento da Porta

- **`transform.position = Vector3.Lerp(transform.position, openPosition, Time.deltaTime * openSpeed)`**:
  - **`transform.position`**: Posição atual da porta.
  - **`openPosition`**: Posição para a qual a porta deve se mover quando aberta.
  - **`Time.deltaTime`**: Tempo desde o último frame. Garante que o movimento seja suave e independente da taxa de quadros.
  - **`openSpeed`**: Velocidade com a qual a porta se move entre a posição fechada e aberta.

  **Como funciona**: A função `Vector3.Lerp` interpola a posição da porta entre a posição atual e a posição de destino (aberta ou fechada). O fator de interpolação é ajustado por `Time.deltaTime * openSpeed`, garantindo que a porta se mova suavemente e de maneira consistente com o tempo decorrido entre frames.

### Distância do Jogador

- **`float distanceToPlayer = Vector3.Distance(transform.position, player.position)`**:
  - **`transform.position`**: Posição atual da porta.
  - **`player.position`**: Posição atual do jogador.
  
  **Como funciona**: A função `Vector3.Distance` calcula a distância euclidiana entre a posição da porta e a posição do jogador. Este valor é usado para determinar se o jogador está dentro da distância de interação para que a porta possa ser aberta ou fechada.

### Distância e Interação

- **`if (distanceToPlayer <= interactionDistance)`**:
  - **`distanceToPlayer`**: Distância calculada entre a porta e o jogador.
  - **`interactionDistance`**: Distância mínima para que o jogador possa interagir com a porta.

  **Como funciona**: Verifica se a distância entre a porta e o jogador é menor ou igual à distância de interação definida. Se for, o jogador está perto o suficiente para interagir com a porta, e o estado da porta pode ser alternado.

## Explicação das Funções e Métodos

- **`Vector3.Distance`**:
  - **`Vector3.Distance(transform.position, player.position)`**: Mede a distância entre a posição da porta e a posição do jogador.

  **Como funciona**: Calcula a distância euclidiana para determinar se o jogador está perto o suficiente para interagir com a porta.

- **`Vector3.Lerp`**:
  - **`Vector3.Lerp(transform.position, openPosition, Time.deltaTime * openSpeed)`**: Move a porta suavemente entre a posição atual e a posição de destino.

  **Como funciona**: Interpola a posição da porta entre a posição fechada e a posição aberta, ajustando a suavidade do movimento com base na velocidade e no tempo.

- **`Input.GetKeyDown`**:
  - **`Input.GetKeyDown(KeyCode.E)`**: Detecta quando a tecla 'E' é pressionada para alternar o estado da porta.

  **Como funciona**: Alterna o estado da porta quando a tecla de interação é pressionada.

- **`if` e `else`**:
  - **`if (isOpen)`** e **`else`**: Decidem se a porta deve se mover para a posição aberta ou fechada com base no estado atual.

  **Como funciona**: Controla o movimento da porta de acordo com seu estado (aberta ou fechada).

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Script da Porta do Final de Jogo

Este script gerencia o teletransporte do jogador para uma nova cena quando ele está próximo de um ponto específico e possui um item necessário. Utiliza o componente `Collider` para detectar a proximidade do jogador e o componente `SceneManager` para carregar a nova cena.

## Funcionalidades

- Detecta quando o jogador está próximo de um ponto específico.
- Verifica se o jogador possui um item necessário.
- Teletransporta o jogador para uma nova cena quando a tecla "E" é pressionada.

## Código

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class Final : MonoBehaviour
{
    public string nomeCena;            // Nome da cena para teletransporte
    public Transform player;           // Referência ao jogador
    private bool playerProximo = false; // Verifica se o jogador está próximo

    // Referência ao script que contém o "hasItem"
    public Dialogue dialogue;

    void Update()
    {
        // Verifica se o jogador está próximo e se o jogador tem o item necessário
        if (playerProximo && dialogue.hasItem)
        {
            // Se o jogador apertar "E", e já tiver o item, teletransporta para a nova cena
            if (Input.GetKeyDown(KeyCode.E))
            {
                Teleportar();
            }
        }
    }

    // Função que realiza o teletransporte
    void Teleportar()
    {
        // Verifica se o nome da cena foi definido
        if (!string.IsNullOrEmpty(nomeCena))
        {
            SceneManager.LoadScene(nomeCena); // Carrega a nova cena
        }
        else
        {
            Debug.LogError("Nome da cena não foi definido na porta de teletransporte.");
        }
    }

    // Detecta quando o jogador entra no gatilho da porta
    private void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            playerProximo = true; // Marca que o jogador está próximo
            Debug.Log("Pressione 'E' para entrar na porta.");
        }
    }

    // Detecta quando o jogador sai do gatilho da porta
    private void OnTriggerExit(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            playerProximo = false; // Marca que o jogador saiu da proximidade
        }
    }
}
```
## Explicação do Código

### Declarações de Variáveis

- **`public string nomeCena`**: Nome da cena para teletransporte. Define qual cena será carregada quando o jogador interagir com o ponto de teletransporte.
- **`public Transform player`**: Referência ao transform do jogador. Usado para verificar a proximidade do jogador com o ponto de teletransporte.
- **`private bool playerProximo`**: Verifica se o jogador está próximo do ponto de teletransporte.
- **`public Dialogue dialogue`**: Referência ao script que contém a propriedade `hasItem`, que indica se o jogador possui o item necessário.

### Método `Update`

- **`if (playerProximo && dialogue.hasItem)`**: Verifica se o jogador está próximo e possui o item necessário.
  - **`if (Input.GetKeyDown(KeyCode.E))`**: Se a tecla "E" for pressionada e o jogador tiver o item, chama o método `Teleportar()` para carregar a nova cena.

### Método `Teleportar`

- **`if (!string.IsNullOrEmpty(nomeCena))`**: Verifica se o nome da cena foi definido.
  - **`SceneManager.LoadScene(nomeCena)`**: Carrega a cena especificada no campo `nomeCena`.
  - **`else`**: Se o nome da cena não foi definido, exibe um erro no console.

### Método `OnTriggerEnter`

- **`if (other.CompareTag("Player"))`**: Verifica se o objeto que entrou no gatilho tem a tag "Player".
  - **`playerProximo = true`**: Marca que o jogador está próximo do ponto de teletransporte.
  - **`Debug.Log("Pressione 'E' para entrar na porta.")`**: Exibe uma mensagem no console indicando que o jogador pode interagir com o ponto.

### Método `OnTriggerExit`

- **`if (other.CompareTag("Player"))`**: Verifica se o objeto que saiu do gatilho tem a tag "Player".
  - **`playerProximo = false`**: Marca que o jogador saiu da proximidade do ponto de teletransporte.

## Explicação das Contas

### Verificação de Proximidade

- **`if (playerProximo && dialogue.hasItem)`**:
  - **`playerProximo`**: Booleano que indica se o jogador está dentro da área de interação.
  - **`dialogue.hasItem`**: Booleano que indica se o jogador possui o item necessário.

  **Como funciona**: Esta condição garante que o teletransporte só será possível se o jogador estiver próximo e tiver o item necessário.

### Teletransporte para Nova Cena

- **`SceneManager.LoadScene(nomeCena)`**:
  - **`nomeCena`**: Nome da cena que será carregada.

  **Como funciona**: Carrega a cena especificada no campo `nomeCena`. Isso realiza a mudança de cenário no jogo, efetivamente teletransportando o jogador para a nova cena.

## Explicação das Funções e Métodos

### `Input.GetKeyDown`

- **`Input.GetKeyDown(KeyCode.E)`**: Retorna `true` se a tecla "E" for pressionada no frame atual.

  **Como funciona**: Detecta a entrada do jogador para acionar a interação com o ponto de teletransporte.

### `SceneManager.LoadScene`

- **`SceneManager.LoadScene(nomeCena)`**: Carrega a cena especificada pelo nome.

  **Como funciona**: Permite carregar uma nova cena quando o jogador interage com o ponto de teletransporte e possui o item necessário.

### `OnTriggerEnter`

- **Método para detectar quando um objeto entra na área de colisão**:
  - **`other.CompareTag("Player")`**: Verifica se o objeto que entrou tem a tag "Player".
  - **`playerProximo = true`**: Marca que o jogador está próximo e pode interagir com o ponto.

  **Como funciona**: Permite identificar quando o jogador está na área de interação e prepara para permitir o teletransporte.

### `OnTriggerExit`

- **Método para detectar quando um objeto sai da área de colisão**:
  - **`other.CompareTag("Player")`**: Verifica se o objeto que saiu tem a tag "Player".
  - **`playerProximo = false`**: Marca que o jogador saiu da área de interação e não pode mais interagir.

  **Como funciona**: Atualiza o estado para impedir o teletransporte quando o jogador sai da área de interação.
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
