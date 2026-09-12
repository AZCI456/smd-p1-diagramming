````plantuml
@startuml
!theme plain
top to bottom direction
hide circle

skinparam linetype ortho
skinparam nodesep 90
skinparam ranksep 80

/'
    Don't touch the above settings - if you wanna change something post it on discord first so we can all see 

    Take a look at the syntax at planttext.com

    Happy coding
'/

/'
Formatting to show who set up what - smarter way to organise blame (its a software term - not the pejorative connitation)

put it after the class name to highlight it - I've listed game as an example with myself
'/

' Aaron
skinparam class<<Aaron>>{
    BackgroundColor #E3F2FD
    BorderColor #1E88E5
}

' Tsun
skinparam class<<Tsun>>{
    BackgroundColor #FFF9C4
    BorderColor #FDD835
}

' Ben
skinparam class<<Ben>>{
    BackgroundColor #E8F5E9
    BorderColor #43A047
}

'''' begin code 

hide stereotype
show <<interface>> stereotype
show <<abstract>> stereotype

class Driver  {
    +{static} DEFAULT_PROPERTIES_PATH : String
    +{static} main(args) : void
}

class PropertiesLoader  {
    +{static} loadPropertiesFile(path) : Properties
}

class Game <<Aaron>> {
    -gameController : GameController
    -gameCallback : GameCallback
    -grid : IGameGrid
    +Game(gameCallback, properties)
    +act() : void
    +runApp() : String
    -buildInitialActorLocations(mapData) : List<ActorLocation>
}

class GameGrid <<JGameGrid>>

class GameCallback  {
    -logBuilder : StringBuilder
    +pacManLocationChanged(...) : void
    +monsterLocationChanged(...) : void
    +pacManEatPillsAndItems(...) : void
    +endOfGame(gameResult) : void
    +getAllLog() : String
}



class GameController  {
    #grid : IGameGrid
    #pacActor : PacActor
    -monsters : List<Monster>
    -collisionHandler : CollisionHandler
    -gameCallback : GameCallback
    +GameController(grid, pacActor, monsters, collisionHandler, gameCallback)
    +act() : void
    +checkEndGame() : void
    +hasEndGame() : boolean
    +getActorLocations() : List<ActorLocation>
    +getPacActor() : PacActor
}            

class CollisionHandler  {
    +checkPacManMonsterCollision(pacActor, monsters) : boolean
    +handleItemConsumption(pacActor, grid, gameCallback) : void
}

interface IGameGrid  {
    +getCell(location) : int
    +setCell(location, value: int) : void
    +isWall(location) : boolean
    +isInBounds(location) : boolean
    +getAvailableCells() : List<Location>
    +getWidth() : int
    +getHeight() : int
}


class PacManGameGrid  implements IGameGrid {
    -mazeArray : int[][]
    +getCell(location) : int
    +setCell(location, value: int) : void
    +isWall(location) : boolean
    +isInBounds(location) : boolean
    +getAvailableCells() : List<Location>
    +getWidth() : int
    +getHeight() : int
}

class PacActor  {
    -pacmanController : PacmanController
    -nbPills : int
    -score : int
    -randomiser : Random
    -grid : IGameGrid
    +act() : void
    +eatPill(location) : void
    +canMove(location) : boolean
}

class PacmanController  {
    -pacActor : PacActor
    +keyRepeated(keyCode) : void
}

interface GGKeyRepeatListener 

abstract class Monster extends Actor {
    #grid : IGameGrid
    #randomiser : Random
    +Monster(type, initialLocation, initialDirection, grid)
    +{abstract} act() : void
    +getType() : MonsterType
    +getState() : String
    #canMoveTo(location) : boolean
}

class Troll extends Monster {
    +act() : void
}

class Ghost extends Monster {
    +act() : void
}


class MonsterFactory  {
    +{static} createMonsters(spawnConfigs : List<MonsterSpawnConfig>, grid : IGameGrid) : List<Monster>
}


class MapLoader  {
    +{static} loadMap(properties : Properties) : MapData
}

class MapData  {
    -mazeArray : int[][]
    -pacManSpawn : Location
    -pacManSpawnDirection : CompassDirection
    -monsterSpawnConfigs : List<MonsterSpawnConfig>
}

class MonsterSpawnConfig  {
    -type : MonsterType
    -location : Location
    -direction : CompassDirection
}

enum MonsterType  {
    TROLL
    GHOST
}


class ActorLocation  {
    -actor : Actor
    -location : Location
    -direction : CompassDirection
}

class Actor  {
    -location : Location
    -direction : CompassDirection
    +getLocation() : Location
    +setLocation(location) : void
    +getDirection() : CompassDirection
    +setDirection(direction) : void
    +{abstract} act() : void
}

' Relationships
Driver ..> Game : creates
Driver ..> PropertiesLoader : loads
Driver ..> GameCallback : creates

Game --> GameGrid
Game --> GameCallback
Game *-- GameController
Game ..> MapLoader : loads map data
Game ..> MonsterFactory : creates monsters
Game ..> ActorLocation : builds initial placements
Game --> IGameGrid

GameController --> GameCallback
GameController --> IGameGrid
GameController *-- PacActor
GameController *-- CollisionHandler
GameController "1" o-- "0..*" Monster
CollisionHandler ..> IGameGrid : queries cells
CollisionHandler ..> GameCallback : reports events

PacActor --> IGameGrid : queries cells
PacmanController --> PacActor : drives via keys
PacmanController ..|> GGKeyRepeatListener

PacActor --|> Actor
Monster --|> Actor
Monster --> IGameGrid : queries cells

MapLoader ..> MapData : produces
MapData "1" --> "0..*" MonsterSpawnConfig
MonsterFactory ..> MonsterSpawnConfig : reads
MonsterFactory ..> Monster : creates
Monster ..> MonsterType : getType()

ActorLocation --> Actor
@enduml
````


