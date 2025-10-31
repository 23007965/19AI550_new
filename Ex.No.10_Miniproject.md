# Ex.No: 10  Implementation of 2D/3D  platform runner game 
### DATE:  3.11.2025                                                                          
### REGISTER NUMBER : 212223230145
### AIM: 

To develop a 2D platform runner game in Unity using basic game components and AI-enhanced scripting.

### Algorithm:

#### 1.	Setup the Scene
o	Open Unity Hub and create a new 2D project.

o	Name the project (for example: "PlatformRunnerGame").
#### 2.	Import Assets from Store
o	Go to the Unity Asset Store and import pixel-art backgrounds, tilesets, characters, items, traps, and UI elements needed for your game.
#### 3.	Create Main Screens and Levels
o	In the "Scenes" folder, create the following scenes: MainMenu, Leve1, Leve2, EndScreen.

o	For each scene, design the level layout using imported assets.
#### 4.	Player Character Creation
o	In each level’s Hierarchy, add your main player sprite.

o	Set the starting position for the player (e.g., at the far left platform).
#### 5.	Add Collectible Items
o	In your hierarchy, add collectible items (such as melons) at various strategic locations in the level.

o	Make sure each collectible has a collider and is set up to interact with the player.
#### 6.	Add Hazards and Moving Traps
o	Place traps such as saws or spikes in the scene using sprites from the asset store.

o	Attach movement scripts (e.g., for saws/platforms) as needed for dynamic hazards.
#### 7.	Create Level Completion and UI
o	Place a finish flag or endpoint at the end of each level.

o	Design the UI to display the collectible count and player status.

o	On MainMenu, add a "PLAY" button. On EndScreen, add "QUIT" and show a completion message.
#### 8.	Apply Scripts
o	Create scripts for PlayerMovement, CameraFollow, Collector (for collectible logic), GameManager (for switching scenes), PlayerHealth (for resets when hit by trap), and Finish (for level completion).

o	Attach relevant scripts to objects (player, hazards, collectibles, flag) in the Inspector.
#### 9.	Connect Scene Transitions
o	Configure GameManager or similar script so that:

o	Clicking "PLAY" loads Leve1.

o	Reaching the flag loads the next level (Leve2).

o	Completing Leve2 loads EndScreen and activates the completion UI.
#### 10. Add Audio Effects and Background Music
o Import audio resources for the background, player jump, item collection, and finish line.

o Set up background music with looping, start on awake.

o Play jump sound in PlayerMovement script when jump triggered.

o Play collectible sound on item pickup event.

o Play finish sound in level complete/flag script.

#### 11.	Test the Game
o	Click Play in Unity Editor.

o	Collect items and avoid traps; ensure hitting a trap or falling resets the player to the level start.

o	Confirm collectible counter UI updates, menu transitions, and level completions work as intended.
#### 12.	Stop the Game Play
o	After testing, stop the game in Unity Editor.



### Program:
#### Camera Follow.cs
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
#### Player Movement.cs
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
#### Player Health.cs
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class PlayerHealth : MonoBehaviour
{
    Animator animator;
    [SerializeField] AudioSource deathAudio;
    private void Start() {
        animator = GetComponent<Animator>();
        deathAudio = GetComponent<AudioSource>();
    }

    private void OnCollisionEnter2D(Collision2D other) {
        if (other.transform.tag == "trap")
        {
            animator.SetTrigger("death");
            GetComponent<PlayerMovement>().enabled = false;
            deathAudio.Play();

        }
    }
    void Update()
    {
        if (transform.position.y < -10)  // Set below platform level as threshold
        {
            Debug.Log("Player fell! Restarting level...");
            RestartLevel();
        }
    }

    public void RestartLevel()
    {
        transform.SetParent(null);
        SceneManager.LoadScene(SceneManager.GetActiveScene().name);
    }
}
```
#### Game Manager.cs
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class GameManager : MonoBehaviour
{
    public void PlayGame()
    {
        SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex + 1);
    }

    public void QuitGame()
    {
        Application.Quit();
    }
}

```
#### Collector.cs

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class Collector : MonoBehaviour
{
    [SerializeField] private Text countText;
    [SerializeField] private AudioSource collectAudio;
    private int countPine = 0;

    private void OnTriggerEnter2D(Collider2D other)
    {
        if (other.gameObject.tag == "fruit")
        {
            if (collectAudio != null)
                collectAudio.Play();
            countPine++;
            if (countText != null)
                countText.text = "PINE : " + countPine;
            Destroy(other.gameObject);
        }
    }
}

```
#### Finish.cs
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class Finish : MonoBehaviour
{
    [SerializeField] AudioSource winAudio;
    private void OnTriggerEnter2D(Collider2D other)
    {
        if(other.gameObject.tag == "Player")
        {
            Invoke("NextLevel", 1.25f);
            winAudio.Play();
        }
    }

    void NextLevel()
    {
        SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex + 1);
    }
}

```
### Output:

<img width="1097" height="523" alt="image" src="https://github.com/user-attachments/assets/29283d4d-1bfd-40ac-9573-518f80ac42dc" />

<img width="1102" height="504" alt="image" src="https://github.com/user-attachments/assets/c356bc5b-e5dc-48ef-a470-a61652c9c821" />

<img width="1095" height="500" alt="image" src="https://github.com/user-attachments/assets/5e7e9070-b485-43f6-a39e-84bd3e0f05ea" />

<img width="1088" height="528" alt="image" src="https://github.com/user-attachments/assets/7f2bb11a-1bf6-4728-bd14-46649ec54592" />

### Result:

Thus, the 2D platform runner game was successfully developed using Unity and adopted AI technology for features like collectible tracking, hazard avoidance logic, and dynamic scene transitions to improve player experience and engagement.
