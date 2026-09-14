````plantuml
@startuml
/' Touch settings at own peril - better to discus online first '/
!theme plain
top to bottom direction
hide circle

skinparam linetype ortho
skinparam nodesep 90
skinparam ranksep 80
' use +, -, # instead of shapes
skinparam classAttributeIconSize 0

' -----------------------------------------------------
' -----------------------------------------------------
' ------------------ CLASS DEFINITIONS ----------------
' -----------------------------------------------------
' -----------------------------------------------------


' Program entry point'
class Driver  {
    +{static} DEFAULT_PROPERTIES_PATH : String
    +{static} main(args) : void
}
' Loads configurable maze / all new game additions
' Info expert assignment - otherwise fine
class PropertiesLoader {
    -{static} loadPropertiesFile(path) : Properties
}

/' Implementation for issue 3 corresponding to incorrect grid class assignment

Details current grid creates high coupling with everything
Remove references referenc ethrough GController

Implementing an abstract interface and a single inheritence is code smell

Will implement in a single class

inherits from jgamegrid: GUI component. The engine automatically creates the game window, renders the underlying background tiles, manages pixel-to-grid coordinate translations, and redraws the display whenever state changes occur
'/

class GameGrid <<JGameGrid>>{
    -mazeArray : int[][]
    ' encapsulation and info expert fields
    -columnCount: int
    -rowCount: int
    ' probably better to return the object at the position. ie the reference to that objects instantiation.
    +getItem(Position) : Cell 
    ' corresponds to the type (int reference) static object at that position
    +getCell(location): int
    ' to be used by Actor classes so they dont pass through walls
    +canMove(location): Boolean
}

/' Super class Too many responsibilities 
    
    Bootstrapper: game should focus on intialisation
    Execoutor: gamecontroller on execution

    Removed: Grid reference, act reference sent to GController

'/

class Game <<Aaron>>{
    -gameController : GameController
    ' retrieve the accumulated log data
    -gameCallback : GameCallback
    +Game(gameCallback, properties)
    ' boots up program
    +runApp() : String
    -buildInitialActorLocations(mapData) : List<ActorLocation>
}

' seems fine ig - TODO: only two connections g and gcon associations (fixed ref)'
class GameCallback  {
    -logBuilder : StringBuilder
    +pacManLocationChanged(...) : void
    +monsterLocationChanged(...) : void
    +pacManEatPillsAndItems(...) : void
    +endOfGame(gameResult) : void
    +getAllLog() : String
}

/' Super class Too many responsibilities 

    Changed : grid visibility modifier and reference pack vis modifier
    Constructor bloat - remove pacActor, monsters,collision handler


'/
class GameController {
    ' runtime orchestrator - information expert 
    -grid : GameGrid
    -pacActor : PacActor
    -monsters : List<Monster>
    ' delegate instatiation to GameController - remove parameter from constructor
    -collisionHandler : CollisionHandler
    -gameCallback : GameCallback
    ' minimal constructor 
    +GameController(grid,  gameCallback)
    'called in construction - can replace with a Object 
    +init_level(pacActor, monsters)
    ' to advance the world by one step
    +act() : void
    ' kinda useless getters and setters TODO: perhaps remove from the design'
    +getActorLocations() : List<ActorLocation>
    +getPacActor() : PacActor
}            

/'Exist as otherwise game controller has to ping too many other classes - not sure what this violates - perhaps its high coupling

creates an intermediary allows for better extensibility and protected variations
'/
class EndGameChecker{
    -gameEnded: Boolean
    ' should return the output rather than void
    +checkEndGame() : Boolean
}
' To-do Create weak dependency lines between end game checker items. create dependency between game controller and this 

/'

grid, gameCallback - removed from item consumptoin replaced with abstract class item

'/
class CollisionHandler  {
    ' smarter is to hashmap monster positions however not necessary for monsters << n'
    +checkPacManMonsterCollision(pacActor : PacActor, monsters : List<Monster>) : boolean
    +handleItemConsumption(pacActor : PacActor, item : Item) : void
}
/'TODO: change all functoin signutures to the format name : Type '/

/'represents everything visible on the map - actors and items'/
abstract class Entity{
    -location : Location
    -sprite: RandoImg.jpg/idk
}



' ------------------------------------------------------
' ------------------ITEMS SECTION ---------------------
' ------------------------------------------------------

abstract class Item{
    
}


' ------------------------------------------------------
' ------------------ACTORS SECTION ---------------------
' ------------------------------------------------------


' Actor location redundant class removed - prevents fragmentation'

' Parent of all dynamic entities'
abstract class Actor extends entity{
    -direction : CompassDirection
    ' forces child class implementation
    +{abstract} act() : void
}

/' PacActor analysis
Issues regarding single responsibility 

    +eatPill(location) : void
    - violates single responsibility and information expert - move to Coll Handler

        +act() : void
    - abstract not necessary 

    irrelevant
    -randomiser : Random
    -grid : IGameGrid

     PacActor --|> Actor

'/

class PacActor extends Actor {
    -pacmanController : PacmanController
    -nbPills : int
    -score : int
    ' resp info expert on pac pos'
    +canMove(location) : boolean
}

' paccontroller unnecessary indirection - does nothing moved to gg key - TODO analyse if this will affect coupling - doesn't make sense not to.

/' provided directly by ch.aplu.jgamegrid - hense inheritence 


TODO: PacActor ..|> GGKeyRepeatListener
'/
interface GGKeyRepeatListener <<JGameGrid>>{
    +keyRepeated(keyCode : int) : void
}

/'
Analysis of Monster: 

issues
    #grid : IGameGrid
    ''+ SRP & Pure Fabrication Violation for below
    randomiser 

'/
abstract class Monster extends Actor {    
    ' Enum as fixed states based on interactions
    -currentState: MonsterState
    ' passing in grid redundent - type is useless with polymorphism'
    +Monster(initialLocation, initialDirection)
    
}

' static class / utility class - completely decouples monster / actor from random generation'
class GameRandomiser {
    -{static} random : Random
    +{static} init(seed : long) : void
    +{static} nextInt(bound : int) : int
    ' Generic Type - select random from any list'
    +{static} pickRandom(items : List<T>) : T
}

/' all possible states after item consumption '/
enum MonsterState {
    NORMAL
    FRIGHTENED
    FROZEN
    ' for wizard esp
    REDUCEDSPEED
}

' ----------- MONSTER SUBCLASSES --------------

/'
    Doesn't need anything - randomiser static
    change location inherited from actor - called in act
'/
class Troll extends Monster {
    +act() : void
}


/'
Spawn pos vs first coin

TODO: constructor
+Orion(patrolLocations : List<Location>)

 overridden act() method simply checks the coordinates of patrolLocations.get(currentIndex), moves one step closer to it (in either free direction), and increments the index when it arrives.
'/
class Orion extends Monster {
    ' The patrol route'
    -patrolLocations : List<Location>
    ' where the next petrol location is (start at 0 use mod to loop around)
    -currentIndex: int
    +act() : void
}

/'

'/
class Wizard extends Monster {
    -checkAdjacentWall(location : Location, direction : CompassDirection) : Location
    +act() : void
}


/' factory faster for creating monsters from the config

Issues: everything else wtf 
    grid X
    Config unnecessary middle man creates unnecessary dependency and another place to update to maintain - bad for protected variation

scraped - everything relating to mapdata / loader all ott bad engineering

simple implementation

'/
class MapLoader <<utility>> {
    +{static} loadMap(properties : Properties, grid : GameGrid) : void
}


/' Monster factory analysis

static utility class

'/
class MonsterFactory {
    ' Troll & Orion creation
    +{static} createMonster(type : MonsterType, location : Location, direction : CompassDirection) : Monster
    ' Orion creation
    +{static} createOrion(location : Location, direction : CompassDirection, goldLocations : List<Location>) : Orion
}



' -----------------------------------------------------
' -----------------------------------------------------
' ------------------ GENERALISATIONS ------------------
' -----------------------------------------------------
' -----------------------------------------------------

PacActor --|> Actor
Monster --|> Actor
Troll --|> Monster
Orion --|> Monster
Wizard --|> Monster
PacActor ..|> GGKeyRepeatListener


' -----------------------------------------------------
' ------------------ OTHER RELATIONSHIPS --------------
' ------------IE ASSOCIATIONS, COMPOSITION, AGGRE... --
' -----------------------------------------------------

' Driver
Driver ..> Game : creates
Driver ..> PropertiesLoader : loads
Driver ..> GameCallback : creates

' Game
Game --> GameCallback
Game *-- GameController
Game ..> MapLoader : loads map data

' GameController
GameController --> GameGrid
GameController --> GameCallback
GameController *-- PacActor
GameController *-- CollisionHandler
GameController --> EndGameChecker : uses
GameController "1" o-- "0..*" Monster

' EndGameChecker
EndGameChecker ..> GameController : queries state

' CollisionHandler
CollisionHandler ..> PacActor : checks
CollisionHandler ..> Monster : checks

' PacActor
PacActor --> GameGrid : queries cells

' Monster & Subclasses
Monster --> GameGrid : queries cells
Monster --> MonsterState : has state
Troll ..> GameRandomiser : uses

' MapLoader
MapLoader ..> GameGrid : configures
MapLoader ..> MonsterFactory : requests creation

' MonsterFactory
MonsterFactory ..> Monster : creates
MonsterFactory ..> Orion : creates with gold locations

@enduml
````


