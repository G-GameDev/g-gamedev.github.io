---
title: "Enums And Bitflags"
thumbnail: "/assets/images/thumbnails/Thumbnail-Enums.png"
synopsis: "How to use enums and bitflags in Unreal game development."

toc: true
toc_label: "Content"
toc_icon: "table"  # corresponding Font Awesome icon name without the "fa" prefix
toc_sticky: true   # enables sticky toc

layout: single
classes:
  - wide_post
---

# Introduction
The contents of this page are derived (mostly) from the YouTube video
'How to use enums and bitflags in Unreal game development'.<br>
It goes into what enums are, how they operate,
what they are useful for, and when to use them (or use something else.)<br>
As usual there will be examples with both blueprints and C++ where appropriate.
{: .notice--info}

# YouTube video:
{% include video id="TuHFeS_eBe8" provider="youtube" %}

# What are Enums?
Enums in Unreal (and for that matter in most development languages or platforms) are a way of grouping a bunch of
possible choices together under one ‘thing’. 
That ‘thing is usually considered to be a `type`, in a similar way that a class you define is a type, 
and that `int`, and `bool` are types provided by the language or platform.
For example, you can define an enum called `BasicFruit`, with entries for Apple, Banana, and Tomato.
If you then declare a variable of type `BasicFruit`, you are saying that only one of those 3 things is a valid ‘value’ for
that variable.

We might be making a game with different kinds of enemies, and create an enum declaring them like so:

~~~cpp
UENUM(BlueprintType)
enum class EDoorPlacementFlags : uint8
{
    Grunt,
    Brawler,
    BigCheese
}
~~~

This would allow you to have an AActor class for your enemies, with a field for the type of enemy they were,
~~~cpp
EEnemyType EnemyType = EEnemyType::Grunt;
~~~
Which would then allow you to do all sorts of things, depending on what specific type of enemy they are, for example, we
might call a different function when the player defeats the enemy, that drops different kinds of loot depending on the
type of enemy.

~~~cpp
void CharacterDied()
{
    switch (EnemyType)
    {
            case EEnemyType::Grunt:
            {
                DropBasicLoot();
                Break;
            }
    
            case EEnemyType::Brawler:
            {
                DropMediumLoot();
                Break;
            }
    
            case EEnemyType::BigCheese:
            {
                DropLuxuryLoot();
                Break;
            }
    }
}
~~~

And we can also use this in any ‘construction’ type functions, 
to give the enemy a different character mesh, or different equipment based on the selected type, 
and calculate different amounts of health, or set different amounts of damage that they can inflict 
to the player when they hit them.

If we take the declaration we made in C++ and decorate it with the appropriate UPROPERTY() specifiers, 
like so:
~~~cpp
UPROPERTY(BlueprintReadOnly, EditAnywhere)
EEnemyType EnemyType = EEnemyType::Grunt;
~~~
We now have an Actor that level designers can simply drop in the level, 
and use a dropdown list to select the specific enemy type. 
(In this case Grunt is the default value.)

Obviously for more complicated enemies with greater differences between them we wouldn’t use 
this approach, we would more than likely create a series of inherited classes. 
But for simpler scenarios this is fine.
{: .notice--warning}

You might use it for something like a ‘treasure chest’ actor; 
where the level designer selects the ‘riches tier’ with a dropdown, 
and the code sets the appropriate loot content, mesh, and even a different tune that plays when the player cracks it open.

## Enums are based on Integers. 
When you create an unreal enum in C++, which can be used by blueprints, you
will (currently) declare that it is based on `uint8`.

Each entry you provide is assigned a numerical value, either by you directly, 
or automatically by your code compiler in C++, or unreal, if you make these things in the editor.
By the way, an unsigned 8-bit integer, or `uint8`, can store 256 different ‘values’ (0 to 255), 
so blueprint visible enums cannot have more entries than that. 

I would make a strong argument that if you are getting even 50 distinct enum values,
you might want to think about using a database and having a table instead of that enum, 
or at the very least leveraging something like unreal’s datatables. 
You will have more flexibility, and amending things might well be easier. 

There are obviously situations where that is NOT an option, 
for example if you are developing a plugin which needs to work with a minimum of additional assets, 
and obviously in the case of the Unreal Engine itself, which has some pretty massive bitflag enumerations, 
but we’ll get to those later.

So the fact enums are really just numbers is useful to know, for example, when we are using databases, 
we can store that value in a standard integer field in a table, 
and likewise read it back in and cast it to the enum.
We can also do other things if we have knowledge of the underlying number.

Consider the previous example, the game with the various kinds of enemies.
If we manually number them ourselves, it might look like this:
~~~cpp
UENUM(BlueprintType)
enum class EDoorPlacementFlags : uint8
{
    Grunt       = 1,
    Brawler     = 2,
    BigCheese   = 3
}
~~~

If we don’t manually number them, then the compiler will do it for us, in the order we declare them.
As long as they are in an order which makes sense, we can do things like:
~~~cpp
if( EnemyType >= EEnemyTypes::Brawler)
{
    Armour += 50; /* Bonus armour for tougher enemies. */
}
~~~

So it makes sense to construct your enums with the entries ordered in a meaningful way (if appropriate).

And you can even be sneaky and use them to store actual data, which is the unique value, 
as long as they are all unique integer values (between 0 and 255).

For example, you might say `Grunt = 10, Brawler = 20, BigCheese = 50`.
You could then just use the integer value of the assigned enum as the base health value, 
or multiply it by something else, or...whatever, you get the idea, 
you can be creative with such things if your situation fits the data.

## Zero value convention
By convention, the value 0 (zero) is generally used to define a `‘none’, ‘nothing’, or ‘empty’` type value. 
But you are not required to do that – not all things you might want to make enums of have a sensible ‘none’ type entry, 
and you might decide that in all places where you use that enum you will always declare a default value from it.

## Unreal specifiers for enums
'Specifiers' are values used in those C++ macros (`UCLASS`, `USTRUCT`, `UPROPERTY`, etc.)<br>
If you are making `blueprintable` enums in C++, you can decorate the individual enum entries with with specifiers to do a few useful things.
You can use `UMETA(DisplayName)` to control how the items are displayed in dropdown lists in the unreal editor.<br>
For example:
~~~cpp
IsNotEqual UMETA(DisplayName = "!=")
~~~

Allows us to have the != symbols familiar to C++ programmers displayed, 
instead of the text ‘IsNotEqual’.<br>
And if you want to give further information to editor users, 
you can apply a `UMETA(ToolTip)` specifier, and provide specific text to be displayed for each entry, 
or just selected ones:
~~~cpp
UENUM(BlueprintType)
enum class EMenuItemTypes : uint8
{
	Undefined,
	QuitItem		 UMETA(ToolTip = "A button style item that closes the game."),
	ActionItem		 UMETA(ToolTip = "A button style item that calls a function when clicked."),
	TransitionItem	 UMETA(ToolTip = "Transitions the camera to another World Menu"),
	SliderItem		 UMETA(ToolTip = "Selects a numeric value using a slider control"),
	ToggleItem		 UMETA(ToolTip = "Selects a boolean value using a toggle control"),
	SpinnerItem		 UMETA(ToolTip = "Selects a numeric value using plus/minus buttons"),
	TextViewItem	 UMETA(ToolTip = "Displays a text value"),
	TextEditItem	 UMETA(ToolTip = "Edits a text value"),
	SaveSlotListItem UMETA(ToolTip = "Displays a list of save game slots"),
	DividerItem		 UMETA(ToolTip = "Displays a line or other seperator between items"),
	CustomWidgetItem UMETA(ToolTip = "A user provided widget type")
};
~~~

Another useful one is the `UMETA(Hidden)` specifier, 
which allows the C++ programmer to define an enum which can be used by the unreal editor, 
but which will not display certain entries to the user.<br>
For example, you might have an ‘Unassigned’, or ‘None’ value which you decide is perfectly valid in your C++ code, 
but which you don’t want to allow level designers to select for placed actors, so you hide it.

For fairly obvious reasons, there is no option to hide entries if you define an enum in the unreal editor.
You would be hiding it from basically everything.<br>
Yes, I’m sure there are some blueprinters who would love to have a feature that let them hide things from the C++ developers, 
but that wouldn’t actually be useful :grin:
{: .notice--info}

## Quick Recap
An enum is basically a collection of human readable names, with associated integer values.<br>
It gives us a way of writing code that talks in terms of (hopefully) understandable words, 
instead of just dealing with numbers, which you might easily forget the meaning of.
And it also gives us the benefit of a certain amount of checking, 
that the values we provide are actually values that are legitimate for the variable we are assigning them to.

# BitFlags
Bitflags are basically an extension of enums, 
in-as-much as they are just enums which are declared and used in a very specific way.
Because we are trying to include Blueprints in this conversation as much as possible, 
we will be primarily considering bitflag enums based on that `uint8` size we mentioned earlier.
The unreal engine uses some much larger ones, based on `uint64`, but they cannot be used directly in the Unreal editor, or
blueprints.

## Binary
So what are bitflag enums, and how they actually work?
To explain that, we need to talk about binary.

Oi you!<br>
Yes you, the blueprint programmer about to skip because talk of binary is obviously C++ stuff, 
and doesn’t relate to you.<br>
It absolutely does relate to you.<br>
Firstly, because at some point you might start including some C++ in your repertoire, 
and secondly because the principle of bitflags is the same in most languages, in fact, 
what I'm about to describe is pretty much the same thing if you are coding for unity in C#, 
or making your enum in the unreal editor.
{: .notice--danger}

So, binary.<br>
Many of you will already be happy with what it is and how it works.<br>
Some of you will be vaguely aware that it’s a bunch of noughts and ones and little more than that.<br>
And some of you will be of the opinion that you don’t need to know anything about it, 
because modern computers and software can hide it from you so you don’t need to know.<br>
Well, it's true; you can be a modern programmer without understanding binary – but you will be a better one if you do.<br>
Not only that, but certain things will start making more sense to you, 
and you will begin to recognise and understand certain common things which you encounter in your projects – 
the standard texture sizes used in 3D graphics for example.

## A Byte
A byte is often considered the smallest unit of computer memory.<br>
You might like to think of it in physics terms;<br>
It’s a little bit like an atom. We know that atoms are made up smaller sub atomic particles - which in this metaphor
represent the binary zeros and ones, but the atom itself is kinda the smallest thing which is useful on its own.

Computer memory is generally organised and addressed in bytes (or multiples of bytes)<br>
Each byte is made of eight bits, those individual zeros or ones.<br>
And that `uint8` we keep talking about? Is just a byte.

## Counting 'normally', in decimal
When we count ‘normally’ in what is called decimal, we start with the units, from 0 to 10, and after that we add another column, 
each digit in that is 10 times more than in the units column, and then the next column is a hundred times more, etc.
Each successive column is 10 times more than the previous column.

## Counting in binary
The columns in binary increase in powers of 2.

What that means, is the first column is a value of either 0 or 1,<br>
the next column is either a value of 0 or 2,<br>
the next column 0 or 4,<br>
then 0 or 8<br>
0 or 16,<br>
0 or 32,<br>
0 or 64,<br>
and finally for the byte, 0 or 128

By specifying a value of 0 or 1 for each of those columns, we can create any value between 0 and 255.
Counting from 1 to 8 in binary (shown as an entire byte with leading zeros) is as follows:

1. 00000001
2. 00000010
3. 00000011
4. 00000100
5. 00000101
6. 00000110
7. 00000111
8. 00001000

Another example:<br>
Binary number 00101010 =<br>
128 * 0 + 64 * 0 + 32 * 1 + 16 * 0 + 8 * 1 + 4 * 0+ 2 * 1 =<br>
42

## So how does any of this apply to bitflags?
Ok, so think of those 8 zeros or ones as 8 individual booleans.<br>
If we assign a different meaning to each of those booleans we can have something useful.

The way the bitflags enum works, is that each distinct thing is assigned a value of one of those binary digits we
mentioned earlier.<br>
So for a byte, you can have entries with individual values of:
* 1
* 2
* 4
* 8
* 16
* 32
* 64
* 128 

One for each bit in the byte.<br>
So, yeah, seems like we went from having 256 enum entries, suddenly down to only 8.<br>
What gives?<br>
Well before, we were just interested in which `ONE` of a selection of entries was the current value.
But now we are saying; which `COMBINATION` of those values do we have.

You can use this for things like common buffs and de-buffs.<br>
You might make a tile based game, and have an enum to represent which landscape features are present on each tile:

| geography type | enum value |
| :------------- | :--------- |
| grassland | 1 |
| hills | 2 |
| rivers | 4 |
| roads | 8 |
| railway lines | 16 |
| ocean | 32 |
| desert | 64 |

You need to be careful though;<br>
firstly, if this is something that you want to expose to usage in the editor, you are currently limited to `uint8` based enums, 
which as previously stated, means a maximum of 8 individual states which can be combined.

If you are using these things solely in C++ code however, you can base your enum on a `uint64`, and have any combination
of 64 states if you want.

The other thing you need to be careful of, is that those individual states are actually things that can be sensibly combined.<br>
Our geography based example, is fine in principle.<br>
But is it really ok to have values for ‘Ocean’ and ‘Desert’, which in theory might not really be able to exist together on the same game tile? 
I mean, you *COULD* allow it, and just say you would never actually do it.<br>
But, ideally, we would have a series of values which make absolute sense in any combination.

However, things are not always so simple, and sometimes we have to do the best we can, 
and we end up with perhaps 4 values that make a sensible combination together,<br>
3 more values which make a sensible combination together, <br>
and basically 1 boolean we stuck on the end because, erm... well, we had a free bit left over and wanted to use it!

~~~cpp
enum class EMessOfThings:uint8
{
    None = 0,
    UseLargeUiFonts = 1,
    UseUiContrastColors = 2,
    UseSubtitles = 4,
    UseUiAudioPrompts = 8,
    IsVulnerableToPoison = 16,
    IsVulnerableToDisease = 32,
    IsVulnerableToPathogens = 64,
    AutoSaveOnLevelChange = 128
}
~~~
Obviously I have greatly exaggerated the differences in this example to demonstrate a point,<br>
you may encounter situations where the distinctions are far more subtle.
{: .notice--warning}

We could look at that enum, and it would be perfectly sensible to say, actually I’ll make 2 separate bitflags enums,
and a separate boolean.<br>
~~~cpp
enum class EUiThings:uint8
{
    None = 0,
    UseLargeFonts = 1,
    UseContrastColors = 2,
    UseSubtitles = 4
}

enum class EVulnerabilities:uint8
{
    None = 0,
    IsVulnerableToPoison = 1,
    IsVulnerableToDisease = 2,
    IsVulnerableToPathogens = 4
}

bool AutoSaveOnLEvelChange = true;
~~~

Or... you might say;<br>
> well, this is something which is going to be replicated for network play, so
I’ll keep it all together as one small compact value that will be transmitted as a single byte, probably faster than the
other options.' 

It really depends on the situation.

# So, more practically, how do we use these things?
Lets take a look at some examples of where enums, and bitflags enums might be used in our games, showing how we go about
interacting with them.

## Standard Enum Examples
Ammunition Types – You might make a variety of different weapons in your game, each using one specific type of ammunition.

We can use an enum to specify all the ammunition types available
~~~cpp
UENUM(BlueprintType)
EAmmunitionType
{
    EnergyPack,
    CaselessBulletRounds,
    MicroRocket,
    CompressedGasProjectile,
    ChemicalLaserRound
}
~~~
We can then have a weapon class, with an `AmmunitionType` property.


# Currency Exchange Rate Example

This Blueprints only example will use an Unreal Editor created `Enumeration` to define several different currencies.<br>
I then go on to create the supporting objects to provide easy conversion between the different currencies.

## Step 1: CurrencyType Enumeration
In the Unreal Editor, I start by creating a new `Enumeration`, which I name `CurrencyType`.

![enum_icon](../assets/unreal/enums/CurrencyType_Enumeration.png){: .align-center}

![currency-type](../assets/unreal/enums/CurrencyType_Creation.png)
And entering four enumeration entries, which represent the four different currencies we might be using in a game.

## Step 2: ExchangeRate Structure
I then create a new `structure`, which I name `ExchangeRate`.<br>
The purpose of this structure is to define a single exchange between two `CurrencyType`s

![structure_icon](../assets/unreal/enums/ExchangeRate_Structure.png){: .align-center}

![exchange_rate_1](../assets/unreal/enums/ExchangeRate_Creation_1.png)
The structure gets a `FromCurrency` which is of type `CurrencyType`,<br>
a `ToCurrency` which is also of `CurrencyType`,<br>
and a `Rate` which is a float.

I set the default values for this structure's three properties:
![exchange_rate_2](../assets/unreal/enums/ExchangeRate_Creation.png)

## Step 3: CurrencyExchange DataTable
Next an Unreal `DataTable` is created to store as many `ExchangeRate` entries as we need.

![data-table](../assets/unreal/enums/CurrencyExchange_DataTable.png){: .align-center}

![](../assets/unreal/enums/CurrencyExchange_Data.png)
I populate the `Data Table` with an entry for each currency pair, in each direction.

To those of you who know a little about real world FX, 
you might think that storing conversions in BOTH directions is a waste of time.<br>
As usually the rate for A >> B is the same as 1 / B >> A<br>
However, it simplifies things for this example, and arguably gives you more flexibility.<br>
For instance, you may decide that you CAN convert A>>B, but not B>>A for some in-game world reason.<br>
Doing it this way would mean we could just omit an entry for B>>A,
and code our conversion routine to handle not finding a valid exchange item.<br>
Or you might say that the exchange 'house' charges more for conversions in a specific direction, for whatever reasons you like.
{: .notice--info}

I leave the [RowName] to the default values that unreal provides for it.

## Step 4: Blueprint function Library
Next, I’ll make a blueprint function library, and add a new function to it called ‘ConvertCurrency’.
![](../assets/unreal/enums/BlueprintFunctionLibrary.png){: .align-center}
![](../assets/unreal/enums/ConvertCurrencyFunction.png){: .align-center}

I’ll add input parameters for the [from] and [to] currencies, and another for the [amount] of from currency that we want
to exchange.
![](../assets/unreal/enums/ConvertCurrencyProperties.png)

What we want to do is find the row in the datatable that has the `FromCCY` and `ToCCY` currencies we are interested in.

There isn't really any simple way to do that though, as `DataTables` work using their `rowname` to pull individual rows
out.<br>
Yeah, sure we could kludge something together, where we used a combination of the from and to enum names as the rowname,
but, well, that’s just a lot of rather pointless work for no real returns.

We don’t have many entries in there, so in this case it's simpler to use a blueprint node that gets all the datatable’s
row names, and just do a for each loop over it, the result is basically for...each(ing) over the datatable rows.

Now that we have actual rows, we can test the `FromCCY` and `ToCCY` currency values to see if they match what we are interested
in. If they do, we use the `Rate` recorded for that row, and multiply it by the `Amount` parameter to get the result, which
we return.

This image shows the full node-graph for the ConvertCurrency function:
![convert-currency-nodes](../assets/unreal/enums/ConvertCurrencyNodes.png)

If we don’t find a row with the specified from and to currencies we return a value of 0. But you could do whatever you
like, such as returning -1 to indicate a conversion failure to the calling code. And if you were going to use something
like this a lot, it might make sense to check if the from and to currencies are the same before even looking at the
datatable, and simply returning the same amount that was passed in if they are.<br>
You know, just in case you forget to ensure that your UI enforces that they are different.
>Belt and Braces

As they say.

# Different ways of numbering your enums in C++
When you are declaring enums and manually assigning the values in C++, you are free to provide the values in a number of
different ways.

## Values in Decimal
![](../assets/unreal/enums/Enum_DECIMAL.png){: .align-right}

You can just stick with decimal, especially if you are only dealing with `uint8` sizes.


## Values in Hexadecimal
{: .cf}
![](../assets/unreal/enums/Enum_HEX.png){: .align-right}

For BitFlags enumerations, many people have traditionally used hexadecimal notation to specify the values.<br>
And a majority of the BitFlag enumerations which Epic uses in the Engine code are declared in this way.

Historically the reason for this is given as;
'it's a bit clearer and less error prone than using decimal.'


## Values in Binary
{: .cf}
![](../assets/unreal/enums/Enum_BINARY.png){: .align-right}

As of C++ 14, we can now write binary literals, which would arguably be the clearest way of declaring these
enumerations, as long as we provide any leading zeros, and align them.

I don’t wanna get into a whole argument about which is better,
for a `uint8`, binary would probably be the clearest to read in your code. But if you have bitflags based on `uint64`… do
you really wanna type out 64 characters for each number? In hexadecimal it's only 16.


## Values as 'left shifted' 1
{: .cf}
![](../assets/unreal/enums/Enum_LSL.png){: .align-right}

And of course, yet another school of thought prefers to use compile time constants, where your enum values are specified
as 1, bitshifted left by the relevant number of bits.

The compiler will recognise these, calculate the appropriate values, and plop them right in there for you.

Its really up to you. I personally tend to specify the values in decimal, but that’s only because once you have worked
with these things for a while you tend to memorise all the values of 2 to n, right up to 2^13 which is 8192, especially
as you keep seeing the various values all over the place in 3d game development, most obviously in texture dimensions.

For this example, I’ll specify them in binary, and put the decimal value in a comment.
{: .cf}

# Bitflag Enum Example
![healthy](../assets/unreal/enums/Injury_Healthy.png){: .align-right}
We are going to make a bitflags enum for tracking injuries to specific body locations on the player character.

In this example I start with an entry for ‘Healthy’ which is set to zero, meaning no injuries, and would most likely be
the default value you would initialise variables to.

The next 6 entries indicate injuries to the player in distinct body locations, and
each is assigned one of those ‘power of 2’ values – that means in binary, each of those values is represented by a
single binary digit, or ‘bit’ in the underlying byte used to store it.

Following that, I have defined 4 more entries, which are simply combinations of the previous entries.
This is just a way of defining particular ‘patterns’ of bits, that might be useful to you. Think of it as a shortcut,
instead of testing for LeftLegDamage and RightLegDamage, you can just test for MobilityImpaired.

You could use UMETA(Hidden) on such 'pattern shortcut' entries, if your enum is something that will be used
from the Unreal Editor. I haven't tried using such an enum as an editor property, 
but I have a sneaky feeling things might look a bit messy if you didnt cover your tracks.
{: .notice--warning}

As you can see, we can set those things, by referencing the previous entries, OR as the last entry shows, by giving the
specific value.

~~~cpp
enum class EPlayerInjuries : uint8
{
    Healthy         = 0b00000000 // 0

    LeftArmDamage   = 0b00000001 // 1
    RightArmDamage  = 0b00000010 // 2

    LeftLegDamage   = 0b00000100 // 4
    RightLegDamage  = 0b00001000 // 8

    CoreDamage      = 0b00010000 // 16
    HeadDamage      = 0b00100000 // 32

    MobilityImpaired = LeftLegDamage + RightLegDamage,		// 0b00001100 (12)
    DexterityImpaired = LeftArmDamage + RightArmDamage,		// 0b00000011 (3)
    SevereIncapacitation = MobilityImpaired + DexterityImpaired,	// 0b00001111 (15)
    AlmostAWrightOff = 63						// 0b00111111 (63)

};
ENUM_CLASS_FLAGS(EPlayerInjuries);
~~~
## Setting Patterns
Once we have a variable typed to our bitflags enum, we can do various things with it.

We can inflict individual wound locations on the Player, additionally to anything they currently have, and if they
already have that specific wound, nothing changes, it’s still injured:

~~~cpp
PlayerWounds |= EPlayerInjuries::HeadDamage;
~~~

And we can also heal specific wounded locations in a similar way:
~~~cpp
PlayerWounds &= ~EPlayerInjuries::HeadDamage;
~~~

Or completely heal the Player with:
~~~cpp
PlayerWounds = EPlayerInjuries::Healthy;
~~~
Similarly, we can test what injuries the Player has and use that information in different ways.<br>
For example, we might decide that the player’s movement speed is affected by certain injuries:
~~~cpp
constexpr float NORMAL_SPEED = 100.0;
constexpr float REDUCED_SPEED = 50.0;

PlayerSpeed = 
    EPlayerInjuries::PlayerWounds & EPlayerInjuries::MobilityImpaired == EplayerInjuries::MobilityImpaired 
    ? REDUCED_SPEED : NORMAL_SPEED;
~~~

Here, we perform the test by doing a bitwise AND (&) of the PlayerWounds variable with the enumeration flags we are
interested in testing for. Any bit which is both 1 in the variable, AND the test flags will return a 1.
Therefore if the result of that is the same as the bitflags we tested it against, then it contains those set bits.

## Common bitflags operations
* Modify a variable to set the bits corresponding to a specific bit pattern.
Use bitwise OR (|) on as many bit patterns you like, and then assign the result to your variable.
In the example above we use C’s bitwise OR and ASSIGNMENT operator, which performs an OR on the variable and the other
value, and then stores the result back in the variable.
* Modify a variable to unset the bits corresponding to a specific bit pattern.
This is the inverse of the previous operation. Before we set the bits in our variable to those in a provided bitpattern,
but this time we are unsetting them. Use the bitwise AND (&) with the inverted value of the bitflags you want to unset,
by applying a NOT (~) operator to those first.
* Test if a specific bit, or pattern of bits are set
If the value of the variable AND the test bits, IS EQUAL to the test bits, it means those testbits are set in the
variable.
* Test if ANY of the bits in a bit pattern are set
Sometimes you only want to know if any bits in a given pattern are used, and maybe you dont care which specifically.
This is basically the same as the previous test, AND the two values together. But this time, we dont compare it back to
the test bits, we simply test if it is GREATER THAN 0. Any value > 0 indicates at least one of the test bits is set.

# Summary
Standard enums, which group a bunch of possible choices together in one convenient entity, allow us to very easily
provide dropdown list selections to our unreal editor users, such as level designers. 
We can test in code which specific choice is used, and as each entry in an enum is just a number, 
we can create them in an order which makes sense, and then use `<, >, >=, <=` operators for certain tasks.

BitFlags enums are a little bit like having a collection of booleans. It means that we can have one simple, compact way
of holding information for things which may or may not use different combinations of associated features, without
needing to have individual booleans defined for each thing.<br>
Variables created from these bitflags enumerations can be set to any arbitrary value, and have specific bits or patterns of bits set or cleared.<br>
We can also test if a variable contains either a specific bit, or specific pattern of bits.
Not only is the memory use smaller than using individual booleans, but consequently communicating state across the network will also be faster.

Hopefully you can see from this basic introduction, how useful enums can be; both for organising your logic, and
producing cleaner code.

As always, good luck with your own game dev projects.
Take care!
