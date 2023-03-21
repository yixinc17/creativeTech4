# Portal Documentation

## Brief

Portals. To see the hidden space behind the wall, to teleport from one place to another. Try to make variations of interaction with the two portals.

[MIRO board](https://miro.com/app/board/uXjVMeucDGQ=/)

## Process

### Teleports

physical transport

Prototype1

Teleport from 1 portal to another with updated location according to the other portal.

<img src="image/01.png" alt="01" style="zoom:50%;" />

<img src="image/02.png" alt="02" style="zoom:50%;" />

Prototype2(from Tutorial)

[Tutorial source](https://www.youtube.com/watch?v=PkGjYig8avo)

![11](image/11.png)

![12](image/12.png)

![13](image/13.png)







### Vision in Portal(Shaders)

camera

![2](image/2.png)

### Variations of Interaction

#### portable object

The script: PortalableObject.cs is for gameobjects to teleport from one portal to another portal, with relative position, rotation, velocity.

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

![5](image/5.png)

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



#### Size Change

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
    // Update size of clone.
    transform.localScale = transform.localScale * scaleMultiplier;
    // ...
}
```

#### User Friendly
