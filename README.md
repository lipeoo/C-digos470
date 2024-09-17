# Controle de Movimento do Personagem

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

### Método Start

- Inicializa os componentes character e animator com os componentes do GameObject.
- Define jump como Vector3.zero.

### Método Update

- Obtém os inputs do jogador para movimento horizontal e vertical.
- Calcula a direção do movimento com base na orientação da câmera.
- Move o personagem na direção calculada.
- Atualiza o estado da animação com base nas ações do jogador.
- Gerencia o pulo e a aplicação de gravidade.
- Teleporta o personagem para a posição inicial se ele cair fora do mapa.

### Método ForaDoMapa

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
