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

### Result:
Thus the game was developed using Unity and adopted _-----------AI technology.
