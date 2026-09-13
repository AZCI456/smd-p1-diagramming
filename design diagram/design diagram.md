````plantuml
@startuml
/' Touch settings at own peril - better to discus online first '/
!theme plain
top to bottom direction
hide circle

skinparam linetype ortho
skinparam nodesep 90
skinparam ranksep 80

/'
Formatting to show who set up what
'/

' Aaron - Blue
skinparam class<<Aaron>>{
    BackgroundColor #E3F2FD
    BorderColor #1E88E5
}

' Tsun - Yellow
skinparam class<<Tsun>>{
    BackgroundColor #FFF9C4
    BorderColor #FDD835
}

' Ben - Green
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

' Loads configurable maze / all new game additions
' Info expert assignment - otherwise fine
class PropertiesLoader <<Aaron>> {
    -{static} loadPropertiesFile(path) : Properties
}

' Super class Too many responsibilities 
class Game <<Aaron>>{
    -gameController : GameController
    -gameCallback : GameCallback
    ' IGameGrid is an interface grid should be an actual class 
    -grid : IGameGrid
    +Game(gameCallback, properties)
    +act() : void
    +runApp() : String
    -buildInitialActorLocations(mapData) : List<ActorLocation>
}

' exists because cannot use package as datatype 
class GameGrid <<JGameGrid>>

class GameCallback  {
    -logBuilder : StringBuilder
    +pacManLocationChanged(...) : void
    +monsterLocationChanged(...) : void
    +pacManEatPillsAndItems(...) : void
    +endOfGame(gameResult) : void
    +getAllLog() : String
}

class GameController <<Tsun>> {
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

' stupido design
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

/'  Des Issue 2 resolved: removed direct dependecies with GameControler
add params to act() to update state instead
    Des Issue 4 resolved: Monster now abstract and polymorphic'/
abstract class Monster <<Tsun>> extends Actor {
    #grid : IGameGrid
    #randomiser : Random
    +Monster(type, initialLocation, initialDirection, grid)
    ' What is the difference between these two below - also note the visibility (+) should it be public? Most probably not @Aaron
    +{abstract} act() : void
    +act(pacActorLocation : Location) : void
    +getType() : MonsterType
    +getState() : String
    #canMoveTo(location) : boolean
}

class Troll <<Tsun>> extends Monster {
}

class Ghost <<Tsun>> extends Monster {
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

' Phat code smell - get rid of make monster abstract and generalise with inheritence @Aaron
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


