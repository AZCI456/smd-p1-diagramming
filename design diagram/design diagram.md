````plantuml
@startuml
/' Touch settings at own peril - better to discuss online first '/
!theme plain
top to bottom direction
hide circle

skinparam linetype ortho
skinparam nodesep 90
skinparam ranksep 80
' use +, -, # instead of shapes
skinparam classAttributeIconSize 0


title PacMan in the Multiverse - Design Model

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
    ' renders the grid'
    -mazeArray : CellType[][]
    ' encapsulation and info expert fields
    -columnCount: int
    -rowCount: int
    ' All items are loaded dynamically on screen based on their position - state - so that it perfectly interacts
    +getItems() : List<Item>
    ' corresponds to the type (int reference) static object at that position
    'UNVERFIED NEED: '+getCell(location): Entity
    ' to be used by Actor classes so they dont pass through walls
    +canMove(location): Boolean
    ' checks if there are any pills or gold left (ignores icecube) - called by EndGameChecker 
    +hasCollectiblesRemaining() : boolean
}

enum CellType {
    WALL
    OPEN
}

/' Super class Too many responsibilities 
    
    Bootstrapper: game should focus on intialisation
    Execoutor: gamecontroller on execution

    Removed: Grid reference, act reference sent to GController

'/

class Game{
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
    ' use method injection to handle updates on call
    -collisionHandler : CollisionHandler
    -gameCallback : GameCallback
    ' minimal constructor 
    +GameController(grid,  gameCallback)
    'called in construction - can replace with a Object 
    +init_level(pacActor, monsters)
    ' to advance the world by one step - public by definition in JGameGrid
    +act() : void
    ' called within act() to check if the game is ended (ie !ONGOING) - if so stop game and '
    -checkGameState(pacActor : PacActor, monsters : List<Monster>) : GAMESTATE
    +GameController(grid : GameGrid, gameCallback : GameCallback)
    -setMonsterStates(state : MonsterState) : void
}            


enum GameState {
    ONGOING
    WIN
    LOSE
}

/'
grid, gameCallback - removed from item consumptoin replaced with abstract class item

'/
class CollisionHandler  {
    ' smarter is to hashmap monster positions however not necessary for monsters << n'
    ' designed to use method injection
    +checkPacManMonsterCollision(gc: GameController, pacActor : PacActor, monsters : List<Monster>) : boolean
    ' method is polymorphic - calls act on item and item handles the changes - more extensible + item info expert'
    +checkItemConsumption(gc : GameController, pacActor : PacActor, items : List<Item>) : void
}
/'TODO: change all function signatures to the format name : Type '/

/'represents everything visible on the map - actors and items'/
abstract class Entity{
    -location : Location
    -sprite: RandoImg.jpg/idk    
}



' ------------------------------------------------------
' ------------------ITEMS SECTION ---------------------
' ------------------------------------------------------

/'TODO: examine interaction of items with rest of system

entity cannot exist without a cell I'm pretty sure (might need to add a cell class and attach it to grid)

orion and gold piece - packman and all items 
monsters and fury
'/

' Assuming that items don't overlap in the testing - TODO: confirm if this introduces issues
abstract class Item{
    +{abstract} applyEffect(gc : GameController) : void
}

class Pill extends Item{
    ' increment the score by 1'
    +applyEffect(gc : GameController) : void
}

' orion `'
class GoldPiece extends Item{
    ' increment score by 5 and then perform fury '
    +applyEffect(gc : GameController) : void

}

class IceCube extends Item{
    ' setup monster freeze state - watch the wizard '
    +applyEffect(gc : GameController) : void
}

' ------------------------------------------------------
' ------------------ACTORS SECTION ---------------------
' ------------------------------------------------------


' Actor location redundant class removed - prevents fragmentation'

' Parent of all dynamic entities'
abstract class Actor extends Entity{
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
    -stateTimer: int
    ' passing in grid redundent - type is useless with polymorphism'
    +Monster(initialLocation, initialDirection)
    +updateStateTimer(newState: MonsterState)
    
}

' static class / utility class - completely decouples monster / actor from random generation'
class GameRandomiser {
    -{static} random : Random
    +{static} init(seed : long) : void
    +{static} nextInt(bound : int) : int
    ' Generic Type - select random from any list'
    +{static} pickRandom(items : List<T>) : T
}

/' all possible states after item consumption 
designed using info expert - the timer is stored with the state
'/
enum MonsterState {
    NORMAL(0)
    FURIOUS(5)
    FROZEN(3)
    REDUCEDSPEED(3)
    -duration : int
    +getDuration() : int
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
    ' The patrol route - not the gold just the locations - decoupled
    -patrolLocations : List<Location>
    ' where the next petrol location is (start at 0 use mod to loop around)
    -currentIndex: int
    +act() : void
    +Orion(location : Location, direction : CompassDirection, patrolLocations : List<Location>)
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
    +{static} createMonster(monsterType : String, location : Location, direction : CompassDirection) : Monster
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

' additional item relationships
Item --|> Entity
Pill --|> Item
GoldPiece --|> Item
IceCube --|> Item

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
GameController "1" o-- "0..*" Monster


' CollisionHandler
CollisionHandler ..> PacActor : checks
CollisionHandler ..> Monster : checks

' PacActor
PacActor --> GameGrid : queries cells

' Monster & Subclasses
' unavoidable under the Information Expert principle. Monsters cannot autonomously navigate, check for wall collisions, or inspect adjacent cells ithout querying grid layout data. consider dependency modelling however
Monster --> GameGrid : queries cells
Monster --> MonsterState : has state
Troll ..> GameRandomiser : uses

' MapLoader
MapLoader ..> GameGrid : configures
MapLoader ..> MonsterFactory : requests creation

' MonsterFactory
MonsterFactory ..> Monster : creates
MonsterFactory ..> Orion : creates with gold locations


' additional....
' Collision & Items
CollisionHandler ..> Item : interacts
Item ..> GameController : modifies state

' Factory
MonsterFactory ..> Wizard : creates
MonsterFactory ..> Troll : creates

' Grid & Entities
' aggregation - can exist without but is contained within
GameGrid "1" o-- "0..*" Item : contains
GameGrid ..> CellType : uses


@enduml
````


