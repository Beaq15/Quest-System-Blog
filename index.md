Create your first c++ project in unreal engine

Introduction

This blog post will be about helping you guys learn from my mistakes when creating an Unreal Engine project in C++ for the first time. 

Who am I?

Hello everyone! My name is Beatrice and I am currently a Y2 student at Breda University of Applied Sciences, following the Creative Media and Game Technologies, Programming path. I decided to create a quest system in this engine because i also have some previous experience with it and thought it wouldn't be too hard. Well, i was wrong. There were different simple tasks that took me days to solve, but don't get discouraged, you eventually get the hang of it. Without further ado, lets get to work.

Creating your project

I decided to use the first person template provided by the engine so that i wouldn't have to worry about implementing features that aren't related to a quest system. I knew that I was going to add a kill type of quest and the third person template doesn't have a gun already implemented. Also, don't forget to choose C++ as project default.

Creating our data table

I used a data table to store the details for my quests, like name, description or to check if it is a main or a side quest. Each quest contains a number of stages with a name and a description and each stage contains however many objectives you want with a name, description, their type which can be (interact, location, kill or collect) their objectiveID and quantity. You can design this however you like, this is just the way i did it. 
A data table you create my right click ing in the contect browser and going to miscellaneous. It will ask for a row structure. For that, in any class that you want, you create it like this : 
struct FQuestDetails : public FTableRowBase. 

Make use of the data table

In the function you want to make use of your data table, declare a variable of type DataTableRowHandle QuestData; and you don't need to initialize the data table or its row name in the constructor or anyone in the code. To the quest data, you add the property of UPROPERTY(EditInstanceOnly, BlueprintReadWrite) and this allows you to set the values in the editor. In order to make use of those variables, all you need to do is:
```cpp
 if (QuestData.DataTable != nullptr && !QuestData.RowName.IsNone())
 {
     FString ContextString(TEXT("Quest Context"));
     FQuestDetails* DataRow = QuestData.DataTable->FindRow<FQuestDetails>("NewRow", ContextString);
     if (DataRow != nullptr)
 }
```
 Make sure that you write the name of the row exactly as you defined it in the data table otherwise this will not work.

 Creating an interface

 Interfaces can be used for different things. This is usually when there are more than 2 classes of different types that need to do the same thing. So, i created an interface class and i can immediately see it creates 2 classes. With the UInterface we don't have to do anything, while in the other one i can add my functions. I should not declare any variables in here or create a definition for the functions, i personally just declare them abstract : virtual FString InteractWith() = 0; I have to mention that in the cpp file, you also need to create the definition of the constructor of the interface, without declaring it in the header file:
 ```cpp
 UInteractionInterface::UInteractionInterface(const class FObjectInitializer& ObjectInitializer)
	: Super(ObjectInitializer) {}
```
 I made my npc and quest giver inherit from the interface by including the header file:
  ```cpp
 class QUESTSYSTEMFPP_API ANPC : public ACharacter, public IInteractionInterface
```
In my player class, i use LineTraceSingleByChannel to see what actor is in front of the it. If it the hit actor implements the interface, than i do whatever i need. 

 ```cpp
FVector Start = GetActorLocation();
FVector End = Start + (GetActorForwardVector() * 200);

FHitResult HitResult;
FCollisionQueryParams CollisionParams;
CollisionParams.AddIgnoredActor(this);

bool bHit = GetWorld()->LineTraceSingleByChannel(
	HitResult,
	Start,
	End,
	ECC_Camera,
	CollisionParams
);

if (bHit)
{
	AActor* HitActor = HitResult.GetActor();
	if (HitActor && HitActor->GetClass()->ImplementsInterface(UInteractionInterface::StaticClass()))
	{
		LookAtActor = HitActor;
		IInteractionInterface* Interface = Cast<IInteractionInterface>(LookAtActor);
    }
}
```

Creating a user widget class

First of all, the design of the UI is done in the editor. Then, in the header file, every variable needs to be declared with the property UPROPERTY(meta = (BindWidget)) and named exactly as in the editor. You then need the constructor where you can assign the text or buttons which should be declared under protected like so protected: virtual void NativeConstruct() override; When creating a widget, besides the pointer to the class, you need to also declare a subclass of it: 
 ```cpp
class UQuestLogEntry_Objective* QuestLogEntryObjectiveWidget;

UPROPERTY(EditAnywhere, BlueprintReadWrite)
TSubclassOf<UQuestLogEntry_Objective> QuestLogEntryObjectiveWidgetClass;
```
And then create it like this in the cpp file:
 ```cpp
QuestLogEntryObjectiveWidget = CreateWidget<UQuestLogEntry_Objective>(GetWorld(), QuestLogEntryObjectiveWidgetClass);
QuestLogEntryObjectiveWidget->AddToViewport(0);
```

When in the editor, don't forget to go to the graph of the class and then in class settings set the parent of the class and in the class defaults set subclass.

If you want to also add widgets that are animated, you can use the animations window in the designer tab of the widget and once you're done, declare it like this:
 ```cpp
UPROPERTY(Transient, meta = (BindWidgetAnim))
UWidgetAnimation* Appear = nullptr;
```

Don't forget to name it exactly the same as in the editor and to check if it exists before playing it so that your program doesn't crash:
 ```cpp
if (Appear)
{
	PlayAnimation(Appear);
}
```
