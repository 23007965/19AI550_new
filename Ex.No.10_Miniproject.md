# Ex.No: 10  Implementation of 2D/3D game -------------------
### DATE:                                                                            
### REGISTER NUMBER : 
### AIM: 
To develop a game -------------------------in Unity 
### Algorithm:
```
1.
```  
### Program:
```
```
### Output:

#### Camera Follow
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CameraFollow : MonoBehaviour
{
    [SerializeField] Transform player;

    // Start is called before the first frame update
    void Start()
    {
    }

    // Update is called once per frame
    void Update()
    {
        transform.position = new Vector3(player.position.x, player.position.y, transform.position.z);
    }
}
```
#### Player Movement
```csharp
using UnityEngine;
using System.Collections;
using System.Collections.Generic;

public class PlayerMovement : MonoBehaviour
{
    [SerializeField] float jumpForce = 18f;
    [SerializeField] float runSpeed = 500f;
    float dirX;
    Rigidbody2D rb;
    BoxCollider2D collider2D;
    [SerializeField] LayerMask groundMask;
    SpriteRenderer spriteRenderer;
    Animator animator;
    bool gamePaused = false;
    private enum MovementState { idle, run, jump, fall }
    [SerializeField] AudioSource jumpAudio;


    void Start()
    {
        rb = GetComponent<Rigidbody2D>();
        collider2D = GetComponent<BoxCollider2D>();
        spriteRenderer = GetComponent<SpriteRenderer>();
        animator = GetComponent<Animator>();
    }

    void Update()
    {
        dirX = Input.GetAxisRaw("Horizontal");

        if (Input.GetKeyDown(KeyCode.Space) && isGrounded())
        {
            rb.linearVelocity = new Vector2(rb.linearVelocity.x, jumpForce);
            jumpAudio.Play();
        }
        HandleAnimation();
        PauseGame();
    }
    void PauseGame()
    {
        if (Input.GetKeyDown(KeyCode.Escape))
        {
            gamePaused = !gamePaused;
        }

        if (gamePaused)
        {
            Time.timeScale = 0f;
            AudioListener.pause = true;
        }
        else
        {
            Time.timeScale = 1f;
            AudioListener.pause = false;
        }
    }


    private void FixedUpdate()
    {
        rb.linearVelocity = new Vector2(dirX * runSpeed * Time.deltaTime, rb.linearVelocity.y);
    }

    bool isGrounded()
    {
        return Physics2D.BoxCast(collider2D.bounds.center, collider2D.bounds.size, 0, Vector2.down, 0.1f, groundMask);
    }
    void HandleAnimation()
    {
        MovementState state;

        if (dirX > 0)
        {
            spriteRenderer.flipX = false;
            state = MovementState.run;
        }
        else if (dirX < 0)
        {
            spriteRenderer.flipX = true;
            state = MovementState.run;
        }
        else
        {
            state = MovementState.idle;
        }

        if (rb.linearVelocity.y > 0.1f)
        {
            state = MovementState.jump;
        }
        else if (rb.linearVelocity.y < -0.1f)
        {
            state = MovementState.fall;
        }

        animator.SetInteger("state",(int)state);
    }

}

```
### Result:
Thus the game was developed using Unity and adopted _-----------AI technology.
