# Vr-FishCream

Hello! In our vr project we decided to make a chaotic comedy game. The game concept is simple. You are a ice cream seller and you hit the kids when the parents look away and in same time where is a pervert that tries to kidnap the kids. 

## Parent and Kids
Before I tell you my area I just want to mention that I wanted to have a challenge for myself, to no use update in the project. I really wanted to learn the event system and coroutines, it was challenging but fun. My area of responsibility was the family system, kids and parent. I wanted to make a object pool for the whole system. Instantiate kids, parent and empty family objects first, then systematicly put the kids and parents under empty family objetcs. 

When the family was ready to go, I made 3 point for them. Point A where they start to go from and move towards point B, which is the kiosk. Because I wanted to create a asyrncous system I used coroutines.Then I move those families to the kiosk staation with coroutines and let the members of the family do their things then leave and go to point C where they resets. 

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

After I instantiate objects I called this function with a coroutine to create new families. 

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

## Kids and parents
The behaviour of the kid was simple. At kiosk place he will go for his ice cream. When he get his order right, he wait for his sibling to be done and leave. 
I thought enum was the perfect system for this so I made a enum system with get and set. 


```
