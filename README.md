# Vr-FishCream

Hello! In our VR project, we've decided to create a chaotic comedy game. The concept is simple: you play as an ice cream seller who hits the kids when their parents look away, all while a pervert is trying to kidnap them.

I don’t have any screen recordings from VR, but here is some footage from Unity.

![ezgif-5-2078f447c6](https://github.com/user-attachments/assets/70aa4ed5-8c78-40ce-8221-2986457ac7c0)
![bild](https://github.com/user-attachments/assets/8240897d-3574-4725-b893-cf806e290599)



## Parent and Kids
Before I tell you about my area, I just want to mention that I wanted to challenge myself by not using the Update method in this project. I really wanted to learn the event system and coroutines—it was challenging but fun. My area of responsibility was the family system, including the kids and parents. I wanted to create an object pool for the whole system, instantiate the kids, parents, and empty family objects first, and then systematically assign the kids and parents to the empty family objects.

Once the family was ready, I set up three points for them: Point A, where they start, moving towards Point B, which is the kiosk. Since I wanted to create an asynchronous system, I used coroutines. I then moved those families to the kiosk station with coroutines, letting the members of the family do their tasks, then leave and go to Point C, where they reset.

Here you can see how I instantiate different objects

```csharp
    private void InstantiateFamilies()
    {
        InstantiateMembers(kidPrefab, kidList, kidName, kidTag, totalKids);
        InstantiateMembers(parentPrefab, parentList, parentName, parentTag, totalParent);

        for (int i = 0; i < totalParent; i++)
        {
            GameObject tmpFamilyObj = new GameObject();
            tmpFamilyObj.name = "Family" + i;
            tmpFamilyObj.AddComponent<BoxCollider>();
            NewFamilyMovement family = tmpFamilyObj.AddComponent<NewFamilyMovement>();
            family.direction = resetPos.position;
            family.gameObject.transform.position = startPos.position;
            family.OnHitB += OnIceCreamKiosk;
            familyList.Add(tmpFamilyObj);

            family.A = startPos;
            family.B = iceCreamPos;
            family.C = resetPos;
        }
    }

    public void InstantiateMembers(GameObject prefab, List<GameObject> listToHold, string prefabName, string prefabTag, int totalMember)
    {
        for (int i = 0; i < totalMember; i++)
        {
            GameObject member = Instantiate(prefab, startPos.position, transform.rotation, objectPool.transform);
            listToHold.Add(member);
            member.name = prefabName;
            member.tag = prefabTag;
            member.SetActive(false);
        }
    }
```
![bild](https://github.com/user-attachments/assets/3845eec7-41e4-4805-8575-519d05b802c9)

After instantiating the objects, I called this function using a coroutine to create new families.

```csharp
    public void CreateFamily()
    {
        int kids = UnityEngine.Random.Range(1, 3);

        GameObject tmpFamily = familyList[familyIndex];
        NewFamilyMovement newFamilyMovement = tmpFamily.GetComponent<NewFamilyMovement>();

        while (newFamilyMovement.IsInScene)
        {
            familyIndex++;
            if (familyIndex >= familyList.Count)
            {
                familyIndex = 0;
            }
            tmpFamily = familyList[familyIndex];
            newFamilyMovement = tmpFamily.GetComponent<NewFamilyMovement>();
        }

        if (familyIndex < totalParent)
        {
            newFamilyMovement.IsInScene = true;
            // family here
            newFamilyMovement.membersInList = new List<GameObject>();
            newFamilyMovement.objectIndex = familyIndex;
            newFamilyMovement.OnHitC += ResetFamilyManager;
            newFamilyMovement.speed = familyMovingSpeed;

            // parent here
            GameObject activatedParent = parentList[familyIndex];
            activatedParent.SetActive(true);
            activatedParent.transform.parent = tmpFamily.transform;
            newFamilyMovement.parentOfTheFamily = activatedParent;
            newFamilyMovement.parentTag = parentTag;

            // kid here

            newFamilyMovement.kidTag = kidTag;

            (GameObject parentType, ParentStates parentEnum) = parentRandomizer.GetParentGameObject();
            ParentTypeComponent parentTypeComponent = parentType.GetComponent<ParentTypeComponent>();

            // parent type here
            parentType.gameObject.transform.SetParent(activatedParent.transform);
            parentType.gameObject.SetActive(true);
            parentType.transform.position = activatedParent.transform.position;
            parentTypeComponent.parentStatesComp = parentEnum;
            Parent parentTmp = activatedParent.GetComponent<Parent>();
            parentTmp.endPosition = resetPos.position;

            
            newFamilyMovement.OnKidsAreDone += parentTmp.CallTheParent;
            newFamilyMovement.OnKidsAreDone += ResetAlVacansies;

            // here I make sure that all the other members follow the first member, which is the parent.
            Transform firstInLine = activatedParent.transform;


            parentList[familyIndex].transform.position = tmpFamily.transform.position;
            newFamilyMovement.membersInList.Add(activatedParent);

            for (int k = 0; k < kids; k++)
            {
                GameObject tmpKid = FindKid();
                Kid kid = tmpKid.GetComponent<Kid>();
                IceCreamOrder iceCreamOrder = tmpKid.GetComponent<IceCreamOrder>();
                tmpKid.SetActive(true);
                tmpKid.transform.SetParent(tmpFamily.transform);
                tmpKid.transform.position = tmpFamily.transform.position;
                kid.DoneKid += newFamilyMovement.KidIsDone;
                iceCreamOrder.OnOrderCmpleted += kid.CallOrderComplete;
                kid.OnIceCreamDone += newFamilyMovement.CheckIfGotIceCream;

                (GameObject kidType, KidSelect enumForType) = kidRandomizator.GetGameObject();
                KidTypeComponent kidTypeComponent = kidType.GetComponent<KidTypeComponent>();

                kidTypeComponent.typeKidSelect = enumForType;

                kidType.gameObject.transform.SetParent(tmpKid.transform);
                kidType.gameObject.SetActive(true);
                kidType.transform.position = tmpKid.transform.position;

                kid.myKidType = kidType;
                newFamilyMovement.membersInList.Add(tmpKid);
                newFamilyMovement.kidListToSend.Add(tmpKid);

                kid.endPositon = resetPos.position;
            }
            newFamilyMovement.FollowingInLineSystem();
            newFamilyMovement.familyStates = FamilyStates.GoToKiosk;
        }
    }

    public GameObject FindKid()
    {
        for (int i = 0; i < kidList.Count; i++)
        {
            if (!kidList[i].activeInHierarchy)
            {
                return kidList[i].gameObject;
            }
        }
        return null;

    }

```
![GIF 2024-10-16 23-11-01](https://github.com/user-attachments/assets/c05ceab1-dc78-4250-89e4-5621eb63eb22)
![ezgif-5-5c6ea06dba](https://github.com/user-attachments/assets/77f5eecd-c917-479b-8c7e-49cccea4a0b4)



## Kids and parents
The goal for the kid and parent was simple. At the kiosk, the kid would go get his ice cream. Once he got his order right, he would wait for his sibling to finish and then leave with the parent.


![ezgif-5-4d61519bed](https://github.com/user-attachments/assets/534b9ed6-c320-4c4b-bdc6-c82e22f44633)


I thought an enum was the perfect solution for this, so I created an enum system with getters and setters.

```csharp
public KidStates kidState
{
    get { return kidStates_; }

    set
    {
        if (kidStates_ != value)
        {
            kidStates_ = value;

            StopAllCoroutines();    

            NewFamilyMovement tmpFamily = GetComponentInParent<NewFamilyMovement>();
            switch (kidStates_)
            {
                case KidStates.defaultState:
                    animationController.IdleAnimation();
                    break;

                case KidStates.FollowingParent:
                    StartCoroutine(CallAnim(animationController => animationController.WalkAnimation()));
                    break;

                case KidStates.GoingToWaitingPos:
                    StartCoroutine(MoveAndRotateCoroutine());
                    FixRotationAndPositionForKidType();

                    break;

                case KidStates.WaitingIceCream:
                    order.GenerateOrder();
                    animationController.IdleAnimation();
                    FixRotationAndPositionForKidType();

                    break;

                case KidStates.GettingKidnaped:
                    KidIsInTheForest();
                    break;

                case KidStates.GotIceCream:
                    ReceiveOrder();
                    animationController.HappyAnimation();
                    OnIceCreamDone.Invoke();
                    pointHandler.ServeIceCream();
                    break;

                case KidStates.GoingToLeavingPos:
                    StartCoroutine(MoveToEnd(leavingPos));

                    break;

                case KidStates.InTheBag:
                    GetInToBag();

                    break;


                case KidStates.Leaving:
                    animationController.WalkAnimation();
                    DoneKid.Invoke();
                    order.ResetIceCreamOrder();

                    break;

            }
        }

    }
}
```
When a kid is satisfied, I get a boolean value that is true. In the FamilyMovement script, I check if the other kid is also true. If not, I don't do anything, but if both are true, I mark them as done.

```csharp
    public void KidIsDone()
{
    CheckIfAllKidsAreDone();
    if(allKidsAreDone)
    {
        OnKidsAreDone.Invoke();
        FollowingInLineSystem();
        iceCreamKiosk.isBusy = false;
        CheckIfFamilyIsDone();
        pointHandler.SatisfiedCompany();
    }
}

public void CheckIfAllKidsAreDone()
{
    List<Kid> memberlist = new List<Kid>();
    foreach (GameObject kids in membersInList)
    {
        if (kids.gameObject.CompareTag(kidTag))
        {
            Kid tmpKid = kids.GetComponent<Kid>();
            memberlist.Add(tmpKid);
        }
    }
    allKidsAreDone = memberlist.TrueForAll(kid => kid.kidIsDone);
}

```
## KidType and ParentType
My entire kid and parent system initially used cubes and spheres. When I first got the skeleton character prefabs, I thought I could simply swap the current prefabs with the new ones, but I quickly realized it wasn't a good idea. Instead, I decided to instantiate all kid characters into an object pool and, from there, pick up characters and assign them to the "kid" objects.

This was the first time I used dictionaries. After instantiating them and adding them to a list, I set them up using dictionaries. I randomized the characters and returned a character along with its enum. This was so I could use that data to reset it later.

 
```csharp
        public KidSelect GetKidRandom()
{
    int enumIntValue = System.Enum.GetValues(typeof(KidSelect)).Length;

    int randomIndex = UnityEngine.Random.Range(0, enumIntValue);
    return kidSelect = (KidSelect)randomIndex;
}

public (GameObject, KidSelect) GetGameObject()
{

    kidSelect = GetKidRandom();

    while (kidDictionary[kidSelect].Count == 0)
    {
        kidSelect = GetKidRandom();
    }
    GameObject kidType = kidDictionary[kidSelect][0];

    if (kidDictionary[kidSelect].Count > 0)
    {
        kidDictionary[kidSelect].RemoveAt(0);
    }

    return (kidType, kidSelect);
}
```
Thank you for reading. From this project, I learned about dictionaries, complex list systems, getters and setters, coroutines, and how to use booleans efficiently.
