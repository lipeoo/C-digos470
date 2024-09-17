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
