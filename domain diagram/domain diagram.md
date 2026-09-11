---
title: SMD Project 1 Domain Diagram

---

```plantuml
@startuml
skinparam style strictuml
hide empty methods
!theme plain
top to bottom direction

skinparam linetype ortho
skinparam nodesep 80
skinparam ranksep 80

title PacMan in the Multiverse - Domain Model

'might remove player and game 

class Game{
    
}

class Player{
    
}

class Maze {
	width
	height
}

class Cell <<abstract>>{
	x
    y
}

class WallCell{
    
}

class PathCell{
    
}

abstract class Character <<abstract>> {
	currentDirection
}

class PacMan {
	score
	lives
}

'individual monsters don't have special attributes - therefore in domain model don't classify as separate entities
abstract class Monster {
	type
	speed
	state
}


class Troll {
}

'hard to manually move around someone will have to research this
class Orion {
}

class Wizard {
}


abstract class Item <<abstract>>{
	pointValue
	isConsumed

}

/'
Pill has distinct domain handling - so even tho no attributes still ok 
'/
class Pill {
}

class GoldPiece {
	furyDuration
}

class IceCube {
	freezeDuration
}



' Domain Associations


Game "1" -- "1" Maze : played on >


Maze "1"  *-- "100..2500" Cell : composed of 

Cell "1" -- "0..*" Character : occupies <
PathCell "1..*" -left- "0..1" Item : placed on <

Player "1" -right- "1" Game: starts >

Orion "1" -- "1..*" GoldPiece : patrols >


' Generalisations
Character <|-- PacMan
Character <|-- Monster


Monster <|-down- Troll
Monster <|-left- Orion
Monster <|-right- Wizard


Item <|-up- Pill
Item <|-down- GoldPiece
Item <|-left- IceCube

Cell <|-left- PathCell 
Cell <|-up- WallCell 



@enduml
``````plantuml
@startuml
skinparam style strictuml
hide empty methods
!theme plain
top to bottom direction

skinparam linetype ortho
skinparam nodesep 80
skinparam ranksep 80

title PacMan in the Multiverse - Domain Model

/'
Leaving player and game out to start lmk if you disagree and why

'/


class Maze {
  width
  height
}

class Cell {
  x
  y
  isWall
}

abstract class Character {
  currentDirection
}

class PacMan {
  score
  lives
}

abstract class Monster {
  speed
  state
}

class Troll {
}

class Orion {
}

class Wizard {
}

abstract class Item {
  pointValue
  isConsumed
}

class Pill {
}

class GoldPiece {
  furyDuration
}

class IceCube {
  freezeDuration
}

' Domain Associations
/'
Game "1" -- "1" Maze : played on >
Game "1" -- "1" PacMan : tracks >
Game "1" -- "1..*" Monster : contains >
removed game for now

TODO: find a way to make pacman line to game more clear without reordering
'/

Maze "1" *-- "100..2500" Cell : composed of >

Cell "1" -- "0..1" Character : occupies <
Cell "1" -- "0..1" Item : placed on <

' Generalisations
Character <|-- PacMan
Character <|-- Monster

Monster <|-- Troll
Monster <|-- Orion
Monster <|-- Wizard

Item <|-- Pill
Item <|-- GoldPiece
Item <|-- IceCube


IceCube -- Monster : affects >

' Interactions
PacMan "1" -- "0..*" Item : eats >
PacMan "1" -- "0..*" Monster : collides with >
Orion "1" -- "1..*" GoldPiece : patrols >

@enduml
```