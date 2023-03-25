# Portal Documentation

## Brief

Portals. To see the hidden space behind the wall, to teleport from one place to another. Try to make variations of interaction with the two portals.

[Process Video](https://youtu.be/N1xAZ7knLsE)

[MIRO board](https://miro.com/app/board/uXjVMeucDGQ=/)

## Process

### Teleports

#### physical transport

##### Prototype1

Teleport from 1 portal to another with updated location according to the other portal.

<img src="image/01.png" alt="01" style="zoom:50%;" />

<img src="image/02.png" alt="02" style="zoom:50%;" />

##### Prototype2(from Tutorial)

[Tutorial source](https://www.youtube.com/watch?v=PkGjYig8avo)

![11](image/11.png)

![12](image/12.png)

![13](image/13.png)

###### Basic understanding of scripts:

**Attached to Player camera**

`CameraMove.cs` is used to control the camera's position and rotation by Keys "AWSD" and the mouse.

`PortalCamera.cs` is used to render the view through two portals with relative movement of camera angles.

![0](image/0.jpg)

```c#
//rotate with camera, get mouse data
		var rotation = new Vector2(-Input.GetAxis("Mouse Y"), Input.GetAxis("Mouse X"));
		
// Move the camera, get key data
        float x = Input.GetAxis("Horizontal");//ad
        float z = Input.GetAxis("Vertical");//ws
        moveVector = new Vector3(x, 0.0f, z) * moveSpeed;
        moveY = Input.GetAxis("Elevation");//qe to move with height
//other undefined key
		Input.GetKey("G")
//check Input names in Project settings-Input Manager

```

![001](image/001.png)

`PortalPlacement.cs` is used for placing portals in a game environment. It requires a `CameraMove` component and a `PortalPair` object which contains the two portals that will be used in the game.

```c#
Input.GetButtonDown("Fire1")//Right mouse button
Input.GetButtonDown("Fire2")//Right mouse button
```

`Crosshair.cs` is a script for managing the crosshair in a portal game. It is responsible for displaying which portals have been placed and their colors.

`PlayerController.cs` defines a player controller class that inherits from a base class called `PortalableObject`.

**Attached to Portals:**

`PortalPair.cs` is attached to PortalPair group.

`Portal.cs` is attached to each portal under PortalPair group.

**Attached to Objects:**

`PortalableObject.cs` is used for object that can be teleported and transformed between two portals. 

### Vision in Portal(Shaders)

**camera**

![2](image/2.png)

### Variations of Interaction

#### portable object

The script: `PortalableObject.cs` is for `gameObject` to teleport from one portal to another portal, with relative position, rotation, velocity.

In the script `PortalableObject.cs`:

The `Awake()` method creates the `cloneObject` as a copy of the object being teleported, initializes its components with the same mesh and materials, and sets its scale to match that of the original object.

Get components of current object and make a new cloneObject.

```c#
 protected virtual void Awake()
    {
        cloneObject = new GameObject();
        cloneObject.SetActive(false);
        var meshFilter = cloneObject.AddComponent<MeshFilter>();
        var meshRenderer = cloneObject.AddComponent<MeshRenderer>();

        meshFilter.mesh = GetComponent<MeshFilter>().mesh;
        meshRenderer.materials = GetComponent<MeshRenderer>().materials;
        cloneObject.transform.localScale = transform.localScale;

        rigidbody = GetComponent<Rigidbody>();
        collider = GetComponent<Collider>();
    }
```

update the cloneobject's parameters with reverse data to simulate going out of the other portal.

```c#
private void LateUpdate()
    {
        if(inPortal == null || outPortal == null)
        {
            return;
        }

        if(cloneObject.activeSelf && inPortal.IsPlaced && outPortal.IsPlaced)
        {
            var inTransform = inPortal.transform;
            var outTransform = outPortal.transform;

            // Update position of clone.
            Vector3 relativePos = inTransform.InverseTransformPoint(transform.position);
            relativePos = halfTurn * relativePos;
            cloneObject.transform.position = outTransform.TransformPoint(relativePos);

            // Update rotation of clone.
            Quaternion relativeRot = Quaternion.Inverse(inTransform.rotation) * transform.rotation;
            relativeRot = halfTurn * relativeRot;
            cloneObject.transform.rotation = outTransform.rotation * relativeRot;

        }
        else
        {
            cloneObject.transform.position = new Vector3(-1000.0f, 1000.0f, -1000.0f);
        }
    }
```

give this object the new relative parameters.

```c#
public virtual void Warp()
    {
        var inTransform = inPortal.transform;
        var outTransform = outPortal.transform;

        // Update position of object.
        Vector3 relativePos = inTransform.InverseTransformPoint(transform.position);
        relativePos = halfTurn * relativePos;
        transform.position = outTransform.TransformPoint(relativePos);

        // Update rotation of object.
        Quaternion relativeRot = Quaternion.Inverse(inTransform.rotation) * transform.rotation;
        relativeRot = halfTurn * relativeRot;
        transform.rotation = outTransform.rotation * relativeRot;

        // Update velocity of rigidbody.
        Vector3 relativeVel = inTransform.InverseTransformDirection(rigidbody.velocity);
        relativeVel = halfTurn * relativeVel;
        rigidbody.velocity = outTransform.TransformDirection(relativeVel);
        
       
        // Swap portal references.
        var tmp = inPortal;
        inPortal = outPortal;
        outPortal = tmp;
    }
```

#### Entering Portals

The portal is not a physical hole, but a related camera view from another portal. So we have to disable the collider of the portal to let the object fall in it.

![5](image/50.png)

The `SetIsInPortal()` method is called when the object enters a portal and sets the `inPortal` and `outPortal` references. It also ignores collisions between the object and the portal's wall using the `Physics.IgnoreCollision()` method.

The `ExitPortal()` method is called when the object exits a portal and undoes the collision ignore setting. 

```c#
    public void SetIsInPortal(Portal inPortal, Portal outPortal, Collider wallCollider)
    {
        this.inPortal = inPortal;
        this.outPortal = outPortal;
//ignore collision of portal so it can fall in the portal
        Physics.IgnoreCollision(collider, wallCollider);
        cloneObject.SetActive(false);
        ++inPortalCount;
    }
    public void ExitPortal(Collider wallCollider)
    {
        Physics.IgnoreCollision(collider, wallCollider, false);
        --inPortalCount;
        if (inPortalCount == 0)
        {
            cloneObject.SetActive(false);
        }
    }

```

#### Variations of Entering Portals

In the script `PortalableObject.cs`:

The `LateUpdate()` method updates the position, rotation, and scale of the `cloneObject` to match that of the original object as it passes through the portal. It also changes the color of the clone object to a specified color.

The `Warp()` method is called when the object is teleported through the portal. It updates the position, rotation, velocity, scale, and color of the original object to match that of the clone object that passed through the portal. It also swaps the `inPortal` and `outPortal` references, so the object can be teleported back through the portal in the opposite direction.

Therefore, if I want to change the effect of the object after entering the portal, I should add relative variables and assign them to new effects under these two functions: `LateUpdate()`and `Warp()` .

##### Size Change

![41](image/41.png)

![42](image/42.png)

```c#
private float scaleMultiplier = 3.0f;
private void LateUpdate()
    {
        //...
            // Update size of clone.
            cloneObject.transform.localScale = transform.localScale * scaleMultiplier;
		//...
    }
```

```c#
public virtual void Warp()
    {
        var inTransform = inPortal.transform;
        var outTransform = outPortal.transform;

        // ...
    // Update size of object.
    transform.localScale = transform.localScale * scaleMultiplier;
    // ...
}
```

##### Color Change

```c#
private void LateUpdate()
    {
        // ...
            // Change color of clone.
        cloneObject.GetComponent<MeshRenderer>().material.color = changedColor;
        }
        // ...
     }
public virtual void Warp()
    {   // ...
    	// Change color of object.
        GetComponent<MeshRenderer>().material.color = changedColor;
    	// ...
	}
```

##### Shape Change

###### method1: create a new cube and destroy the previous sphere.

```c#
//FAILED: object destroyed forever
public virtual void Warp()
    {   
         // Instantiate a new cube object and set its position, rotation, and scale to match the original object.
        GameObject cubeObject = GameObject.CreatePrimitive(PrimitiveType.Cube);
        cubeObject.transform.position = transform.position;
        cubeObject.transform.rotation = transform.rotation;
        cubeObject.transform.localScale = transform.localScale;

        // Destroy the original sphere object.
        Destroy(gameObject);
        // ...
 	}
```

It changed to a cube when entering portal1 but not getting out of portal2. It destroyed the original object(sphere with a script) and stuck in the portal1, because the new cube object has no script attached to it.

![61](image/61.png)

![60](image/60.png)

###### method2: instead of destroy the object, only destroy the mesh, and assign a new mesh of cube to it.

unity:`PrimitiveType`

The various primitives that can be created using the [GameObject.CreatePrimitive](https://docs.unity3d.com/ScriptReference/GameObject.CreatePrimitive.html) function. However, it only has 6 basic primitives: sphere, capsule, cylinder, cube, plane, and quad.

```c#
 private void LateUpdate()
    {
    	// Create a new cube mesh and assign it to the clone object
     	Mesh cubeMesh = GameObject.CreatePrimitive(PrimitiveType.Cube).GetComponent<MeshFilter>().sharedMesh;
     	cloneObject.GetComponent<MeshFilter>().mesh = cubeMesh;
            // Update position of clone.
    }
```

```c#
public virtual void Warp()
    { 
    	// Destroy the previous mesh
        Destroy(cloneObject.GetComponent<MeshFilter>().mesh);
        // Create a new cube mesh and assign it to the clone object
        Mesh cubeMesh = GameObject.CreatePrimitive(PrimitiveType.Cube).GetComponent<MeshFilter>().sharedMesh;
        GetComponent<MeshFilter>().mesh = cubeMesh;
    }
```

###### method3: change to prefab/models

When throwing the portable object (sphere) into the portal, the portal will give back a hamburger.

```c#
// Instantiate the prefab.
GameObject newObject = Instantiate(prefabToSpawn, transform.position, transform.rotation);
newObject.transform.localScale = transform.localScale;
```

The hamburger will stuck in the surface of the portal.

![92](image/92.png)

So I made a slight displacement of the spawn hamburger.

```c#
    // Calculate the spawn position just outside the outPortal
    float portalExitOffset = -1.0f; // Adjust this value based on the size of the prefab
    Vector3 spawnPosition = outTransform.TransformPoint(relativePos) + outTransform.forward * portalExitOffset;
    // Instantiate the prefab.
    GameObject newObject = Instantiate(prefabToSpawn, spawnPosition, transform.rotation);
    newObject.transform.localScale = transform.localScale;
```

However, the generated hamburgers do not have the functions to teleport between two portals, so the hamburgers are stuck in the virtual space in the portal.

![91](image/91.png)

###### method4:change to organic shapes with data.(get portal location)

*Haven't finished yet.* When throwing the portable object (sphere) into the portal, the portal will give back a organic shape of object according to the portal data.

##### switch on/off environment light

Before entering: blue environment light

![93](image/93.png)

After entering: blue environment light shut down

![94](image/94.png)

```c#
//version1
//in LateUpdated()
// Check if the object has entered the portal an even number of times
        if (enterTimes % 2 == 0)
        {
            // Turn off the directional light
            GameObject.FindGameObjectWithTag("DirectionalLight").SetActive(false);
        }else
        {
            GameObject.FindGameObjectWithTag("DirectionalLight").SetActive(true);
        }
```

However, with this script, the light only turn off when the object enters in the portal for the first time. So I print the "enterTimes" to see if the time counter works.

```c#
Debug.Log(enterTimes);
```

![7](image/70.png)

The `enterTimes`works, but the switch on/off according to `enterTimes` doesn't work.

```c#
//version2
//turn light on/off when it is off/on
public void SetIsInPortal(Portal inPortal, Portal outPortal, Collider wallCollider)
{
    // Toggle the state of the directional light when entering the portal
        GameObject directionalLight = GameObject.FindGameObjectWithTag("DirectionalLight");
        bool currentLightState = directionalLight.activeSelf;
        directionalLight.SetActive(!currentLightState);
        Debug.Log(currentLightState);
}
//still only works for the first time
```

```c#
//version3 only turn off once
private void LateUpdate()
    {        
        if (hasEnteredPortal)
        {
        GameObject directionalLight = GameObject.FindGameObjectWithTag("DirectionalLight");
        bool currentLightState = directionalLight.activeSelf;
        directionalLight.SetActive(!currentLightState);
        hasEnteredPortal = false;     
        }        
    }
public void SetIsInPortal(Portal inPortal, Portal outPortal, Collider wallCollider)
    {
        //...
        // Set hasEnteredPortal to true when the object enters the portal
        hasEnteredPortal = true;
	}
```

#### User Friendly

```c#
[SerializeField] public float scaleMultiplier = 3.0f;
[SerializeField] public Color changedColor = Color.red;
[SerializeField] private PrimitiveType primitiveType = PrimitiveType.Cube;
[SerializeField] private GameObject prefabToSpawn; 
```

![80](image/80.png)

```c#
//turn on/off those variations
[SerializeField] private bool changeColor = false;
[SerializeField] private bool changeType = false;
[SerializeField] private bool createNewPrefab = false;
           
//in LateUpdated() and Warp()
		if(changeColor){
            // Change color of clone.
            GetComponent<MeshRenderer>().material.color = changedColor;
        }
        if(changeType){
            // Destroy the previous mesh
            Destroy(cloneObject.GetComponent<MeshFilter>().mesh);
            // Create a new cube mesh and assign it to the clone object
            Mesh cubeMesh = GameObject.CreatePrimitive(primitiveType).GetComponent<MeshFilter>().sharedMesh;
            GetComponent<MeshFilter>().mesh = cubeMesh;
        }
        if(createNewPrefab){
            // Instantiate the prefab.
            GameObject newObject = Instantiate(prefabToSpawn, transform.position, transform.rotation);
            newObject.transform.localScale = transform.localScale;
        }
```

![81](image/81.png)
